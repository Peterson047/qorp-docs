# Qorp Core - Análise de Riscos Técnicos (V1)

Mapeamento de gargalos e vulnerabilidades críticas na arquitetura de orquestração de agentes.

---

## 1. Latência de Roteamento (Supervisor Overhead)
A introdução de uma camada de decisão inicial (Supervisor) adiciona latência fixa em cada turno da conversa.
- **Risco:** Perda de retenção de usuários no WhatsApp se o TTFT (Time To First Token) exceder 5 segundos.
- **Mitigação:** Utilizar modelos de baixa latência (Gemini 1.5 Flash / GPT-4o-mini) dedicados exclusivamente ao roteamento de plugins.

## 2. Fragmentação de Contexto (State Drift)
Dificuldade em manter a coerência das variáveis de estado entre plugins isolados.
- **Risco:** Perda de informações críticas (ex: descontos aprovados, dados de lead) ao trocar de agente orquestrador.
- **Mitigação:** Implementação de um `Shared State` persistente via Checkpoints do LangGraph, garantindo acesso universal a variáveis de contexto essenciais.

## 3. Falha de Classificação (Re-routing Failure)
Incapacidade do Supervisor em identificar corretamente a intenção do usuário ou erro na seleção do plugin especialista.
- **Risco:** Redirecionamento para agentes sem as ferramentas (tools) necessárias para resolver a demanda.
- **Mitigação:** Protocolo de `Back-Routing` onde o Especialista devolve o controle ao Supervisor caso detecte falta de escopo ou ferramentas.

## 4. Disponibilidade de Infraestrutura (Service Dependency)
Dependência crítica de serviços de suporte (Redis para sessões, PostgreSQL para checkpoints e Langfuse para observabilidade).
- **Risco:** Interrupção total do processamento de mensagens em caso de falha em qualquer container da stack Docker.
- **Mitigação:** Implementação de políticas de `Retry` no orquestrador e monitoramento ativo com alertas de saúde do sistema.

## 5. Falsos Positivos de Segurança (Guardrail Latency)
Filtros de Prompt Injection e RBAC que bloqueiam requisições legítimas por excesso de restrição.
- **Risco:** Impacto negativo na utilidade do agente para o usuário final (vendedores e leads).
- **Mitigação:** Ajuste dinâmico de sensibilidade via Dashboard de Gestão (Arquivo 12) e auditoria manual de requisições bloqueadas no Langfuse.

---
**Navegação:**
- [Estratégia de Implementação](./ESTRATEGIA_IMPLEMENTACAO.md)
- [Arquitetura Sistêmica](./ARQUITETURA_SISTEMICA.md)
- [Estimativa de Latência](./ESTIMATIVA_LATENCIA.md)
