# Qorp Core: Especificação do Manifesto de Plugin (O Contrato de Expansão)

Este documento detalha o formato técnico do **Manifesto**, o componente central que permite ao Qorp Core escalar de uma única tarefa específica para toda a inteligência departamental de uma empresa sem alterar o código-fonte original.

---

## 1. A Filosofia do "Plug-and-Play" Corporativo

O Manifesto é o que torna o Qorp Core agnóstico. Ele funciona como uma **carteira de identidade e habilidades** que um novo setor apresenta ao sistema. Se o sistema começa apenas com o setor Comercial, o primeiro manifesto será o desse setor. Quando o RH for adicionado, basta "plugar" um novo arquivo JSON.

---

## 2. Estrutura do Manifesto (`manifest.json`)

Abaixo, a especificação técnica dos campos que compõem o contrato de um plugin:

```json
{
  "plugin_metadata": {
    "id": "comercial-vendas-01",
    "version": "1.2.0",
    "owner": "Departamento Comercial"
  },
  
  "access_control": {
    "required_roles": ["vendedor", "gerente_comercial"],
    "security_level": "standard",
    "data_sandbox": "sales_production_db"
  },

  "intelligence_profile": {
    "identity": "Especialista em Gestão de Estoque e Vendas",
    "instruction_snippet": "Sua prioridade é fornecer dados precisos de estoque. Use tom profissional e consultivo.",
    "hot_tools": [
      "query_inventory_api",
      "calculate_shipping_costs",
      "fetch_client_kpi"
    ]
  },

  "ui_actions": {
    "allowed_commands": ["SET_DASHBOARD_FILTER", "OPEN_PRODUCT_PAGE"],
    "render_mode": "compact_cards"
  }
}
```

---

## 3. Os 4 Pilares do Manifesto

### I. Controle de Acesso (RBAC Nativo)
O campo `required_roles` é o que o **Supervisor de Acesso** consulta. Se um usuário não possui estas roles, o Core nem carrega o restante do manifesto para aquela sessão, garantindo segurança absoluta por omissão.

### II. Perfil de Inteligência (Custom Persona)
O `instruction_snippet` permite que cada setor tenha sua própria voz e diretrizes éticas ou operacionais, sem poluir o "cérebro central" do Qorp Core.

### III. Hot-Tools (Injeção Dinâmica)
Diferente de sistemas fixos, aqui as ferramentas (APIs, SQLs, Scripts) são injetadas no Grafo de execução **apenas quando necessário**. Isso mantém a janela de contexto limpa e reduz o risco de a IA usar uma ferramenta errada.

### IV. Ações de Interface (Agentic UI)
Define quais comandos de interface o plugin está autorizado a enviar para o Frontend (ex: Next.js). Isso garante que o Agente Comercial possa "filtrar a tabela de vendas", mas nunca "mudar configurações de servidor".

---

## 4. O Fluxo de Carregamento (Runtime)

1.  **Discovery:** O Core monitora uma pasta ou banco de manifestos.
2.  **Match:** O Supervisor de Contexto identifica a necessidade de uma Skill específica.
3.  **Hydration:** O Core "hidrata" o StateGraph com as ferramentas e prompts do manifesto escolhido.
4.  **Execution:** O Agente Especialista assume o turno de conversa com as capacidades injetadas.

---

## 5. Conclusão: Crescimento sem Débito Técnico

Com a especificação do Manifesto, o Qorp Core deixa de ser um "projeto de IA" e se torna uma **Plataforma de IA**. Novos setores da empresa podem desenvolver e testar seus próprios plugins de forma independente, acelerando a adoção da tecnologia em toda a organização de forma governada e segura.

---
**Navegação:**
- [Contratos e Segurança](./CONTRATOS_E_SEGURANCA.md)
- [Custom Tools (Exemplo Next.js)](./AGENTE_DE_UI_E_DADOS.md)
- [Gestão de Conflitos](./GESTAO_CONFLITOS.md)
