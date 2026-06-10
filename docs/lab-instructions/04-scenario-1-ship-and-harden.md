# Cenario 1 — Publicar e Fortalecer (~35 min)

A IA pode montar seu deployment no Azure em minutos. Mas voce subiria Bicep gerado por IA para producao sem revisar?

## Parte A — Publicar (~25 min)

> 💡 **Garanta que seu app NAO esta em execucao.** Se voce iniciou `python app.py` durante o checkpoint, pare com **Ctrl+C** antes de continuar.

Se voce ainda nao estiver no diretorio **lego-set-browser**, entre nele e use o prompt abaixo para iniciar uma sessao do Copilot em **modo yolo**.

```
copilot --yolo
```

A flag `--yolo` aprova comandos automaticamente e ignora prompts de confirmacao — seguro aqui porque o lab roda em ambiente sandbox e pode economizar varios minutos ao longo do exercicio. Em seguida, diga ao Copilot:

```
  Create and deploy 2 Azure services 
   
   **Environment:**
   - Subscription: Current subscription
   - Create a new resource group: rg-lego-set-browser-dev
   - Region: West US 3
   
   **Existing Cosmos DB (do NOT create a new one):**
   - Look for the existing cosmos DB in the current subscription
   - Database: LegoDatabase / Container: legoSets
   
   **1. Python Azure Function App** — HTTP POST trigger on Flex Consumption (FC1):
   - Accepts JSON array of LEGO sets; batch-upserts to Cosmos DB above
   - Fields: set_number (→ id), name, theme_name, year_released, number_of_parts, type, image_url
   - User-assigned managed identity for Cosmos DB (Built-in Data Contributor)
   
   **2. Flask app in this folder → Azure Container Apps:**
   - Name: ca-web-lego-<XXXX>
   - Already uses DefaultAzureCredential + env vars COSMOS_ENDPOINT, COSMOS_DATABASE, COSMOS_CONTAINER
   - System-assigned managed identity for Cosmos DB (Built-in Data Reader)
```

Este unico prompt aciona uma **cadeia de 3 skills** — observe o Copilot invocar cada uma:

### 1️⃣ `azure-prepare` ativa primeiro

Observe como ele lida com **dois pontos de partida bem diferentes** em uma unica passada — este e o insight principal deste passo:

- **Flask app → Container Apps (ponto de partida: codigo fonte existente no workspace):**
  - Escaneia o workspace — encontra `requirements.txt` e `app.py`, e classifica como app web Python Flask
  - Escolhe Container Apps como alvo de hospedagem (voce concorda com essa escolha em vez de App Service ou Functions?)
  - Revisa o `Dockerfile` existente (o app ja tem um — observe se a skill reutiliza como esta ou regenera)
  - Gera infraestrutura *ao redor* de codigo que ja existe
- **Python Function App → Azure Functions (ponto de partida: apenas seu prompt — sem codigo ainda):**
  - Nao existe codigo de function no workspace. A skill le seu prompt (trigger HTTP POST, array JSON de conjuntos LEGO, batch-upsert no Cosmos DB) e **busca um template Python adequado de Azure Functions**
  - Modifica o template para seu cenario: reescreve o handler para aceitar o formato JSON do LEGO, mapeia `set_number → id`, conecta ao database/container do Cosmos DB existente e adiciona binding com user-assigned managed identity
  - Escolhe **Flex Consumption (FC1)** como plano de hospedagem e adiciona uma Storage account para o pacote de deployment
  - Gera infraestrutura *e* codigo fonte, a partir de template
- **Compartilhado entre os dois servicos:**
  - Produz um unico `azure.yaml` declarando ambos os servicos e templates Bicep em `infra/` (normalmente `main.bicep` mais modulos por servico)
  - Cria um ambiente AZD e define sua subscription + regiao

> 💡 **Destaque da skill:** `azure-prepare` nao apenas gera arquivos — ele le referencias da skill para runtime da linguagem, padroes Bicep, convencoes AZD e template de Azure Functions Flex Consumption. Abra os arquivos gerados: o Bicep em `infra/` e o novo `function_app.py` vieram de templates de referencia da skill, nao de boilerplate generico.

### 2️⃣ `azure-validate` ativa em seguida

Ela executa verificacoes pre-flight nos dois servicos:
- Compila Bicep (`az bicep build`) para os modulos de Container App e Function App — captura erros de sintaxe antes do deployment
- Verifica se o Docker esta rodando — o build da imagem do Container App falha sem ele
- Verifica a versao do runtime Python e se Flex Consumption (FC1) esta disponivel na regiao selecionada — deployment de Functions falha rapidamente caso contrario
- Confirma seu acesso a subscription e que o nome do resource group nao esta em uso

### 3️⃣ `azure-deploy` ativa por ultimo

Ela executa `azd up --no-prompt`, que provisiona e publica os dois servicos em uma execucao orquestrada:
- **Lado Container App:** Provisiona ACR, Container Apps Environment, Log Analytics e o Container App; faz build da imagem Docker e publica no ACR
- **Lado Function App:** Provisiona Storage account, plano Flex Consumption (FC1), user-assigned managed identity, Application Insights e Function App; empacota e publica o codigo Python da function
- Configura variaveis de ambiente do Cosmos DB nos dois servicos e retorna endpoints HTTPS ativos (URL do Container Apps para a UI Flask e URL do Functions para o endpoint de ingestao)

### Verifique se funcionou

```bash
curl <your-endpoint-url>
```

> 💡 **Encontrando a URL do endpoint:** Se a URL saiu da tela, execute `azd env get-values` ou pergunte ao Copilot: "What's the URL for my Container App?". A URL parece com `https://<app-name>.<region>.azurecontainerapps.io`.

> 💡 **Primeira requisicao pode ser lenta:** A primeira requisicao apos o deployment pode levar 10-15 segundos enquanto a nova revisao ativa. Isso e normal — tente novamente depois de um instante.

> ⚠️ **Acesso ao Cosmos DB pelo Container App:** Apos o deployment, o Container App precisa de permissao para acessar o Cosmos DB. Se voce vir erros de acesso a banco de dados, isso e esperado — voce vai resolver na Parte B (Fortalecer) configurando managed identity com o papel RBAC apropriado do Cosmos DB.

**Estado final:** Endpoint HTTPS ativo servindo o navegador de conjuntos LEGO. Tres skills, um prompt. Publicado, mas ainda nao pronto para producao.

![LEGO Vault app em execucao no navegador](images/workingApp.png)

> 📁 **Quais arquivos foram criados?** Apos a Parte A, seu diretorio `lego-app` deve conter novos arquivos gerados por `azure-prepare`: tipicamente `azure.yaml` e uma pasta `infra/` com templates Bicep (por exemplo, `main.bicep`, `main.parameters.json` e modulos de suporte). O `Dockerfile` existente pode ter sido mantido como estava ou atualizado. A estrutura exata pode variar um pouco conforme a IA montou seu projeto.

---

## Parte B — Fortalecer (~10 min)

> ⚠️ **Observacao sobre permissoes:** Algumas operacoes de hardening (por exemplo, criar role assignments para managed identity) podem exigir papel **Owner** ou **User Access Administrator** na subscription. Se voce encontrar erros de permissao neste passo, isso e esperado em alguns ambientes de lab — foque em entender o padrao e revisar as recomendacoes da IA, em vez de executar cada comando.

Revise os arquivos Bicep gerados no diretorio `infra/`. Dependendo de como o `azure-prepare` executou, ele pode ja ter aplicado parte do hardening de seguranca durante a geracao. Seu trabalho e **auditar o que a IA fez e o que nao fez**.

**Diga ao Copilot:**

```
Review my deployed Container App infrastructure for production readiness gaps. Check for managed identity, Cosmos DB RBAC access (instead of keys), VNet integration, diagnostic settings, and health probes.
```

### O que observar

| Lacuna | Por que importa | Severidade |
|---|---|---|
| **Sem managed identity para pull no ACR** | Container App usa credenciais de admin para puxar imagens. Achado de seguranca. | Alta |
| **Sem managed identity para Cosmos DB** | App usa chaves de conexao em vez de RBAC. Chaves podem vazar. | Alta |
| **Sem integracao com VNet** | Container Apps Environment esta em rede publica. Sem isolamento. | Media |
| **Sem diagnostic settings** | Metricas da plataforma nao sao encaminhadas. Voce perderia alertas de CPU/memoria. | Media |
| **Sem health probe configurado** | O padrao usa probe TCP. Seu app tem rota home — deveria usa-la. | Baixa |

> 💡 **A IA pode ja ter fortalecido parte disso.** A skill `azure-prepare` inclui uma fase de hardening que pode configurar managed identity e RBAC durante a geracao inicial. Se voce encontrar managed identity e AcrPull ja configurados — otimo, isso e a skill funcionando como esperado. Foque no que ainda esta faltando, principalmente acesso ao Cosmos DB via managed identity.

✅ **Checkpoint:** Voce auditou a infraestrutura gerada e confirmou se a IA fortaleceu o ambiente ou identificou proximos passos. O Container App deve estar com managed identity configurada tanto para pull de imagem no ACR quanto para acesso a dados no Cosmos DB.

**Aprendizado:** A IA pode gerar um deployment seguro de primeira — ou nao. A fase de hardening da skill e nao deterministica, e exatamente por isso que revisao humana importa. Seja a IA fazendo hardening ou voce guiando por prompt, a habilidade critica e saber como e um ambiente "pronto para producao" e verificar isso.

---

**Proximo:** [Cenario 2 — Visualizar e Avaliar →](05-scenario-2-see-and-evaluate.md)
