# DevOps Agêntico de ponta a ponta com Azure MCP — Publicar, Fortalecer, Quebrar, Investigar

**Lab prático (75 min) | Nível: 300 | LAB501**

A IA pode implantar seu app no Azure em 5 minutos. Mas você deve confiar no que ela construiu? Neste lab, você vai usar o GitHub Copilot CLI com skills do Azure para implantar um Container App em produção real — uma aplicação Python Flask que navega por um catálogo de conjuntos LEGO com dados no Azure Cosmos DB — junto com uma Function App que faz upsert de novos conjuntos LEGO no Cosmos DB. Depois, você vai colocar o chapéu de arquiteto e avaliar as decisões da IA. Você vai revisar os arquivos Bicep gerados, identificar o que falta para prontidão de produção, orientar a IA a fortalecer o deployment, quebrar o app de propósito e executar uma investigação forense completa — tudo sem abrir o Azure Portal.

> 💡 **As respostas da IA podem variar** em relação ao que está descrito neste guia. Foque em quais skills ativam e nos padrões de raciocínio, não na saída exata. Os prompts foram testados, mas IA e não determinística — seus resultados podem ficar um pouco diferentes.

### Arquitetura Alvo

```mermaid
graph TB
    User((Usuário<br/>Navegador))
    Admin((Cliente de<br/>Ingestão))

    subgraph RG["Resource Group"]
        subgraph Web["Camada Web"]
            ACR["Azure Container Registry"]
            CAE["Container Apps Environment"]
            CA["Container App"]
        end

        subgraph Api["Camada de API"]
            ASP["App Service Plan"]
            FUNC["Function App"]
            UAMI["User-Assigned Managed Identity"]
        end

        subgraph Storage["Armazenamento de apoio"]
            ST["Storage Account"]
            BLOB["Blob Container"]
        end

        subgraph Obs["Observabilidade"]
            LAW["Log Analytics Workspace"]
            AI["Application Insights"]
        end

        subgraph VNetOpt["Opcional"]
            VNET["Virtual Network"]
            APPSUB["Subnet"]
            PESUB["Subnet"]
            PE["Private Endpoints"]
        end
    end

    subgraph CosmosRG["Resource Group Externo"]
        COSMOS["Cosmos DB Account"]
        DB["Cosmos DB Database"]
        CONT["Cosmos DB Container"]
    end

    %% Fluxos de usuário
    User ==>|"HTTPS"| CA
    Admin ==>|"HTTPS POST<br/>x-functions-key"| FUNC

    %% Camada web
    ACR -->|"pull da imagem<br/>(credenciais admin via secret)"| CA
    CA --- CAE
    CAE -->|"logs do app"| LAW

    %% Camada de API
    FUNC --- ASP
    FUNC -.->|"atribuida"| UAMI
    UAMI -->|"Storage Blob Data Owner<br/>+ Contributor"| ST
    ST --> BLOB
    BLOB -->|"pacote de deployment"| FUNC

    %% Plano de dados do Cosmos (cross-RG)
    CA ==>|"consultas SQL via SDK<br/>System-MI → Cosmos Data Reader"| COSMOS
    UAMI ==>|"upsert_item via SDK<br/>UAMI → Cosmos Data Contributor"| COSMOS
    COSMOS --> DB --> CONT

    %% Telemetria
    CA -.->|"APPLICATIONINSIGHTS_<br/>CONNECTION_STRING"| AI
    FUNC -.->|"telemetria (Entra auth)"| AI
    AI --- LAW

    %% Caminho opcional de VNet
    VNET --- APPSUB
    VNET --- PESUB
    FUNC -.->|"integração com VNet<br/>(se habilitado)"| APPSUB
    PESUB -.->|"private link"| PE
    PE -.->|"acesso privado<br/>(se habilitado)"| ST

    classDef external fill:#fef3c7,stroke:#d97706,stroke-width:2px;
    classDef optional fill:#f3f4f6,stroke:#9ca3af,stroke-dasharray: 5 5;
    classDef identity fill:#ede9fe,stroke:#7c3aed;
    class COSMOS,DB,CONT,CosmosRG external;
    class VNET,APPSUB,PESUB,PE,VNetOpt optional;
    class UAMI identity;
```

## O que você vai aprender

- Como as **skills** do Azure se encadeiam — um prompt pode acionar `prepare` → `validate` → `deploy` automaticamente
- Onde a infraestrutura gerada por IA te leva a 80% — e quais lacunas de produção você precisa fechar
- Como revisar criticamente Bicep, Dockerfiles e diagramas de arquitetura gerados por IA
- Como `azure-diagnostics` raciocina sobre problemas: padrões de triagem, correlação de logs, geração de KQL
- Quando confiar nas decisões da IA e quando sobrescreve-las
- Como conectar um app em container a um Azure Cosmos DB pre-provisionado usando managed identity

## Skills usadas — 6 skills em 4 cenarios

| # | Skill | O que faz | Cenário |
|---|---|---|---|
| 1 | `azure-prepare` | Lida com dois pontos de partida em uma passada: envolve o código Flask existente com IaC + configuração (Container Apps) **e** busca um template Python do Azure Functions e o adapta ao prompt (Flex Consumption) | 1A: Publicar |
| 2 | `azure-validate` | Verificacoes pre-flight: compilacao de Bicep, status do Docker (Container Apps), runtime Python + disponibilidade de Flex Consumption (Functions), acesso a subscription | 1A: Publicar |
| 3 | `azure-deploy` | Executa `azd up` — provisiona infraestrutura + build + deploy dos dois serviços (Flask Container App e Python Function App) | 1A: Publicar |
| 4 | `azure-rbac` | Encontra papeis de menor privilegio na documentação Azure e gera comandos de atribuicao | 1B: Fortalecer |
| 5 | `azure-resource-visualizer` | Consulta o Resource Graph, mapeia relacoes e gera diagramas Mermaid | 2: Visualizar |
| 6 | `azure-diagnostics` | Coleta logs do sistema, segue cadeia de raciocínio diagnóstico ate a causa raiz; escreve consultas KQL e cria regras de alerta | 3: Quebrar, 4: Investigar |

> 📖 **Glossario:** **ACR** = Azure Container Registry (repositório privado de imagens Docker). **AZD** = Azure Developer CLI (`azd`). **Bicep** = linguagem IaC do Azure. **Cosmos DB** = banco NoSQL globalmente distribuido do Azure. **KQL** = Kusto Query Language (consultas de logs). **MCP** = Model Context Protocol.

## Secoes do lab

| # | Secao | Arquivo | Duracao |
|---|---------|------|----------|
| 1 | [Pre-requisitos](01-prerequisites.md) | `01-prerequisites.md` | Pre-sessão |
| 2 | [Login e Inicialização](02-login-and-launch.md) | `02-login-and-launch.md` | ~5 min |
| 3 | [Configurar o App Inicial](03-getting-started.md) | `03-getting-started.md` | ~5 min |
| 4 | [Cenário 1 — Publicar e Fortalecer](04-scenario-1-ship-and-harden.md) | `04-scenario-1-ship-and-harden.md` | ~25 min |
| 5 | [Cenário 2 — Visualizar e Avaliar](05-scenario-2-see-and-evaluate.md) | `05-scenario-2-see-and-evaluate.md` | ~10 min |
| 6 | [Cenário 3 — Quebrar e Triar](06-scenario-3-break-and-triage.md) | `06-scenario-3-break-and-triage.md` | ~10 min |
| 7 | [Cenário 4 — Investigar e Operacionalizar](07-scenario-4-investigate-and-operationalize.md) | `07-scenario-4-investigate-and-operationalize.md` | ~15 min |
| 8 | [Solução de Problemas](08-troubleshooting.md) | `08-troubleshooting.md` | Referência |
| 9 | [Próximos Passos](09-whats-next.md) | `09-whats-next.md` | Referência |




