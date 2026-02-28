# Qorp Core: Unificação de Identidade (Omnichannel Handshake)

Este documento detalha a estratégia de vinculação de identidade entre diferentes canais (Web e WhatsApp), garantindo que a segurança e o contexto do usuário sejam preservados, independentemente de onde a conversa comece.

---

## 1. O Problema da Identidade Fragmentada

Em sistemas tradicionais, o usuário é uma pessoa no navegador (logado via Email/JWT) e outra pessoa no WhatsApp (identificada pelo número de telefone). Para o Qorp Core ser um assistente corporativo real, ele precisa de uma **Identidade Única (Single Source of Truth)**.

---

## 2. Estratégia de Handshake (Vínculo de Canal)

O Qorp Core utiliza um processo de "emparelhamento" para garantir que o número de telefone seja vinculado com segurança à conta corporativa do usuário.

### Fluxo de Vinculação Progressiva:
1.  **Registro na Web:** No portal administrativo ou perfil do usuário, o colaborador insere seu número de telefone.
2.  **Validação por Desafio (Challenge):** O Qorp Core gera um token temporário e envia para o WhatsApp do usuário.
3.  **Confirmação:** O usuário insere o token no portal Web ou responde ao bot com um comando específico (ex: "Confirmar acesso").
4.  **Mapeamento de Perfil:** O sistema grava no banco de dados a relação entre o `User_ID` (corporativo) e o `WhatsApp_ID` (MSISDN).

---

## 3. Mapeamento Centralizado de Identidade

O Core mantém uma tabela de identidade que atua como o "tradutor" para o Supervisor de Acesso:

| Atributo | Origem Web (JWT) | Origem WhatsApp (MSISDN) |
| :--- | :--- | :--- |
| **Identificador** | `usr_f92...` | `551199...` |
| **Autenticação** | Token JWT Assinado | Vínculo de Número Validado |
| **Role/Permissões** | `gerente_comercial` | `gerente_comercial` (herdado) |
| **Contexto/Memória** | Sessão `thread_01` | Sessão `thread_01` (Sincronizada) |

---

## 4. Continuidade de Experiência (Seamless Flow)

A unificação permite cenários que aumentam drasticamente a produtividade:

- **Follow-up Assíncrono:** "Peterson, terminei de processar o relatório de vendas que você pediu na Web. Acabei de enviar o PDF aqui no seu WhatsApp para facilitar."
- **Início Mobile, Finalização Desktop:** O usuário começa uma consulta de estoque no chão de fábrica via WhatsApp e, ao sentar no escritório, o chat da Web já contém o histórico e os gráficos gerados.

---

## 5. Segurança e Revogação

O vínculo de identidade é tratado como uma **Credencial de Segurança**:
- **Revogação Instantânea:** Se um colaborador sai da empresa, ao desativar sua conta no sistema central, o acesso via WhatsApp é cortado automaticamente em milissegundos.
- **Auditoria Omnichannel:** No Dashboard de Auditoria, é possível ver se uma ação foi disparada pelo celular ou pelo computador, mantendo a trilha de responsabilidade intacta.

---

## 6. Conclusão: O Usuário no Centro

Com a Unificação de Identidade, o Qorp Core deixa de ser um "bot de chat" e passa a ser uma **extensão da identidade digital do colaborador**. Isso permite que a IA tenha uma visão 360º das necessidades do usuário, respeitando sempre as fronteiras de acesso definidas pela empresa.

---
**Navegação:**
- [Contratos e Segurança](./CONTRATOS_E_SEGURANCA.md)
- [Supervisor de Acesso](./RASCUNHO_SUPERVISOR_ACESSO.md)
- [Dashboard de Auditoria](./DASHBOARD_AUDITORIA.md)
