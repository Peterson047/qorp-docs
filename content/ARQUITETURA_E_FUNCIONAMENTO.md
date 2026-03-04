# Documentação Técnica: Qorp Langgraph

Este documento detalha a arquitetura, o funcionamento e as integrações do sistema **Qorp**, um assistente pessoal agêntico construído com LangGraph e FastAPI.

---

## 🏗️ Arquitetura do Sistema

O Qorp utiliza uma arquitetura baseada em **Grafos de Estado (State Graphs)**, permitindo fluxos de conversa complexos, persistência de memória e tomada de decisão dinâmica.

### 1. Núcleo Agêntico (LangGraph)
O coração do sistema reside em `src/graphs/qorp.py`, onde o fluxo de trabalho é definido:
- **`load_memory`**: Nó inicial que carrega o histórico e o perfil do usuário do MongoDB.
- **`agent`**: O cérebro (LLM - Gemini) que decide o próximo passo com base nas mensagens e ferramentas disponíveis.
- **`tools`**: Nó de execução de ferramentas (Busca Web, Finanças, CEP, etc.).
- **`update_profile`**: Nó final que analisa a conversa para extrair novos fatos e atualizar o perfil comportamental do usuário.

### 2. Camada de API (FastAPI)
Localizada em `src/api/`, provê endpoints para interação:
- **Endpoints V1**: `/api/v1/chat`, `/api/v1/threads`, `/api/v1/history`.
- **Streaming**: Suporte nativo a respostas via Stream para uma experiência de usuário fluida.
- **Lifespan**: Gerenciamento de inicialização do Grafo e conexões com observabilidade.

---

## ⚙️ Funcionamento e Fluxo de Dados

1. **Entrada**: O usuário envia uma mensagem via chat (Frontend Next.js).
2. **Identificação**: O sistema utiliza `user_id` e `thread_id` para buscar o contexto no MongoDB.
3. **Execução do Grafo**:
   - O estado atual é carregado.
   - O LLM analisa se precisa de ferramentas externas.
   - Se sim, executa a ferramenta e volta ao LLM.
   - Se não, gera a resposta final.
4. **Persistência**: Toda a conversa é salva via `MongoDBSaver`, garantindo que o Qorp lembre de interações passadas.
5. **Aprendizado Passivo**: Antes de encerrar, o nó `update_profile` refina as preferências do usuário silenciosamente.

---

## 🔌 Tecnologias e Conectividade

### 🧬 Modelos de IA
- **Principal**: `gemini-3-flash` (via Google Generative AI).
- **Embeddings**: Utilizados para busca vetorial na base de conhecimento.

### 🛠️ Ferramentas e Integrações
- **Busca Web**: Integração com `Tavily` para informações em tempo real.
- **Finanças**: Dados de ações e notícias via `yfinance`.
- **Localização**: Busca de endereços e CEPs via `ViaCEP`.
- **MCP (Model Context Protocol)**:
  - O Qorp é capaz de se conectar a servidores MCP externos.
  - Atualmente configurado para conectar-se ao **BigQuery** para análise de dados corporativos.
- **Busca Vetorial**: Base de conhecimento interna para respostas especializadas.

### 📊 Observabilidade e Infraestrutura
- **Banco de Dados**: `MongoDB` para persistência de chats e perfis.
- **Observabilidade**: Integração nativa com **Langfuse** para rastreamento de traces, latência e custos.
- **Ambiente**: Configurado via variáveis de ambiente (`.env`) para fácil deploy.

---

## 🆔 Gestão de Identidade e Permissões

Para garantir a escalabilidade e segurança em ambientes multi-canal, o Qorp utiliza uma estratégia de desacoplamento de identidade e regras de acesso baseadas em código.

### 1. Identity Registry & JWT (A Evolução da Identidade)
Para garantir a escalabilidade em ambientes multi-canal, o Qorp utiliza duas camadas de identidade:
- **Identity Registry**: Um mapeamento central (Internal UUID) que desvincula o usuário de canais específicos (WhatsApp, Web, Telegram).
- **JWT (Enterprise Path)**: O sistema suporta autenticação via **JWT com Criptografia Assimétrica**. O sistema Metas assina um "crachá digital" (Token) com escopos de acesso (`scope`), que o Qorp valida sem precisar consultar o banco de dados.

### 2. Autorização via Código (Hard Rules & Context Awareness)
Diferente de sistemas que confiam apenas no prompt, o Qorp aplica regras rígidas em Python:
- **Middleware de Permissões**: Valida se o perfil do usuário permite o uso de certas ferramentas.
- **Context Awareness (Origem)**: O sistema identifica se o acesso vem do `webchat` ou `whatsapp`. Em acessos móveis, ferramentas sensíveis são automaticamente ocultadas e a IA ajusta o tom para ser mais concisa.

---

## 🛡️ Segurança e Robustez
O sistema evoluiu de uma abordagem baseada em prompt para uma arquitetura de "Defesa em Camadas":
1. **Camada de Rede**: Chaves de API e validação de origem.
2. **Camada de Orquestração**: Supervisor filtrando agentes por permissão.
3. **Camada de Execução**: Código Python (Hard Rules) validando chamadas de ferramentas.
4. **Camada de Dados**: Isolamento de threads e contextos via `user_id` no MongoDB.
