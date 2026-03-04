# Guia de Integração: Sistema Metas & Qorp Core (Arquitetura JWT)

Este documento detalha o modelo de integração **Enterprise** entre o sistema **Metas** (Mestre de Identidade) e o **Qorp Core** (Motor de Inteligência), utilizando autenticação baseada em **JWT (JSON Web Tokens)** e criptografia assimétrica.

---

## 1. O Modelo de Confiança (Crachá Digital)

Nesta arquitetura, o Qorp não possui um banco de dados de usuários local para autenticação. Ele confia nas **Claims** (afirmações) contidas em um token assinado pelo sistema Metas.

*   **Metas (Issuer):** Possui a **Chave Privada**. Ele é o único capaz de emitir crachás (Tokens).
*   **Qorp (Validator):** Possui a **Chave Pública**. Ele consegue ler e verificar a autenticidade do crachá, mas não consegue emitir novos.

---

## 2. Estratégia de Escopos (Evitando Verbosidade)

Para evitar que o Token fique gigante com centenas de permissões, utilizamos o conceito de **Scopes (Escopos)**. O Metas agrupa permissões granulares em "títulos" que o Qorp entende.

**Exemplo de Payload do JWT:**
```json
{
  "sub": "qorp_u_8fb2c9",          // ID Interno Mestre
  "name": "Peterson",
  "scope": "comercial_master",    // Em vez de listar 50 permissões, usamos uma Role
  "origin": "web",                // Define o nível de poder (PC vs Mobile)
  "iat": 1712345678,              // Data de emissão
  "exp": 1712350000               // Expiração (Curta duração para segurança)
}
```

---

## 3. Implementação no Backend (Lado do Metas)

O backend do Metas é responsável por gerar o token quando o usuário faz login ou quando uma mensagem chega via canal externo (WhatsApp).

### Exemplo em Python (Geração de Token Assinado)

```python
import jwt # PyJWT
from datetime import datetime, timedelta

PRIVATE_KEY = open("metas_private_key.pem").read()

def generate_qorp_token(user_data, origin="web"):
    """Gera um crachá digital assinado para o Peterson falar com o Qorp."""
    
    payload = {
        "sub": user_data['qorp_user_id'],
        "name": user_data['nome'],
        "scope": user_data['perfil_slug'], # Ex: 'vendas_especialista'
        "origin": origin,
        "exp": datetime.utcnow() + timedelta(hours=1) # Token expira em 1h
    }
    
    # Assinatura usando algoritmo RS256 (Asimétrico)
    token = jwt.encode(payload, PRIVATE_KEY, algorithm="RS256")
    return token
```

---

## 4. Integração do Chat Interno (Frontend)

No Dashboard do Metas (Next.js/React), a comunicação com o Qorp acontece de forma direta e segura.

### O Fluxo de Comunicação:
1.  O Frontend do Metas solicita o `qorp_token` ao seu próprio backend.
2.  O componente de chat envia as mensagens diretamente para a API do Qorp.
3.  **Auth Header:** O token é enviado no cabeçalho `Authorization: Bearer <TOKEN>`.

### Benefícios:
*   **Zero Latência de Auth:** O Qorp valida o token localmente em microssegundos (matemática pura), sem precisar consultar o banco de dados.
*   **Isolamento Total:** Se o banco de dados do Metas estiver lento, o chat do Qorp continua funcionando normalmente porque a identidade está "no bolso" do usuário.

---

## 5. Handover de Identidade (WhatsApp)

Para mensagens vindas do WhatsApp (via n8n ou Gateway):
1.  O Gateway identifica o número do remetente.
2.  O Gateway pergunta ao Metas: *"Quem é o dono do número 551199...?"*.
3.  O Metas responde com os dados do usuário.
4.  O Gateway gera um **JWT temporário** (usando a mesma lógica do item 3) com o `origin: "whatsapp"`.
5.  O Qorp recebe o token, vê que a origem é móvel e aplica as restrições de segurança automaticamente.

---
**Navegação:**
- [Identity Registry Spec](./IDENTITY_REGISTRY_SPEC.md)
- [Contratos e Segurança](./CONTRATOS_E_SEGURANCA.md)
- [Arquitetura e Funcionamento](./ARQUITETURA_E_FUNCIONAMENTO.md)
