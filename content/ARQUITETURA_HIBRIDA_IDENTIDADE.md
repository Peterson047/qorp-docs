# Especificação: Arquitetura Híbrida de Identidade (JWT + Registry)

Este documento detalha o modelo híbrido de autenticação do Qorp, projetado para equilibrar a segurança máxima em ambientes Web com a agilidade necessária em canais externos como o WhatsApp.

---

## 1. Por que uma Arquitetura Híbrida?

Diferentes canais de comunicação possuem diferentes capacidades técnicas e riscos de segurança:
- **Web/PC:** Permite o uso de cabeçalhos seguros e sessões autenticadas.
- **WhatsApp/n8n:** Funciona via webhooks simples, onde a segurança reside na confiança entre o Gateway e o Core.

A solução é o **Modelo Híbrido**: O Qorp aceita tanto um "Crachá Digital" (JWT) quanto uma "Consulta de Lista" (Registry).

---

## 2. Cenário A: Alta Confiança (Web/PC) via JWT 🎫

No ambiente corporativo logado (Metas ou Administrativo), utilizamos o fluxo de **Identidade Federada via JWT**.

1.  **Emissão:** O sistema mestre assina um token contendo as *Claims* (Identidade, Cargo, Escopo).
2.  **Envio:** O frontend anexa o token em cada requisição (`Authorization: Bearer <JWT>`).
3.  **Validação:** O Qorp valida a assinatura usando a **Chave Pública** do mestre.
4.  **Processamento:** A identidade é extraída diretamente do token. 
    - **Vantagem:** O Qorp não precisa consultar o banco de dados para saber quem é o usuário. É instantâneo e ultra-seguro.

---

## 3. Cenário B: Canais Externos (WhatsApp/n8n) via Registry 📇

Para o n8n e gateways de chat, o uso de JWT assinado seria tecnicamente verboso e complexo. Aqui, entra o **Identity Registry (IR)**.

1.  **Entrada:** O n8n envia apenas o identificador do canal (ex: número de telefone).
2.  **Lookup:** O Qorp recebe o número e consulta a tabela `identities` no MongoDB.
3.  **Resolução:** O Registry "traduz" o número de telefone para o `Internal_UUID` e carrega as permissões associadas.
4.  **Processamento:** O Qorp injeta o contexto e responde via Gateway.
    - **Vantagem:** O n8n permanece simples (apenas um HTTP Request com API Key). A complexidade de "quem é quem" fica isolada no Core.

---

## 4. O Elo de Ligação: O Handshake de Pareamento 🤝

O que torna o modelo híbrido coeso é o momento em que o **Cenário A encontra o Cenário B**.

*   **O Vínculo:** Para que o Registry saiba que o "WhatsApp X" é o "Usuário Y", o Peterson deve estar logado no PC (Cenário A - Seguro) para autorizar o vínculo do seu telefone (Cenário B).
*   **A Sincronização:** Uma vez feito o pareamento, o Registry guarda o vínculo permanentemente. A partir daí, o Peterson ganha a conveniência do WhatsApp com a segurança configurada no Metas.

---

## 5. Matriz de Comparação Técnica

| Característica | Fluxo JWT (Web) | Fluxo Registry (Whats) |
| :--- | :--- | :--- |
| **Ponto de Verificação** | Memória (Matemática) | Banco de Dados (IO) |
| **Dependência do Mestre** | Nenhuma (Stateless) | Alta (Stateful) |
| **Ideal para...** | Sistemas Internos / Dashboards | n8n / WhatsApp / Automações |
| **Complexidade no Client** | Alta (Precisa assinar/gerir token) | Baixa (Apenas envia ID) |
| **Nível de Poder** | Full Power (PC) | Context Aware (Mobile/Limited) |

---

## 6. Exemplos de Representação de Dados

Para visualizar como a identidade "viaja" entre os sistemas, aqui estão as estruturas de dados JSON:

### A. Registro Inicial (Origem: Sistema Metas)
Enviado via `POST /api/v1/identity/sync` no momento do cadastro ou login inicial.

```json
{
  "internal_user_id": "qorp_u_8fb2c9",
  "name": "Peterson",
  "corporate_data": {
    "system_origin": "metas_v3",
    "external_id": "metas_789",
    "email": "peterson@empresa.com"
  },
  "access_control": {
    "role": "admin",
    "sector": "comercial",
    "scope": "comercial_full_access"
  }
}
```

### B. Documento no Registry (Após vínculo via WhatsApp/n8n)
O estado final do documento no MongoDB da Qorp, permitindo a resolução via número de telefone.

```json
{
  "internal_user_id": "qorp_u_8fb2c9",
  "name": "Peterson",
  "status": "active",
  "providers": [
    {
      "type": "web",
      "id": "metas_789",
      "verified": true
    },
    {
      "type": "whatsapp",
      "id": "5511999999999",
      "verified": true,
      "paired_at": "2026-03-04T01:40:00Z"
    }
  ],
  "preferences": {
    "origin_rules": {
      "whatsapp": { "limit_mcp": true, "concise_mode": true },
      "webchat": { "limit_mcp": false, "concise_mode": false }
    }
  }
}
```

---
**Conclusão:** 
O modelo híbrido garante que o Qorp seja **robusto o suficiente para a TI** (via JWT) e **ágil o suficiente para o Negócio** (via Registry/n8n). É a arquitetura que permite escalar o sistema Metas para toda a empresa sem criar gargalos de integração. 🤪🚀

---
**Navegação:**
- [Relações Sistêmicas e Delegação](./RELACOES_SISTEMICAS_E_DELEGACAO.md)
- [Identity Registry Spec](./IDENTITY_REGISTRY_SPEC.md)
- [Integração: Sistema Metas](./INTEGRACAO_SISTEMA_METAS.md)
