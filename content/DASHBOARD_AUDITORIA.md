# Qorp Core - Interface de Gestão Técnica (Dev-First)

Este dashboard é uma camada de **configuração e intervenção em tempo real**. A análise de performance, pipeline e traces detalhados ficam delegados ao **Langfuse**. Aqui, o foco é o controle do comportamento do sistema.

---

## 1. Gestão de State & Checkpoints (LangGraph)
Como o Langfuse já cuida do histórico, esta interface foca no **Controle do Grafo**:
- **Checkpoint Viewer:** Visualizar o estado atual das variáveis do grafo (Shared State) para depurar por que a IA tomou uma decisão específica.
- **State Injection:** Interface para o Dev editar manualmente um valor no estado da conversa (ex: resetar a variável `attempts` ou alterar o `current_plugin_id`) para destravar um fluxo.
- **Thread Management:** Listagem técnica de Threads ativas com ID interno do banco.

## 2. Configuração Dinâmica (Hot-Reload)
Interface para gerenciar o que a IA **pode** fazer, sem mexer no código:
- **Editor de Manifesto:** Interface para ativar/desativar ferramentas (Tools) e plugins de forma atômica.
- **Overriders de Negócio:** Campos para alterar variáveis críticas (ex: `max_discount_allowed`, `tax_calculation_mode`) que são injetadas no prompt via System Message.
- **Prompt Hot-Fix:** Um campo para injetar uma "Instrução de Emergência" global que o Supervisor lê em todas as chamadas até que o bug seja corrigido no código.

## 3. Human-in-the-Loop: Centro de Decisão
Foco exclusivo em ações que exigem autorização:
- **Fila de Aprovação:** Lista técnica de chamadas de Tool que estão em `PENDING`.
- **Botões de Resposta:** `[COMMIT]`, `[ROLLBACK]` ou `[RE-EXECUTE WITH MODIFIED PARAMS]`.

## 4. Integração Langfuse
- **Deep Link por Thread:** Cada conversa no dashboard possui um link direto para o respectivo Trace no Langfuse.
- **Sincronia:** O dashboard utiliza o `traceId` do Langfuse como chave primária para garantir que a auditoria técnica e a intervenção de negócio falem a mesma língua.

---
**Navegação:**
- [Próximos Passos (O Que Falta?)](./O_QUE_FALTA.md)
- [Contratos e Segurança](./CONTRATOS_E_SEGURANCA.md)
