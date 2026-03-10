# Nexus — Documentação de Arquitetura
> Registro das decisões técnicas e discussões de design — Março 2026

---

## Índice

1. [Gerenciamento de Memória](#1-gerenciamento-de-memória)
2. [Autenticação e Identidade](#2-autenticação-e-identidade)
3. [Infraestrutura e Banco de Dados](#3-infraestrutura-e-banco-de-dados)
4. [Integração WhatsApp via n8n](#4-integração-whatsapp-via-n8n)
5. [Arquitetura de Agentes](#5-arquitetura-de-agentes)
6. [Controle de Acesso a Tools](#6-controle-de-acesso-a-tools)
7. [Roadmap de Melhorias](#7-roadmap-de-melhorias)

---

## 1. Gerenciamento de Memória

### 1.1 Três Camadas

A memória é organizada em três horizontes temporais independentes:

| Camada | Horizonte | Armazenamento | Conteúdo |
|---|---|---|---|
| Curto Prazo | Turno atual | State (in-memory) | Últimas N mensagens |
| Médio Prazo | Sessão corrente | State: `summary` | Resumo comprimido das mensagens antigas |
| Longo Prazo | Cross-session | Banco vetorial externo | Fatos, preferências, entidades persistentes |

### 1.2 Fluxo do Ciclo

**Fase Pre-LLM — Recuperação:**

- **R1** busca semanticamente no banco vetorial os fatos mais relevantes ao input atual e os anexa ao state como `retrieved_facts`
- **R2** carrega o campo `summary` do state — resumo comprimido das mensagens antigas da sessão. Persiste entre turnos via checkpointer
- **R3** aplica sliding window, truncando a lista de mensagens para as `SHORT_TERM_LIMIT` mais recentes

O contexto é montado em ordem: fatos recuperados → resumo de sessão → últimas N mensagens (do mais geral para o mais específico).

**Geração:**

O LLM recebe o contexto consolidado e produz a resposta. Único ponto de invocação do modelo principal no ciclo.

**Fase Post-LLM — Consolidação:**

- **C1** verifica se o total de mensagens excedeu `SUMMARIZATION_THRESHOLD`. Se sim, o nó de sumarização comprime as mensagens antigas, atualiza `summary` e remove as mensagens sumarizadas
- **C2** avalia se a última interação contém fatos para persistência. Se sim, extrai entidades estruturadas (`entity`, `value`, `type`) e persiste no banco vetorial com `created_at`

### 1.3 Auditoria Transversal

Nós de auditoria são inseridos nos pontos críticos do ciclo. Cada nó registra assincronamente (fire-and-forget) e retorna o state intacto — nunca modifica o fluxo.

| Nó | Posição | O que registra |
|---|---|---|
| `AUDIT_IN` | Entrada | input, session_id, user_id, timestamp, trace_id |
| `AUDIT_R1` | Após recuperação | fatos recuperados e scores |
| `AUDIT_LLM` | Após geração | resposta, tokens, latência |
| `AUDIT_SUM` | Após sumarização | mensagens removidas, novo summary |
| `AUDIT_LTM` | Após persistência | entidades persistidas, tipo de operação |
| `AUDIT_OUT` | Fim do ciclo | snapshot completo do state final |

O `trace_id` é gerado na entrada e amarra todos os logs do ciclo. O banco de auditoria é separado do banco de longo prazo.

### 1.4 Busca Semântica com Filtro Temporal

O vetor armazena a semântica do conteúdo. O timestamp fica nos metadados do payload — são dimensões independentes.

```
{
  "id": "fact_abc123",
  "vector": [...],
  "metadata": {
    "content": "usuário prefere Python para scripts",
    "created_at": "2026-02-28T14:32:00Z",
    "user_id": "user_001"
  }
}
```

Quando o input contém referência temporal ("semana passada", "mês passado"), um extrator leve resolve o range de datas antes de R1. A busca vetorial roda com filtro de metadados — `WHERE created_at BETWEEN $1 AND $2` — sem alterar a interface do nó.

### 1.5 Variáveis de Configuração

| Variável | Padrão | Descrição |
|---|---|---|
| `SHORT_TERM_LIMIT` | 15 | Máximo de mensagens na janela deslizante |
| `SUMMARIZATION_THRESHOLD` | 30 | Total de mensagens que aciona sumarização |
| `LONG_TERM_TOP_K` | 5 | Número de fatos recuperados por busca |

### 1.6 Pontos de Extensão Planejados

Cada melhoria abaixo é isolada — não altera a estrutura do grafo, só a implementação interna do nó correspondente:

- **Threshold de similaridade** em R1 — `if score >= 0.75` antes de injetar fatos
- **Router semântico** — nó condicional antes de R1 que decide se a busca é necessária
- **Atualização incremental de resumo** — nó entre CORE e C1 que atualiza `summary` a cada turno
- **Merge/Dedup em L2** — upsert em vez de insert, verifica se entidade já existe
- **TTL/Decay** — job periódico externo, não toca no grafo
- **OpenTelemetry / Langfuse** — troca a implementação de `_audit_write`, sem alterar nós

---

## 2. Autenticação e Identidade

### 2.1 Separação de Responsabilidades

```
Google OAuth  →  autentica quem é o usuário
IAM Nexus     →  define o que esse usuário pode fazer
Middleware    →  aplica as permissões no runtime do grafo
```

O Google não autentica o WhatsApp. Os dois canais chegam por caminhos distintos e convergem no IAM.

### 2.2 Fluxo Web (Metas)

O login é iniciado pelo sistema Metas via Google OAuth. O Supabase Auth gerencia o fluxo OAuth, elimina a implementação manual de PKCE e refresh token.

Após validação, o IAM do Nexus:
1. Cria ou atualiza o usuário no Identity Registry (`google_sub + email + role`)
2. Resolve a deny list do cargo
3. Emite um JWT próprio com `sub`, `role`, `available_agents[]` e `channel`

O JWT do Nexus é o que circula em todas as requisições subsequentes — o token do Google não é usado diretamente no grafo.

### 2.3 Deny List por Cargo

O modelo adotado é **deny list**, não allow list. O pressuposto é que funcionários têm acesso a tudo por padrão — só se restringe o que não pode.

```sql
-- Só o que o cargo NÃO pode usar
CREATE TABLE role_denials (
    role        TEXT NOT NULL,
    agent_name  TEXT NOT NULL,
    reason      TEXT,
    PRIMARY KEY (role, agent_name)
);

-- Override individual por usuário
CREATE TABLE user_denials (
    user_id     UUID NOT NULL,
    agent_name  TEXT NOT NULL,
    reason      TEXT,
    PRIMARY KEY (user_id, agent_name)
);
```

**Por que deny list:** ao adicionar um novo agente ao sistema, todos os cargos já têm acesso automaticamente. Só restringe se precisar. A lista de exceções é sempre menor que a lista de permissões.

**Hierarquia de decisão:**
1. `user_denials` — exceção individual tem prioridade máxima
2. `role_denials` — restrição do cargo
3. Default — tudo permitido

### 2.4 JWT do Nexus

O token carrega `available_agents[]` já resolvido — diferença chave em relação ao modelo anterior que carregava `permissions[]` de tools individuais.

```json
{
  "sub": "user_123",
  "role": "analyst",
  "available_agents": ["agente_dados", "agente_financas"],
  "channel": "web",
  "exp": 1234567890
}
```

Criptografia assimétrica (RS256). O sistema Metas assina, o Nexus valida sem consultar o banco.

### 2.5 Identity Registry

Mapeia o usuário de canais específicos para um `user_id` interno único. Desacopla a identidade do canal de entrada.

```sql
CREATE TABLE users (
    id              UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    email           TEXT UNIQUE NOT NULL,
    google_sub      TEXT UNIQUE NOT NULL,
    role            TEXT NOT NULL,
    wa_phone        TEXT UNIQUE,
    external_system_id TEXT,
    created_at      TIMESTAMPTZ DEFAULT now()
);
```

O `external_system_id` mantém o vínculo com o sistema Metas — se o cargo mudar lá, sincroniza aqui via upsert.

---

## 3. Infraestrutura e Banco de Dados

### 3.1 Migração MongoDB → Supabase (PostgreSQL)

O MongoDB servia dois propósitos no Qorp original:
- **Checkpointer do LangGraph** — persistência do state entre turnos
- **Perfil do usuário** — preferências comportamentais

Ambos migram para Supabase com impacto mínimo no código.

**Checkpointer — substituição direta:**

```python
# Antes
from langgraph.checkpoint.mongodb import MongoDBSaver

# Depois
from langgraph.checkpoint.postgres import PostgresSaver
checkpointer = PostgresSaver.from_conn_string(DIRECT_URL)
```

LangGraph cria as tabelas automaticamente. Zero mudança no grafo.

**Perfil do usuário — schema estruturado:**

```sql
CREATE TABLE user_facts (
    id          UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    user_id     UUID NOT NULL,
    entity      TEXT,
    value       TEXT,
    type        TEXT,  -- preference | fact | task
    created_at  TIMESTAMPTZ DEFAULT now()
);
```

Cada fato é uma linha — permite filtrar por tipo, aplicar TTL e fazer busca granular no futuro.

### 3.2 Supabase como Plataforma Única

O Supabase resolve três necessidades em um único serviço:

- **PostgreSQL** para checkpointer e perfis
- **pgvector** para memória de longo prazo (extensão nativa)
- **Row Level Security** para isolamento por `user_id` na camada de dados
- **Supabase Auth** para o fluxo Google OAuth sem implementação manual

### 3.3 Atenção ao Connection Pooling

O `PostgresSaver` usa advisory locks que não funcionam com PgBouncer em modo `transaction`. Usar a URL direta (porta 5432) para o checkpointer, não a URL do pooler (porta 6543).

```python
DIRECT_URL = "postgresql://postgres:[senha]@db.[ref].supabase.co:5432/postgres"
checkpointer = PostgresSaver.from_conn_string(DIRECT_URL)
```

Demais queries (perfil, memória vetorial) podem usar o pooler normalmente.

### 3.4 Redis — Decisão de Não Adotar Agora

Redis foi analisado para cache de perfil, rate limiting e idempotência de webhook. Conclusão: não necessário no estágio atual.

- O gargalo do Nexus é latência do LLM, não do banco
- Idempotência de webhook ficou delegada ao n8n
- Rate limiting pode ser tratado no n8n

Redis entra de forma cirúrgica no futuro se aparecerem: múltiplos workers precisando compartilhar estado, rate limiting cross-canal, ou cache de queries caras no BigQuery.

---

## 4. Integração WhatsApp via n8n

### 4.1 Arquitetura da Integração

O n8n é a camada de integração entre o provedor de WhatsApp e o backend Nexus. O backend não expõe webhook público — o n8n absorve essa responsabilidade.

```
WhatsApp → n8n (recebe, roteia) → Nexus Backend (/internal/*)
```

Vantagem principal: troca de provedor (Evolution API, Twilio, Z-API) só muda o node no n8n. O backend não toca.

### 4.2 Vinculação de Número

Fluxo iniciado pelo usuário logado no Metas:

1. Usuário informa o número na interface web
2. Backend gera código de 6 dígitos com TTL de 15 minutos
3. **Backend chama o n8n** que dispara a mensagem com o código via WhatsApp — o usuário não copia e cola nada
4. Usuário responde com o código
5. n8n faz POST em `/internal/wa/link` com número + código
6. Backend valida código + número + expiração juntos
7. Vincula `wa_phone` ao `user_id`

O código expirado ou inválido reinicia o fluxo de geração. Número já vinculado a outro usuário notifica o usuário anterior e remove o vínculo antigo.

### 4.3 Dois Workflows n8n

**Workflow 1 — Recebimento:**
```
Trigger: mensagem recebida
      ↓
IF: texto é código de 6 dígitos?
      ↓ Sim                     ↓ Não
POST /internal/wa/link    POST /internal/wa/message
      ↓                         ↓
      └──────── Envia reply ao usuário ────────┘
```

**Workflow 2 — Disparo de código:**
```
Trigger: webhook chamado pelo backend
      ↓
WhatsApp node: envia mensagem com o código
```

### 4.4 Autenticação das Rotas Internas

As rotas `/internal/*` são autenticadas por API key compartilhada com o n8n — não por JWT de usuário.

```python
@router.post("/internal/wa/message")
async def receive_wa_message(
    payload: WaMessagePayload,
    api_key: str = Header(alias="X-N8N-Key")
):
    if api_key != settings.N8N_SECRET:
        raise HTTPException(403)
    ...
```

### 4.5 Fluxo WA no IAM

Mensagens vindas do WhatsApp passam pelo IAM antes do Core — igual ao fluxo web, mas sem JWT. O backend resolve o `user_id` pelo número e o IAM valida diretamente pelo `user_id`:

```
n8n → POST /internal/wa/message
    → resolve user_id pelo wa_phone
    → IAM_GUARD valida user_id e monta available_agents[]
    → Supervisor recebe contexto com permissões resolvidas
```

### 4.6 Estrutura de Rotas

```
POST /webhooks/whatsapp          ← não existe mais (responsabilidade do n8n)
POST /internal/wa/message        ← privada, autenticada por X-N8N-Key
POST /internal/wa/link           ← privada, autenticada por X-N8N-Key
POST /api/v1/whatsapp/link       ← privada, autenticada por JWT (usuário logado)
DELETE /api/v1/whatsapp/link     ← privada, JWT (desvinculação)
POST /internal/users             ← privada, autenticada por chave interna (Metas → Nexus)
```

---

## 5. Arquitetura de Agentes

### 5.1 Problema do Modelo Flat

O modelo onde o IAM injeta tools diretamente não escala:

- Matriz de N usuários × M tools impossível de manter
- Agentes isolados sem comunicação
- Sem hierarquia — tool trivial no mesmo nível que tool sensível
- Adicionar agente = N linhas novas de permissão

### 5.2 Modelo Supervisor

O IAM passa a controlar acesso a **agentes**, não a tools individuais. Um agente supervisor orquestra os especialistas.

```
IAM → available_agents: ["agente_dados", "agente_financas"]
            ↓
       Supervisor
      (orquestra)
      ↙        ↘
Agente Dados   Agente Finanças
(dono das      (dono das
suas tools)    suas tools)
```

**Vantagens:**
- Adicionar agente = 1 entrada na deny list se necessário, nada mais
- Tools encapsuladas dentro de cada agente
- IAM nunca precisa conhecer tools individuais

### 5.3 Tool Registry — Instâncias Compartilhadas

Tools ficam num registry central com instâncias únicas. Agentes declaram o que precisam — não instanciam:

```python
TOOL_REGISTRY = {
    "bigquery": Tool(instance=BigQueryTool(), ...),
    "tavily":   Tool(instance=TavilyTool(), ...),
    "yfinance": Tool(instance=YFinanceTool(), ...),
}

class DataAgent:
    required_tools = ["bigquery", "tavily"]

class ReportAgent:
    required_tools = ["bigquery", "yfinance"]  # mesma instância do bigquery
```

Sem duplicação de código, sem dois pools de conexão para a mesma tool.

### 5.4 State Acumulado do Supervisor

O supervisor mantém os outputs dos agentes já executados no state. O próximo agente recebe esse contexto automaticamente — sabe o que foi feito, não quem fez.

```python
class SupervisorState:
    input:            str
    agent_outputs:    dict   # {"agente_dados": "resultado..."}
    available_agents: list
    next_agent:       str
```

O agente não sabe que outros agentes existem. O supervisor é o único com visão global.

### 5.5 Handoff

Quando um agente percebe em runtime que precisa de uma **capacidade** que não tem, emite um `HandoffRequest` descrevendo a necessidade — não o nome do agente destino:

```python
HandoffRequest(
    capability="market_analysis",  # não "agente_dados"
    payload={"query": "..."}
)
```

O supervisor mapeia `capability → agente` e roteia. Agentes permanecem completamente desacoplados entre si.

**Regra de decisão:**

| Situação | Solução |
|---|---|
| Dois agentes precisam da mesma tool | Registry central, ambos declaram `required_tools` |
| Agente precisa de resultado de outro | Supervisor injeta via state acumulado |
| Agente precisa de capacidade que não tem | HandoffRequest via supervisor |
| Lógica duplicada entre agentes | Extrair para nova tool no registry |

---

## 6. Controle de Acesso a Tools

### 6.1 O Problema

Sem controle, o supervisor poderia injetar uma tool sensível em qualquer agente que a declarasse em `required_tools`. Era necessário uma segunda camada de controle além do IAM.

### 6.2 Duas Camadas Ortogonais

```
Camada 1 — IAM (Deny List)
  Controla: quais agentes o usuário pode acionar

Camada 2 — Tool Registry (allowed_agents)
  Controla: quais agentes podem receber a tool
```

As duas camadas são independentes. Uma não conhece as regras da outra.

### 6.3 Implementação

```python
TOOL_REGISTRY = {
    "tavily": Tool(
        instance=TavilyTool(),
        access_level="public"        # qualquer agente
    ),
    "bigquery_producao": Tool(
        instance=BigQueryProdTool(),
        access_level="restricted",
        allowed_agents=["agente_dados", "agente_relatorio"]
    ),
    "dados_rh": Tool(
        instance=HRTool(),
        access_level="restricted",
        allowed_agents=["agente_rh"]
    ),
}
```

Na montagem do agente, dois critérios precisam ser verdadeiros para a tool ser injetada:

```
Tool restrita é injetada somente se:
  1. agente está em tool.allowed_agents   (regra do sistema)
                  AND
  2. usuário tem o agente em available_agents  (regra do IAM)
```

### 6.4 Exemplos de Decisão

```
Analista + Agente Dados + bigquery_producao:
  allowed_agents contém agente_dados     ✅
  analista tem agente_dados disponível   ✅
  → tool injetada

Analista + Agente RH + dados_rh:
  allowed_agents contém agente_rh        ✅
  analista NÃO tem agente_rh (negado)    ❌
  → tool não injetada, agente nem é acionado

Agente Finanças + bigquery_producao:
  allowed_agents = [agente_dados, agente_relatorio]
  agente_financas não está na lista      ❌
  → tool não injetada independente do usuário
```

### 6.5 O Supervisor é o Último na Cadeia

O supervisor só orquestra o que já foi montado. Ele não tem como demandar uma tool que não foi injetada — ela simplesmente não existe no agente que ele aciona.

```
Deny List (IAM)   →  filtra quais agentes o usuário acessa
Tool Registry     →  filtra quais tools cada agente pode receber
build_agent()     →  monta o agente com as tools que passaram nos dois filtros
Supervisor        →  orquestra o que sobrou
```

---

## 7. Roadmap de Melhorias

### Fase 2 — Qualidade da Recuperação de Memória
- Threshold de similaridade em R1 (cosine ≥ 0.75)
- Router semântico antes de R1
- Atualização incremental do resumo de sessão

### Fase 3 — Integridade da Memória de Longo Prazo
- Merge/Dedup na escrita (upsert em vez de insert)
- Resolução de conflitos entre fatos contraditórios
- TTL/Decay — job periódico externo

### Fase 4 — Observabilidade
- OpenTelemetry ou Langfuse substituindo `_audit_write`
- Dashboard de memória para visualizar banco de longo prazo
- Métricas de custo por agente e por usuário

### Fase 5 — Paralelismo de Agentes
- Agentes Dados e Finanças rodando simultaneamente
- Agente consolidador juntando os outputs
- Modelo multi-agent paralelo do LangGraph

---

> Todos os componentes foram desenhados para extensão isolada. Nenhuma melhoria das fases acima exige reescrita do grafo principal — apenas adição de nós ou substituição de implementações internas.
