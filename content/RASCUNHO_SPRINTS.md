### Sprint 1: Fundação e Infraestrutura Base (Setup e Persistência)
**Foco:** Estabelecer a base do sistema, migrando arquiteturas temporárias para soluções de produção relacional e cache.
*   **Deploy da Infraestrutura Local (Docker Compose):** Subir os containers do backend FastAPI (qorp-api), Postgres, Redis e Langfuse Self-Hosted.
*   **Persistência de Estado (LangGraph):** Migrar o `Checkpointer` da memória volátil (In-Memory) para persistência real utilizando o PostgreSQL, garantindo que as sessões possam ser retomadas sem perda de contexto.
*   **Lock de Sessão (Redis):** Implementar um sistema de travas (locks) no Redis para evitar que requisições duplicadas (como o "clique duplo" do usuário no WhatsApp) processem o grafo ao mesmo tempo.
*   **Setup da Observabilidade:** Configurar a comunicação interna na rede Docker para que o orquestrador envie todos os dados e métricas para o Langfuse.

### Sprint 2: Identidade e Segurança (Gateway e Omnichannel)
**Foco:** Estruturar o reconhecimento de usuários ("Dono do Mapa") e os mecanismos de segurança (RBAC).
*   **Identity Registry (SSOT):** Criar a coleção `identities` no MongoDB que funcionará como a "Fonte Única de Verdade" para mapear os canais do usuário (ex: vinculando o ID do WhatsApp ao ID interno do Metas).
*   **Autenticação JWT (Sistema Metas):** Implementar a validação de Tokens JWT assinados com criptografia assimétrica, garantindo autenticação sem latência de banco de dados no ambiente Web.
*   **Handshake do WhatsApp:** Desenvolver o fluxo de "Challenge-Response", onde o usuário gera um código de pareamento no sistema mestre e envia no WhatsApp para vincular seu número de telefone à conta corporativa de forma segura.
*   **Mapeamento de Acesso:** Vincular as identidades de canais externos (wa_id) para injetar as permissões de acesso (roles) no Supervisor, filtrando ferramentas e domínios antes mesmo da IA processar o pedido.

### Sprint 3: Orquestração e Raciocínio (Agente e Latência)
**Foco:** Construir o fluxo cognitivo inicial da inteligência artificial e otimizar o tempo de resposta.
*   **Arquitetura Monoagente Modular:** Desenvolver inicialmente o motor do LangGraph usando um agente com identidade fluida e ferramentas filtradas (ao invés de múltiplos agentes competindo), facilitando o debug inicial.
*   **Otimização de Latência (Streaming):** Habilitar a resposta via *Streaming* nativo no FastAPI para que a resposta seja exibida em tempo real no frontend, reduzindo o tempo percebido pelo usuário (TTFT).
*   **Implementação de Modelos Mistos:** Configurar o uso de modelos de baixa latência (ex: Gemini Flash) para as decisões de roteamento (Supervisor) e acionar modelos mais pesados apenas na formulação final da resposta pelo Especialista.
*   **Tipagem com Pydantic:** Estruturar as respostas e as chamadas de ferramentas com a biblioteca Pydantic, garantindo o "Structured Outputs" e impedindo que a IA envie parâmetros em formatos incorretos para a API.

### Sprint 4: Ecossistema de Plugins e Ferramentas (Expansão)
**Foco:** Tornar o projeto escalável, permitindo o acoplamento de novas habilidades (Skills) e a melhoria das integrações.
*   **Hot-Reload de Plugins:** Finalizar a lógica para que o orquestrador leia os contratos `MANIFESTO.json` ativamente, permitindo injetar dinamicamente novas "Hot-Tools" ou personas de setores no sistema em tempo real sem *downtime*.
*   **Paralelização de Ferramentas:** Modificar o loop de execução (Action Loop) para disparar chamadas assíncronas concorrentes quando a IA precisa usar mais de uma ferramenta (ex: consultar o Estoque e o CRM ao mesmo tempo), diminuindo o tempo de processamento.
*   **Tratamento de Conflitos Cross-Domain:** Estabelecer o Estado Compartilhado (Shared State), permitindo que um Especialista possa fazer um *Cross-Call* seguro para buscar um dado em outro departamento quando uma pergunta envolver dois setores.

### Sprint 5: Governança, Dashboard e Auditoria
**Foco:** Refinar a interface dev-first, depuração, controle de custos e salvaguardas (Guardrails) da versão corporativa.
*   **Integração Fina Langfuse SDK:** Mapear os `span_ids` com o esqueleto do grafo para monitorar exatamento o caminho das mensagens (Usuário > Supervisor > Tools > Resposta), centralizando também a gestão dinâmica de *Prompts*.
*   **Dashboard de Gestão e Human-in-the-loop:** Finalizar as rotas de API para a interface administrativa, incluindo os botões de ação técnica (`[COMMIT]`, `[ROLLBACK]`) para ferramentas pendentes de aprovação humana e edição manual de *State*.
*   **Guardrail Model Outbound:** Implementar um modelo/nó de auditoria final (Vigia) que analisa a resposta final da IA para garantir que regras de sigilo (ex: não divulgar informações da folha de pagamento para um vendedor) sejam 100% cumpridas.

### Sprint 6: Deploy, CI/CD e Lançamento (Go-Live)
**Foco:** Empacotar a aplicação, estruturar atualizações automatizadas e colocar no ar o cenário MVP.
*   **Configuração de Hospedagem:** Criar um ambiente (preferencialmente utilizando Coolify em uma VPS ou PaaS similar) apontando para o repositório GitHub para viabilizar um ambiente com HTTPS, backups e facilidade de operação.
*   **Pipeline de CI/CD (Automação):** Configurar webhooks entre o repositório GitHub e o Coolify. Assim, um *push* iniciará um *build* automático dos containers, mantendo logs e histórico de traces no banco self-hosted intactos.
*   **Fase 1 (Lançamento Controlado):** Fazer o deploy da primeira fase (apenas 1 Setor ou Tarefa crítica, como o Comercial). Utilizar segurança rígida via Código (Hard Rules) para garantir que toda a operação funcione sob total controle e previsibilidade antes de escalar para os demais departamentos.