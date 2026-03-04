# Especificação Técnica: Identity Registry (IR)

O **Identity Registry** é o componente central de autenticação e autorização do Qorp. Ele serve como a "Fonte Única de Verdade" para mapear identidades de múltiplos canais para uma conta de usuário interna única.

---

## 1. Modelo de Dados (Schema MongoDB)

As identidades são armazenadas na coleção `identities`.

```json
{
  "internal_user_id": "qorp_u_8fb2c9",
  "name": "Peterson",
  "created_at": "2026-03-04T00:00:00Z",
  "status": "active",
  "providers": [
    {
      "provider": "web_auth",
      "external_id": "peterson@empresa.com",
      "verified": true,
      "last_seen": "2026-03-04T00:45:00Z"
    },
    {
      "provider": "whatsapp",
      "external_id": "5511999999999",
      "verified": true,
      "paired_at": "2026-03-01T10:00:00Z"
    }
  ],
  "access_control": {
    "role": "admin",
    "sector": "comercial",
    "permissions": ["read_inventory", "write_sales", "execute_mcp_bigquery"]
  },
  "metadata": {
    "preferred_language": "pt-BR",
    "timezone": "America/Bahia"
  }
}
```

---

## 2. Fluxo de Handshake (Vínculo de Canal)

Para garantir que o `external_id` (ex: WhatsApp) realmente pertence ao usuário, seguimos o fluxo de **Challenge-Response**:

1.  **Request:** O usuário (já logado na Web) solicita "Vincular WhatsApp".
2.  **Challenge:** O sistema gera um `pairing_code` (6 dígitos) e salva no Redis/Cache por 5 minutos.
3.  **Action:** O usuário envia esse código para o número oficial do Qorp no WhatsApp.
4.  **Verification:** O backend do WhatsApp recebe o código, valida contra o Cache e adiciona o objeto `whatsapp` na lista de `providers` do usuário.

---

## 3. Integração com o Grafo (LangGraph)

O Identity Registry é consultado no início de cada interação no nó `load_memory`:

1.  **Input:** A API recebe um `user_id` (que pode ser o e-mail ou número de telefone).
2.  **Lookup:** O sistema busca na coleção `identities` qualquer objeto que contenha esse ID na lista de `providers`.
3.  **Injeção:** O `internal_user_id` e as `permissions` são injetados no `AgentState`.
4.  **Enforcement:** O nó do Agente e as ferramentas (MCP) usam essas permissões para decidir o que mostrar ou executar.

---

## 4. Benefícios de Segurança

*   **Isolamento Real:** Se o WhatsApp for clonado, a revogação do provider no Registry corta o acesso imediatamente, sem afetar o login Web.
*   **Zero Semantic Risk:** As permissões são validadas por código Python (Hard Rules) comparando a lista de `permissions` do Registry, ignorando qualquer tentativa de "convencimento" via prompt.
*   **Audit Trail:** Todas as ações são logadas com o `internal_user_id`, permitindo saber exatamente quem fez o quê, independente do canal utilizado.

---
**Navegação:**
- [Unificação de Identidade](./UNIFICACAO_DE_IDENTIDADE.md)
- [Contratos e Segurança](./CONTRATOS_E_SEGURANCA.md)
- [Arquitetura e Funcionamento](./ARQUITETURA_E_FUNCIONAMENTO.md)
