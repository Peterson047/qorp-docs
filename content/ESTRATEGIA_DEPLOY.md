# Qorp Core - Estratégia de Deploy (Simplificada)

Para manter a agilidade do MVP Comercial, o foco do deploy é **isolamento e reprodutibilidade** usando containers.

---

## 1. Stack de Deploy (Docker Compose)
A forma mais fácil e robusta de subir o projeto completo é através de um arquivo `docker-compose.yml`. Ele garante que toda a infraestrutura suba com um único comando.

### Serviços no Container:
- **`qorp-api`**: O backend FastAPI com o orquestrador LangGraph.
- **`qorp-front`**: O dashboard de gestão técnica (Vite/Static).
- **`langfuse`**: Instância self-hosted para observabilidade.
- **`postgres`**: Banco de dados para checkpoints e logs do Langfuse.
- **`redis`**: Cache de sessões e locks.

---

## 2. Opções de Hospedagem

### A. VPS Tradicional (DigitalOcean / Hetzner / AWS Lightsail)
- **Como:** Criar uma máquina Linux (Ubuntu), instalar Docker e rodar o compose.
- **Prós:** Controle total, custo fixo e soberania de dados garantida.
- **Contras:** Exige configuração manual de firewall e certificados (Nginx/Certbot).

### B. PaaS Moderno (Coolify - Recomendado)
O **Coolify** é uma alternativa open-source ao Heroku/Vercel que você instala na sua própria VPS.
- **Como:** Você aponta para o seu repositório no GitHub, e ele faz o build e deploy automático de todos os containers.
- **Prós:** Interface visual, SSL (HTTPS) automático, backups inclusos e extremamente fácil de gerenciar.

---

## 3. Fluxo de CI/CD (Pipeline)
1. **Push no GitHub:** Você sobe o código novo.
2. **Webhooks:** O Coolify (ou um script simples na VPS) detecta a mudança.
3. **Hot-Reload:** O container da API reinicia com o novo grafo, enquanto o Langfuse (que é self-hosted) mantém todos os dados de prompts e traces intactos.

---

## 4. Comandos Rápidos de Operação
```bash
# Subir tudo em background
docker-compose up -d

# Ver logs do orquestrador em tempo real
docker logs -f qorp-api

# Reiniciar apenas a inteligência (API) após ajuste
docker-compose restart qorp-api
```

---
**Navegação:**
- [O Que Falta?](./O_QUE_FALTA.md)
- [Guia de Introdução](./GUIA_SOBREVIVENCIA_IA.md)
