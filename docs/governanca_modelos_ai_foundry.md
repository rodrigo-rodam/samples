# Governança de Modelos no Azure AI Foundry

*(Template Operacional)*

## 1. Objetivo

Garantir que todos os modelos e fluxos de IA sejam desenvolvidos, avaliados, implantados e monitorados de forma segura, responsável e conforme políticas corporativas.

***

## 2. Estrutura de Governança

*   **Hubs & Projects**: isolamento lógico por caso de uso.
*   **Model Registry**: repositório único para modelos, prompt flows, datasets e avaliações.
*   **Catálogo Aprovado**: lista de modelos e regiões permitidas.

***

## 3. Papéis e Responsabilidades (RACI)

| Papel                | Responsabilidade Principal           |
| -------------------- | ------------------------------------ |
| Product Owner        | Aprovação do caso de uso e orçamento |
| Tech Lead            | Arquitetura e pipeline CI/CD         |
| Data Steward         | Governança e qualidade dos dados     |
| RAI Officer          | Avaliações de IA Responsável         |
| Segurança/Compliance | Políticas, auditoria e incidentes    |
| FinOps               | Controle de custos e quotas          |

***

## 4. Checklist de Liberação (Gate de Produção)

*   ✅ Avaliações de segurança (toxicity, jailbreak, PII)
*   ✅ Avaliações de qualidade (groundedness, precisão)
*   ✅ Evidências de red teaming
*   ✅ Plano de rollback e kill switch
*   ✅ Aprovação do RAI Board e Change Advisory

***

## 5. Políticas e Guardrails

*   **RBAC**: acesso mínimo necessário (Princípio do Privilégio Mínimo)
*   **Azure Policy**: bloquear regiões/modelos não aprovados
*   **Rede Privada**: VNet + Private Link
*   **Purview**: classificação e linhagem dos dados
*   **Quota & Rate Limits**: evitar abuso de recursos

***

## 6. Exemplos Detalhados de Políticas Azure Policy

### **1. Bloquear regiões não autorizadas**

Objetivo: Garantir que recursos sejam implantados apenas em regiões aprovadas (ex.: East US e Brazil South).

```json
{
  "properties": {
    "displayName": "Permitir apenas implantação em East US e Brazil South",
    "policyType": "Custom",
    "mode": "All",
    "parameters": {
      "allowedLocations": {
        "type": "Array",
        "defaultValue": ["eastus", "brazilsouth"]
      }
    },
    "policyRule": {
      "if": {
        "not": {
          "field": "location",
          "in": "[parameters('allowedLocations')]"
        }
      },
      "then": {
        "effect": "deny"
      }
    }
  }
}
```

***

### **2. Exigir Private Link para endpoints**

Objetivo: Garantir que serviços de IA sejam acessados apenas via rede privada.

```json
{
  "properties": {
    "displayName": "Obrigar Private Link em serviços de IA",
    "policyType": "Custom",
    "mode": "Indexed",
    "policyRule": {
      "if": {
        "field": "Microsoft.Network/privateEndpoints[*]",
        "exists": "false"
      },
      "then": {
        "effect": "deny"
      }
    }
  }
}
```

***

### **3. Bloquear modelos não aprovados**

Objetivo: Permitir apenas modelos que constam no catálogo corporativo.

```json
{
  "properties": {
    "displayName": "Permitir apenas modelos aprovados no AI Foundry",
    "policyType": "Custom",
    "mode": "All",
    "parameters": {
      "allowedModels": {
        "type": "Array",
        "defaultValue": ["gpt-4", "mistral-7b", "llama-2-70b"]
      }
    },
    "policyRule": {
      "if": {
        "not": {
          "field": "Microsoft.AI/modelName",
          "in": "[parameters('allowedModels')]"
        }
      },
      "then": {
        "effect": "deny"
      }
    }
  }
}
```

***

### **4. Limitar custo diário por recurso**

Objetivo: Evitar abuso de recursos e controlar gastos.

```json
{
  "properties": {
    "displayName": "Limitar custo diário para serviços de IA",
    "policyType": "Custom",
    "mode": "Indexed",
    "policyRule": {
      "if": {
        "field": "Microsoft.CostManagement/dailyCost",
        "greater": "100"
      },
      "then": {
        "effect": "audit"
      }
    }
  }
}
```

***

### **5. Exigir tags de governança**

Objetivo: Garantir rastreabilidade e compliance.

```json
{
  "properties": {
    "displayName": "Obrigar tags de governança em recursos de IA",
    "policyType": "Custom",
    "mode": "Indexed",
    "policyRule": {
      "if": {
        "not": {
          "field": "tags['Governance']",
          "exists": "true"
        }
      },
      "then": {
        "effect": "deny"
      }
    }
  }
}
```

***

## 7. Pipeline CI/CD com Gates

*(mesmo diagrama Mermaid anterior)*

***

## 8. Observabilidade e Auditoria

*   Métricas: groundedness, taxa de recusa, latência, custo
*   Alertas: degradação de qualidade ou violações de segurança
*   Logs: CI/CD, Registry, runtime, avaliações

***

## 9. Integração com Purview

*   Linhagem completa: dados → prompt flow → modelo
*   Políticas de acesso e classificação
*   Evidências para auditoria regulatória

***

## 10. Revisão Periódica

*   Frequência: trimestral
*   Itens: reavaliação de riscos, atualização do catálogo, sunset de versões antigas

***

## 11. Próximos Passos

1.  Definir **Catálogo Aprovado** com modelos e regiões permitidas.
2.  Configurar **Azure Policy** e RBAC para impor restrições.
3.  Criar pipeline CI/CD com **gates de avaliação**.
4.  Integrar **Purview** para governança de dados.
5.  Publicar este template no repositório interno e treinar equipe.

***

## 12. Modelo de Catálogo Aprovado

| Modelo           | Versão     | Status     | Região Permitida      |
| ---------------- | ---------- | ---------- | --------------------- |
| GPT-4            | 2025-10-01 | Aprovado   | East US, Brazil South |
| Mistral-7B       | 2025-09-15 | Aprovado   | Brazil South          |
| LLaMA-2-70B      | 2025-08-20 | Em análise | East US               |
| Modelo Interno X | 2025-07-10 | Bloqueado  | N/A                   |

***