# Cenário 1 — Publicar e Fortalecer (~35 min)

A IA pode montar seu deployment no Azure em minutos. Mas você subiria Bicep gerado por IA para produção sem revisar?

## Parte A — Publicar (~25 min)

> 💡 **Garanta que seu app NÃO está em execução.** Se você iniciou `python app.py` durante o checkpoint, pare com **Ctrl+C** antes de continuar.

Se você ainda não estiver no diretório **lego-set-browser**, entre nele e use o prompt abaixo para iniciar uma sessão do Copilot em **modo yolo**.

```
copilot --yolo
```

A flag `--yolo` aprova comandos automaticamente e ignora prompts de confirmacao — seguro aqui porque o lab roda em ambiente sandbox e pode economizar vários minutos ao longo do exercicio. Em seguida, diga ao Copilot:

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

Este único prompt aciona uma **cadeia de 3 skills** — observe o Copilot invocar cada uma:

### 1️⃣ `azure-prepare` ativa primeiro

Observe como ele lida com **dois pontos de partida bem diferentes** em uma única passada — este e o insight principal deste passo:

- **Flask app → Container Apps (ponto de partida: código fonte existente no workspace):**
  - Escaneia o workspace — encontra `requirements.txt` e `app.py`, e classifica como app web Python Flask
  - Escolhe Container Apps como alvo de hospedagem (você concorda com essa escolha em vez de App Service ou Functions?)
  - Revisa o `Dockerfile` existente (o app já tem um — observe se a skill reutiliza como está ou regenera)
  - Gera infraestrutura *ao redor* de código que já existe
- **Python Function App → Azure Functions (ponto de partida: apenas seu prompt — sem código ainda):**
  - Não existe código de function no workspace. A skill lê seu prompt (trigger HTTP POST, array JSON de conjuntos LEGO, batch-upsert no Cosmos DB) e **busca um template Python adequado de Azure Functions**
  - Modifica o template para seu cenário: reescreve o handler para aceitar o formato JSON do LEGO, mapeia `set_number → id`, conecta ao database/container do Cosmos DB existente e adiciona binding com user-assigned managed identity
  - Escolhe **Flex Consumption (FC1)** como plano de hospedagem e adiciona uma Storage account para o pacote de deployment
  - Gera infraestrutura *e* código fonte, a partir de template
- **Compartilhado entre os dois serviços:**
  - Produz um único `azure.yaml` declarando ambos os serviços e templates Bicep em `infra/` (normalmente `main.bicep` mais módulos por servico)
  - Cria um ambiente AZD e define sua subscription + regiao

> 💡 **Destaque da skill:** `azure-prepare` não apenas gera arquivos — ele lê referências da skill para runtime da linguagem, padrões Bicep, convenções AZD e template de Azure Functions Flex Consumption. Abra os arquivos gerados: o Bicep em `infra/` e o novo `function_app.py` vieram de templates de referência da skill, não de boilerplate genérico.

### 2️⃣ `azure-validate` ativa em seguida

Ela executa verificacoes pre-flight nos dois serviços:
- Compila Bicep (`az bicep build`) para os módulos de Container App e Function App — captura erros de sintaxe antes do deployment
- Verifica se o Docker está rodando — o build da imagem do Container App falha sem ele
- Verifica a versão do runtime Python e se Flex Consumption (FC1) está disponível na região selecionada — deployment de Functions falha rapidamente caso contrário
- Confirma seu acesso a subscription e que o nome do resource group não está em uso

### 3️⃣ `azure-deploy` ativa por ultimo

Ela executa `azd up --no-prompt`, que provisiona e pública os dois serviços em uma execução orquestrada:
- **Lado Container App:** Provisiona ACR, Container Apps Environment, Log Analytics e o Container App; faz build da imagem Docker e pública no ACR
- **Lado Function App:** Provisiona Storage account, plano Flex Consumption (FC1), user-assigned managed identity, Application Insights e Function App; empacota e pública o código Python da function
- Configura variáveis de ambiente do Cosmos DB nos dois serviços e retorna endpoints HTTPS ativos (URL do Container Apps para a UI Flask e URL do Functions para o endpoint de ingestão)

### Verifique se funcionou

```bash
curl <your-endpoint-url>
```

> 💡 **Encontrando a URL do endpoint:** Se a URL saiu da tela, execute `azd env get-values` ou pergunte ao Copilot: "What's the URL for my Container App?". A URL parece com `https://<app-name>.<region>.azurecontainerapps.io`.

> 💡 **Primeira requisição pode ser lenta:** A primeira requisição após o deployment pode levar 10-15 segundos enquanto a nova revisão ativa. Isso e normal — tente novamente depois de um instante.

> ⚠️ **Acesso ao Cosmos DB pelo Container App:** Apos o deployment, o Container App precisa de permissão para acessar o Cosmos DB. Se você vir erros de acesso a banco de dados, isso e esperado — você vai resolver na Parte B (Fortalecer) configurando managed identity com o papel RBAC apropriado do Cosmos DB.

**Estado final:** Endpoint HTTPS ativo servindo o navegador de conjuntos LEGO. Tres skills, um prompt. Publicado, mas ainda não pronto para produção.

![LEGO Vault app em execução no navegador](images/workingApp.png)

> 📁 **Quais arquivos foram criados?** Apos a Parte A, seu diretório `lego-app` deve conter novos arquivos gerados por `azure-prepare`: tipicamente `azure.yaml` e uma pasta `infra/` com templates Bicep (por exemplo, `main.bicep`, `main.parameters.json` e módulos de suporte). O `Dockerfile` existente pode ter sido mantido como estava ou atualizado. A estrutura exata pode variar um pouco conforme a IA montou seu projeto.

---

## Parte B — Fortalecer (~10 min)

> ⚠️ **Observacao sobre permissões:** Algumas operacoes de hardening (por exemplo, criar role assignments para managed identity) podem exigir papel **Owner** ou **User Access Administrator** na subscription. Se você encontrar erros de permissão neste passo, isso e esperado em alguns ambientes de lab — foque em entender o padrão e revisar as recomendacoes da IA, em vez de executar cada comando.

Revise os arquivos Bicep gerados no diretório `infra/`. Dependendo de como o `azure-prepare` executou, ele pode já ter aplicado parte do hardening de segurança durante a geração. Seu trabalho e **auditar o que a IA fez e o que não fez**.

**Diga ao Copilot:**

```
Review my deployed Container App infrastructure for production readiness gaps. Check for managed identity, Cosmos DB RBAC access (instead of keys), VNet integration, diagnostic settings, and health probes.
```

### O que observar

| Lacuna | Por que importa | Severidade |
|---|---|---|
| **Sem managed identity para pull no ACR** | Container App usa credenciais de admin para puxar imagens. Achado de segurança. | Alta |
| **Sem managed identity para Cosmos DB** | App usa chaves de conexão em vez de RBAC. Chaves podem vazar. | Alta |
| **Sem integração com VNet** | Container Apps Environment está em rede pública. Sem isolamento. | Média |
| **Sem diagnostic settings** | Métricas da plataforma não são encaminhadas. Você perderia alertas de CPU/memória. | Média |
| **Sem health probe configurado** | O padrão usa probe TCP. Seu app tem rota home — deveria usa-la. | Baixa |

> 💡 **A IA pode já ter fortalecido parte disso.** A skill `azure-prepare` inclui uma fase de hardening que pode configurar managed identity e RBAC durante a geração inicial. Se você encontrar managed identity e AcrPull já configurados — otimo, isso e a skill funcionando como esperado. Foque no que ainda está faltando, principalmente acesso ao Cosmos DB via managed identity.

✅ **Checkpoint:** Você auditou a infraestrutura gerada e confirmou se a IA fortaleceu o ambiente ou identificou próximos passos. O Container App deve estar com managed identity configurada tanto para pull de imagem no ACR quanto para acesso a dados no Cosmos DB.

**Aprendizado:** A IA pode gerar um deployment seguro de primeira — ou não. A fase de hardening da skill e não determinística, e exatamente por isso que revisão humana importa. Seja a IA fazendo hardening ou você guiando por prompt, a habilidade crítica e saber como e um ambiente "pronto para produção" e verificar isso.

---

**Próximo:** [Cenário 2 — Visualizar e Avaliar →](05-scenario-2-see-and-evaluate.md)




