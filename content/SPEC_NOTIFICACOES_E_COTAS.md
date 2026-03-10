# Especificação: Operação, Cotas e Notificações Proativas

Este documento detalha as regras de sustentabilidade financeira e engajamento proativo do Qorp, garantindo que o sistema seja eficiente em termos de custo e útil além da interação passiva.

---

## 1. Gestão de Cotas e Rate Limiting (Proteção de Custo)

Para evitar abusos e controlar o custo de processamento (LLM + Ferramentas), o IAM Central deve gerenciar cotas baseadas em cargos.

| Cargo | Cota Diária (MSGs) | Cota Ferramentas (Tool Calls) | Prioridade de Processamento |
| :--- | :--- | :--- | :--- |
| **Comercial** | 100 | 50 | Alta |
| **Administrativo** | 50 | 20 | Média |
| **Visitante/Guest** | 10 | 0 (Bloqueado) | Baixa |

### Mecanismo de Enforcement:
1.  **Check:** A cada requisição, o Qorp Core consulta o IAM: *"Usuário X atingiu o limite?"*.
2.  **Bloqueio Semântico:** Se atingir, a IA responde educadamente: *"Peterson, você atingiu seu limite diário de análises profundas. Podemos continuar com conversas simples, ou você pode pedir um upgrade de cota no Nexus."*

---

## 2. Notificações Proativas (Outbound AI) 📣

O Qorp deixa de ser apenas reativo e passa a iniciar conversas relevantes via WhatsApp (via n8n).

### Fluxo de Notificação:
1.  **Evento:** Um gatilho ocorre (ex: Uma meta importante foi batida ou um relatório longo terminou de ser processado).
2.  **Permissão:** O Qorp verifica no Identity Registry se o usuário tem o provider `whatsapp` verificado e se a flag `allow_proactive_notifications` está ativa.
3.  **Entrega:** O Qorp envia um webhook para o n8n: 
    - `target_id`: `5511...`
    - `message`: *"Oi Peterson, aquele relatório de ROI que você pediu no PC está pronto! 📊"*

---

## 3. Retenção de Dados e Privacidade (LGPD) 🔐

Regras para o gerenciamento das "Memórias" do usuário.

### Expurgo Automático:
*   **Conversas:** Histórico de chat é movido para "frio" após 90 dias e deletado após 180 dias.
*   **Memória Comportamental:** O resumo do perfil (`user_profile`) é persistente, a menos que o usuário seja removido.

### Direito ao Esquecimento:
Quando um usuário é removido via Nexus:
1.  O IAM sinaliza o Qorp.
2.  O Qorp dispara uma tarefa de limpeza (`Purge Task`) que apaga:
    - Todos os registros na coleção `user_memory`.
    - Todos os checkpoints de thread no MongoDB.
    - O vínculo no Identity Registry.

---

## 4. Monitoramento no Nexus (Admin UI)

O painel administrativo Nexus deve exibir:
- **Consumo de Tokens:** Top 10 usuários que mais consomem processamento.
- **Taxa de Notificações:** Quantas msgs proativas foram abertas/respondidas.
- **Health Check MCP:** Status de conexão com os servidores de ferramentas externos.

---
**Navegação:**
- [RBAC: Setor Comercial](./SPEC_RBAC_COMERCIAL.md)
- [IAM Central: Google + Whats](./ARQUITETURA_IAM_SSO_WHATSAPP.md)
- [Arquitetura e Funcionamento](./ARQUITETURA_E_FUNCIONAMENTO.md)
