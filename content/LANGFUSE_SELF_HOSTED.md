# Qorp Core - Langfuse Self-Hosted (Soberania de Dados)

Rodar o Langfuse na nossa própria infra não é só por "segurança", é pra garantir que o dono da empresa durma tranquilo sabendo que os dados estratégicos (vendas, margens e clientes) não estão passeando em nuvens de terceiros.

---

## 1. Por que rodar na nossa casa?
Para o **MVP Comercial**, a gente lida com faturamento e dados de leads.
- **Privacidade Real:** Nenhum trace de conversa sai da nossa rede.
- **Custo Fixo:** A gente paga o servidor, não o volume de mensagens. Se o comercial bombar e tiver 1 milhão de mensagens, o custo de observabilidade continua o mesmo.
- **Compliance:** Já nascemos preparados para empresas que exigem que os dados fiquem no Brasil ou em servidores próprios.

## 2. A Stack (O que precisamos subir)
O Langfuse não roda sozinho, ele vem com um "combo" via Docker Compose:
1.  **Langfuse Server:** O painel onde a gente mexe nos prompts e vê os traces.
2.  **PostgreSQL:** Onde ficam guardados todos os nossos logs e versões de prompts.
3.  **Redis:** Para deixar a interface rápida e não travar o orquestrador.

## 3. Como o Qorp Core conversa com ele
No nosso `.env`, a gente só aponta para o endereço local:
```bash
LANGFUSE_HOST="http://localhost:3000" # Endereço do container
LANGFUSE_PUBLIC_KEY="pk-lf-..."
LANGFUSE_SECRET_KEY="sk-lf-..."
```
O Orquestrador comercial vai mandar os dados direto para esse container via rede interna do Docker, o que deixa tudo muito mais rápido do que mandar para uma API externa.

## 4. Segurança e Backup
- **Acesso Restrito:** Só o nosso front de gestão e a gente (devs) acessa o painel do Langfuse.
- **Backup de Ouro:** O volume do PostgreSQL precisa de backup diário. Se a gente perder o banco do Langfuse, a gente perde o histórico de "inteligência" e as melhorias dos prompts.

---
**Navegação:**
- [Estratégia Langfuse](./ESTRATEGIA_LANGFUSE.md)
- [Dashboard de Gestão Técnica](./DASHBOARD_AUDITORIA.md)
- [Arquitetura Sistêmica](./ARQUITETURA_SISTEMICA.md)
