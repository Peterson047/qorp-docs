# Especificação: Arquitetura IAM Centralizada (Google SSO + WhatsApp)

Este documento detalha a estratégia de gestão de identidades em larga escala para o ecossistema Qorp, utilizando o Google Workspace como provedor de identidade (IdP) e um backend centralizado para orquestração de acessos omnichannel.

---

## 1. Visão Geral: O Fim da Fragmentação

Para evitar a redundância de dados e falhas de segurança ao gerenciar múltiplos sistemas (Metas, Administrativo, etc.), a arquitetura evolui para um modelo de **Identidade Federada Centralizada**.

*   **Fonte da Verdade:** Google Workspace (Corporate Domain).
*   **Gestor de Acesso:** Backend IAM (Identity & Access Management).
*   **Consumidor de Inteligência:** Qorp Core.

---

## 2. O Fluxo de Autenticação (SSO)

O usuário não possui credenciais locais na Qorp. Todo o acesso é validado pelo Google.

1.  **Login:** O colaborador acessa qualquer sistema (Web) e clica em "Entrar com Google".
2.  **Validação:** O Google confirma a identidade (`nome@empresa.com`) e o status da conta.
3.  **Sessão:** O IAM Central recebe a confirmação e emite um **JWT Único** que contém o cargo e setor do colaborador, sincronizado com a base de RH.

---

## 3. Vínculo de Canal (O Handshake do WhatsApp)

Como o número de telefone não é uma informação garantida pelo SSO do Google, o vínculo ocorre via um processo de pareamento seguro.

### O "Aperto de Mão" Digital:
1.  **Contexto Seguro:** Peterson loga via Google no Dashboard Web.
2.  **Challenge:** O sistema gera um código temporário vinculado à sua conta Google (ex: `QORP-GOOGLE-9921`).
3.  **Resposta:** Peterson envia o código para o número oficial da empresa no WhatsApp.
4.  **Registro de Vínculo:** O Backend IAM grava a relação:
    - **ID Mestre (Primary Key):** `peterson@empresa.com`
    - **Canal Vinculado:** `5511999999999`
    - **Status:** Verificado.

---

## 4. O Cérebro Omnichannel (Qorp)

Com o vínculo estabelecido, o Qorp Core funciona como um motor de inteligência que reconhece o usuário em qualquer lugar.

*   **No PC (Webchat):** O Qorp recebe o JWT assinado pelo Google/IAM.
*   **No WhatsApp (n8n/Gateway):** O Qorp recebe o número de telefone, pergunta ao IAM: *"Quem é o dono deste número?"*. O IAM responde: *"É o `peterson@empresa.com`, ele é do Comercial"*.

---

## 5. Revogação Instantânea (Kill Switch) 🚫

Este é o pilar de segurança máxima. O acesso ao Qorp é dependente da existência da conta corporativa.

- **Desligamento:** Se um colaborador sai da empresa, o TI desativa sua conta no Google Workspace.
- **Efeito Dominó:** 
    1. O login Web para de funcionar imediatamente.
    2. O IAM Central bloqueia qualquer nova emissão de token.
    3. O Qorp Core, ao consultar o IAM sobre mensagens vindas do WhatsApp, recebe a instrução de **Acesso Revogado**.
- **Resultado:** O acesso via WhatsApp é cortado automaticamente sem que ninguém precise apagar o número manualmente de uma lista.

---

## 6. Benefícios Estratégicos

1.  **Segurança Enterprise:** MFA (Multi-fator) e políticas de senha delegadas ao Google.
2.  **Manutenção Zero:** Mudanças de cargo no RH refletem automaticamente na IA.
3.  **Auditoria Centralizada:** Todos os logs de conversa são atrelados à conta Google oficial do colaborador.

---
**Navegação:**
- [Relações e Delegação](./RELACOES_SISTEMICAS_E_DELEGACAO.md)
- [Arquitetura Híbrida (JWT+IR)](./ARQUITETURA_HIBRIDA_IDENTIDADE.md)
- [Fluxo de Pareamento (Whats)](./FLUXO_VINCULACAO_IDENTIDADE.md)
