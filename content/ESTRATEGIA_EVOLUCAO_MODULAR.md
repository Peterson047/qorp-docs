# Transição Modular: Do Monoagente para a Orquestração

**Data:** 04/03/2026
**Status:** Decisão Arquitetural Ativa

## O Conceito "Camaleão"
A estratégia Qorp para a fase de tração inicial (Metas/Setores específicos) foca na simplicidade operacional sem sacrificar a visão de longo prazo. Em vez de uma malha complexa de agentes que competem por atenção e contexto, utilizamos um **Monoagente Modular**.

### 1. Por que não ir direto para Multi-Agente?
- **Debug:** Rastrear decisões em múltiplos saltos de LLM é exponencialmente mais difícil.
- **Latência:** Cada agente adiciona segundos à resposta final.
- **Custo:** Múltiplas chamadas para uma única tarefa simples.

### 2. A Estrutura do Core
O Agente atual funciona como o **Core do Orquestrador**. Ele possui:
- **Identidade Fluida:** O prompt é montado dinamicamente com base no perfil do usuário carregado do MongoDB.
- **Ferramentas Dinâmicas:** O acesso ao servidor MCP é filtrado. O agente só "enxerga" as ferramentas permitidas para aquele perfil específico.

### 3. O Caminho para a Versão Parruda
Quando o volume de perfis e a complexidade das tarefas excederem a capacidade de um único System Message, a transição será natural:
1. O **Core** torna-se o **Supervisor**.
2. Os **Perfis** tornam-se **Sub-agentes** independentes.
3. As **Ferramentas MCP** permanecem as mesmas, apenas mudando de "mão".

---
*Este documento serve como guia de implementação para garantir que o desenvolvimento atual suporte a escala futura da Qorp.*
