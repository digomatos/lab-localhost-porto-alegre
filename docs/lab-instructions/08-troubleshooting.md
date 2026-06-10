# Solucao de Problemas

## Conexao com Cosmos DB falha localmente
**Sintoma:** `python app.py` mostra erro de autenticacao ou conexao.
**Causa:** O app usa `DefaultAzureCredential`, que exige login ativo no Azure CLI.
**Correcao:** Execute `az login` e confirme que esta na subscription correta. Verifique se `COSMOS_ENDPOINT` no `.env` corresponde ao Cosmos DB provisionado.

## Nome do ACR com hifen → deployment falha
**Sintoma:** `azd up` falha com erro de nome invalido de ACR.
**Causa:** Nomes de ACR devem ser alfanumericos. Hifens no nome do ambiente AZD propagam para o nome do registry.
**Correcao:** Use nome de ambiente sem hifen (exemplo: `lab501app`). Execute `azd init` novamente com novo nome.

## AZD publica na subscription errada
**Sintoma:** Recursos aparecem em outra subscription, ou voce recebe erro de permissao.
**Causa:** AZD mantem configuracao propria de subscription, separada de `az account show`.
**Correcao:** Execute `azd env set AZURE_SUBSCRIPTION_ID $(az account show --query id -o tsv)` para alinhar.

## Container App nao conecta ao Cosmos DB apos deployment
**Sintoma:** O app publica, mas mostra erros de banco ao acessar.
**Causa:** O Container App nao tem permissoes ou variaveis de ambiente corretas para acessar o Cosmos DB.
**Correcao:** Verifique se `COSMOS_ENDPOINT`, `COSMOS_DATABASE` e `COSMOS_CONTAINER` estao definidos no Container App. Se usar managed identity, confirme se a identity system-assigned possui o papel RBAC correto no Cosmos DB.

## `az containerapp ingress update` trava por mais de 2 minutos
**Sintoma:** O comando parece preso apos executar.
**Causa:** O CLI espera a nova revisao do Container Apps ativar.
**Correcao:** Isso e esperado. Aguarde concluir — nao use Ctrl+C.

## Primeira requisicao apos deployment da timeout ou resposta lenta
**Sintoma:** `curl` da timeout ou leva mais de 10 segundos na primeira requisicao.
**Causa:** A nova revisao esta ativando (cold start). `minReplicas: 1` esta configurado, mas a ativacao inicial ainda leva tempo.
**Correcao:** Aguarde ~15 segundos apos concluir o deployment e tente novamente.

## Consulta KQL sem resultados no Cenario 4
**Sintoma:** Consultas retornam tabelas vazias.
**Causa:** Ingestao no Log Analytics tem latencia de ~5 minutos. Metricas tem latencia de ~15 minutos.
**Correcao:** Aguarde 5 minutos apos o Cenario 3 e execute novamente.

## `az monitor scheduled-query create` falha com "command not found"
**Sintoma:** O CLI nao reconhece o comando `scheduled-query`.
**Causa:** A extensao preview do CLI nao esta instalada.
**Correcao:** Execute `az extension add --name scheduled-query --yes`

## Build Docker falha durante `azd up`
**Sintoma:** Deployment falha com erro relacionado ao Docker.
**Causa:** Docker Desktop nao esta em execucao.
**Correcao:** Inicie o Docker Desktop e valide com `docker version`. Depois execute `azd up` novamente.

## Instalacao de dependencias Python falha
**Sintoma:** `pip install -r requirements.txt` falha com erros.
**Causa:** Dependencias de sistema ausentes ou versao Python incorreta.
**Correcao:** Verifique se esta usando Python 3.13+ com `python --version`. Tente `pip install --upgrade pip` antes.

## Escape de aspas no PowerShell em consultas KQL
**Sintoma:** KQL `where Reason_s == "ProbeFailed"` falha com erro de sintaxe no PowerShell.
**Causa:** O PowerShell trata aspas duplas de forma diferente do bash.
**Correcao:** Use operador `has`: `where Reason_s has "ProbeFailed"`. A IA geralmente lida com isso automaticamente.

## Divergencia de porta do gunicorn apos deployment
**Sintoma:** O Container App retorna 503 mesmo apos deployment novo.
**Causa:** A porta alvo do ingress nao corresponde a porta de bind do gunicorn (8000).
**Correcao:** Garanta que a porta alvo do ingress do Container App esta em `8000` (igual ao CMD do `Dockerfile`: `gunicorn --bind 0.0.0.0:8000`).

---

**Voltar para:** [Visao Geral](00-overview.md) | [Proximos Passos →](09-whats-next.md)
