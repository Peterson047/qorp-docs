# Qorp Core: Implementação de Custom Tools (Exemplo Next.js)

Diferente das ferramentas nativas já prontas (como busca no Google ou leitura de arquivos), o verdadeiro poder do Qorp Core reside nas **Custom Tools**. Este documento exemplifica como criar uma ferramenta personalizada que permite à IA interagir diretamente com o Frontend em **Next.js**.

---

## 1. O Conceito: Agentic UI Bridge

Neste exemplo, vamos criar uma ferramenta chamada `update_dashboard_view`. Ela não apenas retorna um texto, mas dispara um evento que altera o que o usuário está vendo na tela do sistema comercial.

### Por que Custom Tools?
- **Integração com Legado:** Permite que a IA chame funções dentro do seu sistema PHP ou Next.js.
- **Controle de Interface:** Transforma o chat em um controle remoto do software.
- **Contexto de Negócio:** Você define exatamente o que a IA pode ou não enviar para suas APIs.

---

## 2. Definindo a Tool (Backend - LangGraph/Python)

No Qorp Core, definimos a ferramenta usando tipagem rigorosa (Pydantic). Isso garante que a IA saiba exatamente quais parâmetros enviar.

```python
from langchain_core.tools import tool
from pydantic import BaseModel, Field

class DashboardFilter(BaseModel):
    view_type: str = Field(description="Tipo de visualização: 'chart', 'table' ou 'summary'")
    period: str = Field(description="Período de tempo, ex: 'last_7_days', 'this_month'")
    highlight_id: str = Field(None, description="ID de um vendedor ou produto para destacar na tela")

@tool("update_dashboard_view", args_schema=DashboardFilter)
def update_dashboard_view(view_type: str, period: str, highlight_id: str = None):
    """
    Atualiza a interface do Dashboard do usuário em tempo real.
    Use esta ferramenta quando o usuário pedir para filtrar dados, mudar gráficos ou focar em um item.
    """
    # Aqui o Core envia um evento via WebSocket ou Pub/Sub para o Frontend
    payload = {
        "action": "UI_UPDATE",
        "params": {"view": view_type, "range": period, "focus": highlight_id}
    }
    emit_to_frontend(payload) 
    
    return f"Interface atualizada para {view_type} referente ao período {period}."
```

---

## 3. Consumindo no Frontend (Next.js)

No lado do cliente, seu componente Next.js escuta as instruções enviadas pela Tool customizada do Qorp Core.

```javascript
// Exemplo de Hook no Next.js
export function useQorpBridge() {
  useEffect(() => {
    const socket = subscribeToQorpEvents((event) => {
      if (event.action === 'UI_UPDATE') {
        const { view, range, focus } = event.params;
        
        // Lógica tradicional de Frontend:
        setDashboardView(view);
        setFilterPeriod(range);
        if (focus) scrollAndHighlightRow(focus);
        
        toast.success(`IA atualizou sua visão para: ${view}`);
      }
    });
    return () => socket.disconnect();
  }, []);
}
```

---

## 4. O Fluxo na Prática

1.  **Usuário:** "Me mostre os gráficos de vendas deste mês e destaque o vendedor Peterson."
2.  **IA (Supervisor):** Entende a intenção e chama a Custom Tool `update_dashboard_view`.
3.  **Tool:** Valida os parâmetros e dispara o comando para o barramento de dados.
4.  **Next.js:** Recebe o comando e executa as animações/filtros na tela do usuário.
5.  **Resultado:** O usuário vê o sistema se movendo sozinho conforme ele pediu no chat.

---

## 5. Melhores Práticas para Desenvolvedores

- **Nomes Descritivos:** O nome da função e o `description` são o que a IA usa para decidir o uso. Seja extremamente claro.
- **Validação:** Sempre use Pydantic no Backend para evitar que a IA envie lixo para sua API.
- **Feedback:** A Tool deve sempre retornar uma confirmação textual curta para que a IA possa confirmar ao usuário que a ação foi concluída.

---
**Navegação:**
- [Guia de Sobrevivência IA](./GUIA_SOBREVIVENCIA_IA.md)
- [Arquitetura Sistêmica](./ARQUITETURA_SISTEMICA.md)
- [Contratos e Segurança](./CONTRATOS_E_SEGURANCA.md)
