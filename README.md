<p align="center">
<img src="img/banner-build-26.png" alt="Microsoft Build 2026" width="1200"/>
</p>

# [Microsoft Build 2026](https://build.microsoft.com)

## 🔥 LAB501: Do zero ao deploy no Azure com agentes de IA

### Descrição da sessão

O que acontece quando você deixa agentes de IA construírem por você? Neste lab prático, você vai de um terminal vazio até um app implantado no Azure — com GitHub Copilot CLI e agentes de código cuidando de scaffolding, codificação, depuração e deployment. Você vai usar as novas Azure skills para provisionar recursos e conectar serviços por linguagem natural, sem precisar do portal. Isto não é uma demo para assistir. Você sai com um fluxo de desenvolvimento real e funcional para levar direto ao próximo projeto.

Ao longo de 75 minutos (Nível 300), você vai publicar dois serviços a partir de um único prompt no Copilot — um **navegador de conjuntos LEGO em Python Flask** no Azure Container Apps e um **Azure Function App em Python** que faz batch-upsert de conjuntos LEGO no Azure Cosmos DB — e depois vai colocar o chapéu de arquiteto para avaliar as decisões da IA: revisar o Bicep gerado, fortalecer o deployment com managed identity + RBAC, quebrar propositalmente e executar uma investigação forense completa com KQL — tudo a partir do GitHub Copilot CLI.

### 🏫 Começando em uma sessão guiada

Para começar em uma sessão guiada do lab:
- Entre na VM de lab do Skillable e inicie o **Docker Desktop**
- Abra o PowerShell e execute `az login`, `azd auth login` e depois `copilot` — siga [Login & Launch](docs/lab-instructions/02-login-and-launch.md) para instalar o plugin **Azure Skills** (`/plugin install azure@azure-skills`)
- Siga a [Visão Geral do Lab](docs/lab-instructions/00-overview.md) e percorra os quatro cenários em ordem

### 🏠 Começando no seu próprio ambiente

Se você estiver seguindo os passos no seu ritmo:
- Clone este repositório: `git clone https://github.com/microsoft/Build26-LAB501.git`
- Instale os [pré-requisitos](docs/lab-instructions/01-prerequisites.md): Python 3.13+, Docker Desktop, Git, Azure CLI (com Bicep), Azure Developer CLI (`azd`), GitHub Copilot CLI e uma assinatura Azure com acesso de Contributor
- Provisione seu próprio Azure Cosmos DB com o dataset LEGO (database `LegoDatabase`, container `legoSets`) e comece em [Configurar o App Inicial](docs/lab-instructions/03-getting-started.md), seguindo os quatro cenários

### 🧠 Resultados de aprendizado

Ao final desta sessão, você será capaz de:

- Encadear Azure skills (`azure-prepare` → `azure-validate` → `azure-deploy`) a partir de um único prompt em linguagem natural no GitHub Copilot CLI para gerar IaC, Docker e configuração e publicar um app em contêiner no Azure
- Revisar criticamente Bicep, Dockerfiles e diagramas de arquitetura gerados por IA — identificar lacunas de produção e usar `azure-rbac` e `azure-resource-visualizer` para corrigi-las com managed identity de menor privilégio e documentação precisa
- Fazer triagem e operacionalizar incidentes de produção com `azure-diagnostics` — seguir a cadeia de raciocínio dos logs de sistema até a causa raiz, e transformar o post-mortem em consultas KQL e regras de alerta sem abrir o Azure Portal

### 💬 Continue aprendendo com Copilot

Experimente estes prompts com GitHub Copilot para explorar os tópicos desta sessão. Abra o Copilot Chat no VS Code (`Ctrl+Alt+I` no Windows/Linux, `Cmd+Shift+I` no Mac), cole um prompt e veja o que aprende. Tente conectar o [Microsoft Learn MCP Server](#-microsoft-learn-mcp-server) para acessar a documentação oficial mais recente.

Use como ponto de partida — ou escreva os seus próprios!

- "Scaffold a Python Flask app on Azure Container Apps **and** a Python Azure Function App with Bicep and `azd`, using managed identity for ACR pulls and Cosmos DB access."
- "Review my Container App Bicep for production-readiness gaps — managed identity, RBAC, VNet integration, diagnostic settings, and health probes — and propose fixes."
- "Find the minimum-privilege Cosmos DB RBAC role for an app that only reads data, and generate the `az cosmosdb sql role assignment create` command."
- "Visualize the resources in my resource group as a Mermaid architecture diagram, including cross-resource-group dependencies like Cosmos DB."
- "My Container App is returning 503. Pull system logs, correlate with ingress configuration, and tell me the root cause and fix."
- "Write a KQL query against `ContainerAppSystemLogs_CL` that calculates downtime between the first `ProbeFailed` event and the next `RevisionReady` event, then turn it into an `az monitor scheduled-query create` alert rule."

### 💻 Tecnologias usadas

1. GitHub Copilot CLI + plugin Azure Skills (`azure-prepare`, `azure-validate`, `azure-deploy`, `azure-rbac`, `azure-resource-visualizer`, `azure-diagnostics`) e Azure MCP Server
2. Azure Container Apps, **Azure Functions (Python, Flex Consumption)** para ingestão em lote de conjuntos LEGO, Azure Container Registry e Azure Cosmos DB (NoSQL) com managed identity + RBAC
3. Azure Developer CLI (`azd`), Bicep, Docker, Python 3.13 / Flask, Azure Monitor + Log Analytics e KQL

### 📚 Recursos e próximos passos

| Recurso | Descrição |
|:---------|:------------|
| [https://aka.ms/build26-next-steps](https://aka.ms/build26-next-steps) | Dê o próximo passo na sua jornada de aprendizado após o Build 2026 |
| [Visão Geral do Lab](docs/lab-instructions/00-overview.md) | Diagrama de arquitetura, mapa de skills e guia do lab por seção |
| [What's Next](docs/lab-instructions/09-whats-next.md) | Ideias de extensão — private endpoints, integração com VNet, Key Vault, CI/CD com OIDC, Terraform |
| [Announcing the Azure Skills Plugin](https://devblogs.microsoft.com/all-things-azure/announcing-the-azure-skills-plugin/) | Contexto sobre as Azure skills usadas ao longo do lab |
| [Azure MCP Server docs](https://learn.microsoft.com/azure/developer/azure-mcp-server) | Referência para os MCP tools que sustentam as Azure skills |
| [GitHub Copilot CLI docs](https://docs.github.com/en/copilot/github-copilot-in-the-cli) | Instalar, configurar e estender o GitHub Copilot CLI |
| [Deploy and manage Container Apps](https://learn.microsoft.com/training/paths/deploy-manage-container-apps) | Trilha da Microsoft Learn para Azure Container Apps |
| [Azure Cosmos DB documentation](https://learn.microsoft.com/azure/cosmos-db) | Modelagem de dados, RBAC e orientações operacionais para Cosmos DB |

### 🌟 Microsoft Learn MCP Server

[![Install in VS Code](https://img.shields.io/badge/VS_Code-Install_Microsoft_Docs_MCP-0098FF?style=flat-square&logo=visualstudiocode&logoColor=white)](https://vscode.dev/redirect/mcp/install?name=microsoft.docs.mcp&config=%7B%22type%22%3A%22http%22%2C%22url%22%3A%22https%3A%2F%2Flearn.microsoft.com%2Fapi%2Fmcp%22%7D)

O Microsoft Learn MCP Server é um servidor MCP remoto que permite que clientes como GitHub Copilot e outros agentes de IA tragam informações confiáveis e atualizadas diretamente da documentação oficial da Microsoft. Comece usando o botão acima de instalação com um clique no VS Code ou acesse o arquivo [mcp.json](.vscode/mcp.json) incluído neste repositório.

Para mais informações, instruções de configuração para outros clientes de desenvolvimento e para enviar comentários e perguntas, visite nosso repositório do Learn MCP Server no GitHub em [https://github.com/MicrosoftDocs/MCP](https://github.com/MicrosoftDocs/MCP). Encontre outros MCP Servers para conectar ao seu agente em [https://mcp.azure.com](https://mcp.azure.com).

*Observação: ao usar o Learn MCP Server, você concorda com os Termos de Uso do [Microsoft Learn](https://learn.microsoft.com/en-us/legal/termsofuse) e dos [Microsoft API Terms](https://learn.microsoft.com/en-us/legal/microsoft-apis/terms-of-use).*

## Responsáveis pelo conteúdo

<!-- TODO: Add yourself as a content owner
1. Change the src in the image tag to {your github url}.png
2. Change INSERT NAME HERE to your name
3. Change the github url in the final href to your url. -->

<table>
<tr>
    <td align="center"><a href="http://github.com/yunjchoi">
        <img src="https://github.com/yunjchoi.png" width="100px;" alt="Yun Jung Choi"/><br />
        <sub><b>Yun Jung Choi</b></sub></a><br />
            <a href="https://github.com/yunjchoi" title="talk">📢</a>
    </td>
</tr></table>

## Contribuindo

Este projeto recebe contribuições e sugestões. A maioria das contribuições exige que você concorde com um
Contributor License Agreement (CLA), declarando que você tem o direito de nos conceder, e de fato concede,
os direitos de usar sua contribuição. Para detalhes, visite [Contributor License Agreements](https://cla.opensource.microsoft.com).

Quando você envia um pull request, um bot de CLA determina automaticamente se você precisa fornecer
um CLA e marca o PR adequadamente (por exemplo, verificação de status, comentário). Basta seguir as instruções
fornecidas pelo bot. Você precisará fazer isso apenas uma vez em todos os repositórios que usam nosso CLA.

Este projeto adotou o [Microsoft Open Source Code of Conduct](https://opensource.microsoft.com/codeofconduct/).
Para mais informações, consulte o [Code of Conduct FAQ](https://opensource.microsoft.com/codeofconduct/faq/) ou
entre em contato pelo email [opencode@microsoft.com](mailto:opencode@microsoft.com) para dúvidas ou comentários adicionais.

## Marcas registradas

Este projeto pode conter marcas registradas ou logotipos de projetos, produtos ou serviços. O uso autorizado de marcas
ou logotipos da Microsoft está sujeito e deve seguir
[Microsoft's Trademark & Brand Guidelines](https://www.microsoft.com/legal/intellectualproperty/trademarks/usage/general).
O uso de marcas ou logotipos da Microsoft em versões modificadas deste projeto não deve causar confusão nem implicar patrocínio da Microsoft.
Qualquer uso de marcas ou logotipos de terceiros está sujeito às políticas desses terceiros.
