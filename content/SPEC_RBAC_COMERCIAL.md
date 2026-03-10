# Especificação: RBAC Setor Comercial (V01)

Este documento define as regras de acesso, ferramentas permitidas e comportamento da IA para o cargo **Comercial**, servindo de guia para a implementação no Nexus (Admin) e no Identity Registry.

---

## 1. Definição do Perfil (Role)

*   **Identificador:** `role_comercial`
*   **Público:** Vendedores, Gerentes de Vendas e Analistas de Mercado.
*   **Objetivo:** Agilizar consultas de metas, análise de vendas no BigQuery e prospecção de mercado.

---

## 2. Matriz de Ferramentas (Toolbelt)

O Qorp Core deve filtrar dinamicamente as ferramentas MCP baseando-se nesta permissão:

| Ferramenta | Permissão | Descrição |
| :--- | :--- | :--- |
| **Google Search** | `allow` | Pesquisa livre sobre concorrentes e tendências. |
| **BigQuery (Vendas)** | `allow` | Consulta via linguagem natural no dataset de vendas. |
| **Metas API** | `allow` | Consulta de performance individual e de equipe. |
| **CRM Lookup** | `allow` | Consulta básica de histórico de clientes (Nome, Última Compra). |
| **Exportação CSV** | `deny (mobile)` | Permitido apenas no PC via Webchat. |

---

## 3. Comportamento e Personalidade (System Prompt)

Ao identificar que o usuário é do setor Comercial, o Qorp injeta as seguintes instruções:

> *"Você é o Qorp Especialista Comercial. Seja persuasivo, direto e focado em resultados (ROI). Ao apresentar números, tente sempre oferecer um insight de melhoria ou um comparativo com a meta estabelecida. Use emojis de forma moderada para manter o engajamento corporativo. 💼"*

---

## 4. Travas de Segurança e Origem (Context Awareness)

Regras aplicadas via código Python baseadas na origem da requisição:

### No Webchat (Desktop/Full Power):
- Acesso total às ferramentas listadas.
- Geração de tabelas Markdown completas.
- Permissão para gerar links de download de arquivos.

### No WhatsApp (Mobile/Safe Mode):
- **Ocultação de Tabelas:** Se o retorno do BigQuery for muito grande, o Qorp deve resumir em 3 bullet points principais.
- **Bloqueio de Dados Sensíveis:** Informações de margem de lucro líquida total da empresa não devem ser detalhadas (apenas tendências percentuais).
- **Concisão:** Respostas curtas para leitura rápida em movimento.

---

## 5. Implementação no Nexus (Admin UI)

A interface de gerenciamento no Nexus deve permitir:
1.  **Atribuição:** Botão para marcar o usuário Google com a tag `Comercial`.
2.  **Audit Trail:** Ver quais consultas esse usuário fez no BigQuery (via traces do Langfuse).
3.  **Override:** Capacidade de desativar temporariamente apenas a ferramenta de BigQuery de um usuário específico sem remover o cargo dele.

---
**Navegação:**
- [IAM Central: Google + Whats](./ARQUITETURA_IAM_SSO_WHATSAPP.md)
- [Relações e Delegação](./RELACOES_SISTEMICAS_E_DELEGACAO.md)
- [Identity Registry Spec](./IDENTITY_REGISTRY_SPEC.md)
