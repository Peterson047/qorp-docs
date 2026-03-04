# Especificação: Fluxo de Vinculação de Identidade (Handshake)

Este documento detalha o processo técnico e lógico de como um usuário do **Sistema Metas** vincula sua conta corporativa a canais externos, como o **WhatsApp**, garantindo uma identidade unificada e segura.

---

## 1. A Arquitetura de "Ponte"

Para que o sistema seja flexível e seguro, utilizamos três camadas de identificação:
1.  **ID Mestre (Internal UUID):** A chave primária única dentro do Qorp (ex: `qorp_u_123`).
2.  **Corporate ID (Metas ID):** O identificador do usuário no sistema legado/mestre (ex: `metas_789`).
3.  **Provider ID (Channel ID):** O identificador no canal de comunicação (ex: `551199...` para WhatsApp).

O **Identity Registry (IR)** é o componente responsável por manter o mapa que liga essas três pontas.

---

## 2. Passo a Passo do Handshake (O Pareamento)

O processo de pareamento é o "aperto de mão" que garante que o dono do número de telefone é o mesmo dono da conta corporativa.

### Passo 1: Solicitação no Metas
O usuário logado no Dashboard do Metas acessa a tela de configurações de IA e clica em **"Vincular WhatsApp"**.
- O backend do Metas gera um token aleatório de 6 dígitos (`Challenge Code`).
- O código é salvo temporariamente no Redis com um tempo de expiração (ex: 5 minutos), associado ao `internal_user_id`.

### Passo 2: Ação no WhatsApp
O usuário envia o código (ex: `QORP-8821`) para o número oficial da empresa no WhatsApp.
- O **n8n** (ou Gateway) recebe a mensagem e a encaminha para o Qorp.
- O Qorp identifica que a mensagem contém um padrão de código de pareamento.

### Passo 3: Validação e Registro
O Qorp verifica no cache se o código `QORP-8821` existe e a quem ele pertence.
- Se válido, o Qorp cria uma nova entrada (ou atualiza a existente) no Identity Registry:
  ```json
  {
    "internal_user_id": "qorp_u_123",
    "providers": [
      { "type": "web", "id": "metas_789" },
      { "type": "whatsapp", "id": "5511999999999", "verified": true }
    ]
  }
  ```
- O Qorp envia uma confirmação no WhatsApp: *"Tudo pronto, Peterson! Agora eu te reconheço por aqui."*

---

## 3. Fluxo Operacional (n8n + Qorp)

Após o vínculo, o n8n atua apenas como um "Carteiro Inteligente":

1.  **Incoming:** Peterson manda msg no Whats.
2.  **Dispatch:** O n8n envia para o Qorp: `POST /chat { "user_id": "55119...", "message": "Qual minha meta?" }`.
3.  **Resolve:** O Qorp consulta o Registry e descobre que o `user_id` do Whats pertence ao `internal_user_id` do Peterson.
4.  **Action:** O Agente agora tem acesso a toda a memória e permissões do Peterson no Metas para responder a pergunta com precisão.

---

## 4. Ciclo de Vida e Segurança

### Desativação (Offboarding)
Se um colaborador sai da empresa, o processo é imediato:
1.  O **Metas** desativa a conta do usuário.
2.  Uma trigger ou webhook avisa o **Qorp** para desativar o `internal_user_id`.
3.  Qualquer mensagem vinda do WhatsApp vinculado será ignorada ou respondida com uma mensagem de "Acesso Revogado".

### Troca de Número
O usuário pode solicitar a revogação do provider `whatsapp` e iniciar um novo handshake a qualquer momento pelo portal Web, garantindo que o controle da identidade esteja sempre no sistema mestre (Metas).

---
**Navegação:**
- [Guia de Integração: Sistema Metas](./INTEGRACAO_SISTEMA_METAS.md)
- [Identity Registry Spec](./IDENTITY_REGISTRY_SPEC.md)
- [Arquitetura e Funcionamento](./ARQUITETURA_E_FUNCIONAMENTO.md)
