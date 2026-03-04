# Especificação: Relações Sistêmicas e Delegação de Identidade

Este documento detalha como o **Qorp Core** se posiciona como um Hub de Inteligência para múltiplos sistemas (Metas, Administrativo, etc.) e como a responsabilidade de gestão de dados é delegada entre eles.

---

## 1. O Conceito: Dono do Cadastro vs. Dono do Mapa 🗺️

Uma dúvida crucial de arquitetura é onde reside o gerenciamento de usuários. A resposta curta é: o Qorp não é o "dono" do cadastro, ele é o **"Dono do Mapa"**. 🤪🗺️

Para o sistema ser profissional e evitar redundância de dados, seguimos a regra da **Fonte Única de Verdade (SSOT)**:

### A. O Sistema Mestre (Dono do Usuário) 🏛️
O gerenciamento "administrativo" (CRUD) fica no sistema mestre (ex: Metas ou Administrativo):
- É onde se cria o usuário, define senha, altera cargo ou desativa contas.
- O sistema mestre é quem detém a autoridade: *"O Peterson é do Comercial e tem nível de acesso 5"*.

### B. O Qorp Core (Registrador de Identidade) 📇
O Qorp guarda apenas o que é estritamente necessário para a operação da IA:
- **Identity Registry (IR):** Uma tabela de "De-Para" que mapeia as identidades. Ex: *"O Peterson do Metas (ID 789) é o mesmo Peterson do WhatsApp (ID 5511...)"*.
- **Memória de Longo Prazo:** O perfil comportamental e preferências que a IA aprendeu sobre o usuário ao longo do tempo.

> **Por que não centralizar tudo no Qorp?** 
> Para evitar a manutenção dupla. Se você tem 500 funcionários, não quer atualizar dois sistemas quando alguém muda de setor. O fluxo ideal é: Altera no Mestre -> O Mestre notifica o Qorp via Webhook/API -> O Qorp atualiza o mapa instantaneamente.

---

## 2. O Princípio da Fonte Única de Verdade (SSOT)

A regra de ouro da arquitetura Qorp é o **desacoplamento administrativo**.

| Responsabilidade | Sistema Mestre (Metas/Admin) | Qorp Core (AI Hub) |
| :--- | :--- | :--- |
| **Criação de Usuário** | 🟢 Sim (CRUD Master) | 🔴 Não |
| **Definição de Cargo/Permissões** | 🟢 Sim (Dono da Regra) | 🔴 Não (Apenas consome) |
| **Vínculo WhatsApp/Telegram** | 🟡 Intermediário (Handshake) | 🟢 Sim (Dono do Mapa) |
| **Memória Comportamental** | 🔴 Não | 🟢 Sim (Especialista) |
| **Logs de Auditoria de IA** | 🔴 Não | 🟢 Sim (Dono do Trace) |

### Analogia do Aeroporto ✈️🤪
- **Sistema Mestre:** É o **Cartório/Governo**. Ele emite o seu Passaporte (Identidade) e diz legalmente quem você é.
- **Qorp Core:** É a **Alfândega do Aeroporto**. Ele tem uma lista de quem tem visto para entrar e sabe que o passageiro do portão A (WhatsApp) é o mesmo que estava no portão B (Web) ontem. Ele não te "criou", ele apenas te **reconhece** e te dá permissão de trânsito.

---

## 3. Arquitetura Multi-Tenant (Múltiplos Sistemas)

O Qorp foi desenhado para atuar como um **AI Gateway** centralizado, atendendo diferentes frentes de negócio simultaneamente sem misturar os contextos.

### O Mecanismo de "Troca de Skin" (Context Switching)
Quando um sistema secundário (ex: Administrativo) se conecta ao Qorp, a integração segue o modelo de **Injeção de Perfil**:

1.  **Request do Admin:** O backend do Administrativo envia uma mensagem acompanhada de um `Client_ID` e um `user_profile` específico.
2.  **Roteamento Interno:** O Qorp identifica a origem e "troca a gaveta" de ferramentas. Ele oculta o cinto de utilidades do Comercial (Metas) e carrega as ferramentas de RH/Financeiro (Administrativo).
3.  **Isolamento de Memória:** Embora o `internal_user_id` seja le mesmo, o Qorp pode separar a memória de longo prazo por `namespace`, garantindo que o que foi discutido no financeiro não "vaze" como sugestão em uma conversa comercial.

---

## 4. Fluxo de Delegação de Autoridade

Para garantir a segurança, o Qorp nunca "toma uma decisão" de acesso sozinho. Ele sempre obedece à delegação do sistema mestre via **JWT Scopes**.

1.  **Autenticação:** O Peterson faz login no **Administrativo**.
2.  **Assinatura:** O **Administrativo** assina um Token (JWT) dizendo: *"Este é o Peterson, ele é `admin_full`. Eu (Administrativo) autorizo ele a usar a IA nestes termos."*
3.  **Reconhecimento:** O Qorp recebe o token, valida a assinatura digital e executa a ordem. Ele não questiona o nível de acesso, ele apenas o **faz cumprir (Enforcement)**.

---

## 5. O Desafio do WhatsApp (Omnichannel Routing)

No canal de WhatsApp, onde o usuário pode perguntar sobre qualquer sistema da empresa, o Qorp atua como um **Supervisor de Intenção**:

*   **Identificação:** O Registry localiza o `internal_user_id` pelo número de telefone.
*   **Classificação:** A IA analisa a frase: *"Preciso do balancete de ontem"*.
*   **Encaminhamento:** O Supervisor percebe que a intenção é Administrativa, carrega o contexto do sistema correspondente e processa a resposta usando as ferramentas delegadas para aquele sistema.

---
**Conclusão:** 
O gerenciamento de usuários (Vida e Morte da conta) fica no Mestre. O Qorp apenas **indexa** essa identidade para que a IA saiba com quem está falando em qualquer canal. Isso mantém o sistema mestre como o centro de tudo, e o Qorp como um "módulo inteligente" que obedece às ordens do mestre. 🤪🚀

---
**Navegação:**
- [Guia de Integração: Sistema Metas](./INTEGRACAO_SISTEMA_METAS.md)
- [Identity Registry Spec](./IDENTITY_REGISTRY_SPEC.md)
- [Fluxo de Pareamento (Whats)](./FLUXO_VINCULACAO_IDENTIDADE.md)
