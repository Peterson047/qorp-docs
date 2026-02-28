# Qorp Core - Roadmap de Finalização (V1)

Lista técnica de lacunas críticas para atingir o estado de Produção no Setor Comercial.

---

## 1. Infraestrutura e Persistência
- [ ] **LangGraph Checkpointer (Postgres):** Migrar o histórico de memória de "In-Memory" para persistência real em banco relacional.
- [ ] **Redis Lock System:** Implementar travas de sessão para evitar que duas mensagens do mesmo usuário (clique duplo no WhatsApp) processem o grafo simultaneamente.

## 2. Orquestração e Plugins
- [ ] **Plugin Hot-Reload:** Finalizar a lógica que lê o `MANIFESTO.json` do banco e atualiza os nós do grafo em tempo de execução.
- [ ] **Tool Parallelization:** Configurar o executor assíncrono para disparar chamadas de ferramentas (ex: Estoque + CRM) de forma concorrente.

## 3. Segurança (RBAC)
- [ ] **JWT Mapping:** Vincular o `wa_id` (WhatsApp) a um token JWT válido para consultar permissões no Supervisor de Acesso.
- [ ] **Guardrail Model:** Implementar um nó de "Audit" final que verifica se a resposta da IA contém dados sensíveis vazados.

## 4. Observabilidade
- [ ] **Langfuse SDK:** Mapear todos os `span_ids` para garantir que o Trace no Langfuse reflita exatamente a estrutura do grafo de agentes.
- [ ] **Dashboard de Gestão:** Criar as rotas de API para os botões de `COMMIT/ROLLBACK` de ferramentas no front de dev.

---
**Navegação:**
- [Visão Geral do Produto](./VISAO_PRODUTO_CORE.md)
- [Análise de Riscos](./ANALISE_DE_RISCOS.md)
