# Qorp Core - O Cronômetro da Venda (Latência)

No Comercial, tempo é dinheiro. Se o cliente manda um "Qual o preço?" no WhatsApp e a IA demora 10 segundos pra responder, ele já fechou a aba ou chamou o concorrente. 

Aqui está o nosso plano de guerra pra manter o Qorp Core rápido.

---

## 1. Onde o tempo "foge" (Cenário MVP)
Abaixo, o que acontece num pedido real (Ex: "Tem o produto X em estoque e qual o desconto?"):

| O que a IA faz | Tempo Estimado | Por que demora? |
| :--- | :--- | :--- |
| **Supervisor** | ~0.8s | Decide se o pedido é comercial, financeiro ou papo furado. |
| **Especialista** | ~1.2s | Pensa quais ferramentas usar (Estoque + CRM). |
| **Ferramentas** | ~0.5s | Consulta o banco de dados e APIs externas (Async). |
| **Resposta Final** | ~2.0s | Escreve o texto amigável pro cliente. |
| **TOTAL** | **~4.5s** | **Limite crítico para o WhatsApp.** |

---

## 2. Como a gente ganha segundos preciosos

### A. Streaming (Não espere a IA terminar)
A gente não entrega a resposta inteira de uma vez. O usuário começa a ver o texto saindo assim que o primeiro token é gerado. No front-end (Web), isso dá a sensação de resposta instantânea (~1.5s), mesmo que o agente leve 4s pra terminar de escrever.

### B. Ferramentas em Paralelo
Se a IA precisar consultar o estoque e o cadastro do cliente, o orquestrador dispara as duas ferramentas ao mesmo tempo (Async). 
- **Economia:** Se cada uma demora 300ms, em paralelo a gente gasta só 300ms (ao invés de 600ms).

### C. Cache de Roteamento (Sticky Session)
Se o cliente já está falando com o "Vendedor", a gente pula o Supervisor nas próximas 3 ou 4 mensagens. O sistema assume que ele continua no Comercial até que o contexto mude drasticamente.
- **Economia bruta:** Ganhamos quase **1 segundo** em cada mensagem do meio da conversa.

### D. Modelos Mistos (O Segredo)
- **Supervisor:** Usa um modelo ultra-rápido (ex: Gemini Flash ou GPT-4o-mini). Decisão binária não precisa de "cérebro gigante".
- **Vendedor:** Usa o modelo "inteligente" (Pro/Large) só quando precisa fechar a venda ou lidar com objeção complexa.

---

## 3. Veredito Técnico
Para o MVP Comercial, nosso alvo é o **"Time to First Token" (TTFT) abaixo de 2 segundos**. Se a gente bater isso, a experiência no WhatsApp fica lisa e o cliente sente que está falando com um humano rápido, não com um robô travado.

---
**Navegação:**
- [Estratégia de Implementação](./ESTRATEGIA_IMPLEMENTACAO.md)
- [Arquitetura Sistêmica](./ARQUITETURA_SISTEMICA.md)
- [Análise de Riscos](./ANALISE_DE_RISCOS.md)
