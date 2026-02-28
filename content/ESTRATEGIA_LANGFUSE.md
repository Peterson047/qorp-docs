# Qorp Core - Langfuse na Prática

O Langfuse é o que vai permitir que a gente não fique louco tentando debugar o que a IA está fazendo. Ele é a nossa caixa preta: tudo o que o orquestrador e os agentes fizerem passa por aqui.

---

## 1. Onde ele entra no código
O Langfuse visualiza o "esqueleto" das nossas chamadas. Imagine ver isso aqui para cada pedido do cliente:

<img src="/images/langfuse/tracing-overview.png" width="600" style="border-radius: 8px; border: 1px solid #e2e8f0; margin: 20px 0;" alt="Traces Langfuse" />

No LangGraph, a gente injeta o Langfuse como um callback global. 
- **Trace de Grafo:** Dá pra ver o caminho exato: Usuário > Supervisor > Tool de Estoque > Resposta final. 
- **Gargalos:** Se o cliente reclamar de demora, a gente abre o Langfuse e vê qual nó do grafo está travando.

## 2. Gestão de Prompts (Sem deploy)
Essa é a parte mais importante pra gente. Ao invés de deixar os prompts hardcoded ou em arquivos locais, o Core busca eles no Langfuse.

<img src="/images/langfuse/prompts_v2.png" width="600" style="border-radius: 8px; border: 1px solid #e2e8f0; margin: 20px 0;" alt="Prompts Langfuse" />

- Se o agente comercial estiver sendo educado demais e não fechar venda, a gente edita o prompt na interface do Langfuse, salva e pronto.
- Tem rollback fácil se a gente fizer bobagem no prompt.

## 3. Olhando o bolso (Custos)
Como vamos começar pelo Comercial, o Langfuse vai taguear tudo.
- Dá pra saber exatamente quanto de dólar o orquestrador comercial gastou no dia.
- Ajuda a decidir se vale a pena trocar um modelo caro por um mais barato em tarefas simples.

<img src="/images/langfuse/langfuse-model-comparison.png" width="600" style="border-radius: 8px; border: 1px solid #e2e8f0; margin: 20px 0;" alt="Model Comparison Langfuse" />

## 4. Melhoria Contínua
- **Anotações:** Quando a IA falar groselha, a gente marca o erro lá.
- **Datasets:** Esses erros viram exemplos para a gente testar o código antes de subir uma versão nova.

---
**Navegação:**
- [Dashboard de Gestão Técnica](./DASHBOARD_AUDITORIA.md)
- [Arquitetura Sistêmica](./ARQUITETURA_SISTEMICA.md)
