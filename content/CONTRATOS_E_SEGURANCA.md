# Qorp Core: Contratos e Segurança (Gatekeeping de Plugins)

Este documento define como o Qorp Core garante a integridade do sistema e o isolamento de dados. A arquitetura foi desenhada para iniciar com **um setor e tarefa específica**, mas com a fundação pronta para escalar para múltiplos departamentos e regras complexas de toda a empresa.

---

## 1. O Contrato de Plugin (Manifesto)

A base da escalabilidade do Qorp Core é o **Manifesto**. Mesmo que o sistema comece atendendo apenas uma tarefa (ex: Consulta de Estoque no Comercial), ele já nasce tratando essa tarefa como um "Plugin". Isso garante que, quando o RH ou o Financeiro entrarem, eles apenas sigam o mesmo contrato.

```json
{
  "plugin_id": "comercial_estoque_v01",
  "name": "Especialista em Estoque Comercial",
  "domain": "comercial",
  "allowed_roles": ["vendedor", "estoquista", "admin"],
  "tools": ["query_stock_api", "check_delivery_date"],
  "system_prompt": "Você é um assistente focado em precisão de estoque..."
}
```

---

## 2. Estratégia de Segurança Progressiva

Para o lançamento inicial (Setor Único), focamos em uma segurança eficiente que não adicione complexidade desnecessária, mas que já prepare o terreno para o uso corporativo total.

### Cenário A: Camada de Proteção Nativa (Simples e Direto)
Ideal para a fase inicial de uma tarefa específica.
- **Como funciona:** O Core utiliza **Instruções de Sistema Imutáveis** e **Filtros de Código**. 
- **O Gatekeeper:** Antes da IA processar a ferramenta, o código verifica: "Este usuário tem a Role necessária para esta tarefa específica?".
- **Vantagem:** Rapidez extrema na resposta e facilidade de depuração. É a base sólida para o primeiro setor.

### Cenário B: Camada de Proteção com ModelGuard (Segurança Empresarial)
Conforme o Qorp Core se expande para outros setores (RH, Financeiro) e manipula dados sensíveis de toda a empresa, ativamos o **ModelGuard**.
- **Como funciona:** Uma IA secundária (vigia) analisa se a intenção do usuário é burlar as fronteiras entre os setores. 
- **O Gatekeeper:** Ele atua como um "Firewall Cognitivo". Se um usuário do Comercial tentar perguntar sobre "Folha de Pagamento" (mesmo que por curiosidade), o ModelGuard bloqueia a intenção antes dela chegar a qualquer Agente Especialista.
- **Vantagem:** Previne vazamentos de dados entre departamentos e ataques de engenharia social (Prompt Injection) em larga escala.

---

## 3. Visão de Escala: Do Setor para a Empresa

| Fase | Escopo | Segurança Sugerida |
| :--- | :--- | :--- |
| **Fase 1: Lançamento** | 1 Setor / 1 Tarefa Crítica | **Cenário A** (Rígido via Código + Prompt) |
| **Fase 2: Expansão** | +2 Setores Interligados | **Cenário A + B** (Início da triagem de intenção) |
| **Fase 3: Enterprise** | Todos os Departamentos | **ModelGuard Ativo + Auditoria em Tempo Real** |

---

## 4. Conclusão: Pronto para o Amanhã

O diferencial do Qorp Core é que ele **não é um sistema engessado**. Ao definir o contrato via Manifesto desde o dia 1, garantimos que a expansão para novos setores seja apenas uma questão de configuração, mantendo a segurança sempre um passo à frente da complexidade.

---
**Navegação:**
- [Visão de Produto Core](./VISAO_PRODUTO_CORE.md)
- [Arquitetura Sistêmica](./ARQUITETURA_SISTEMICA.md)
- [Blueprint de Funcionamento](./BLUEPRINT_FUNCIONAMENTO.md)
