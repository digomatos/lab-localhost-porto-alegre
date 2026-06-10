# Solução de Problemas

## Conexão com Cosmos DB falha localmente
**Sintoma:** `python app.py` mostra erro de autenticação ou conexão.
**Causa:** O app usa `DefaultAzureCredential`, que exige login ativo no Azure CLI.
**Correção:** Execute `az login` e confirme que está na subscription correta. Verifique se `COSMOS_ENDPOINT` no `.env` corresponde ao Cosmos DB provisionado.

## Nome do ACR com hífen → deployment falha
**Sintoma:** `azd up` falha com erro de nome inválido de ACR.
**Causa:** Nomes de ACR devem ser alfanuméricos. Hifens no nome do ambiente AZD propagam para o nome do registry.
**Correção:** Use nome de ambiente sem hífen (exemplo: `lab501app`). Execute `azd init` novamente com novo nome.

## AZD pública na subscription errada
**Sintoma:** Recursos aparecem em outra subscription, ou você recebe erro de permissão.
**Causa:** AZD mantém configuração própria de subscription, separada de `az account show`.
**Correção:** Execute `azd env set AZURE_SUBSCRIPTION_ID $(az account show --query id -o tsv)` para alinhar.

## Container App não conecta ao Cosmos DB após deployment
**Sintoma:** O app pública, mas mostra erros de banco ao acessar.
**Causa:** O Container App não tem permissões ou variáveis de ambiente corretas para acessar o Cosmos DB.
**Correção:** Verifique se `COSMOS_ENDPOINT`, `COSMOS_DATABASE` e `COSMOS_CONTAINER` estão definidos no Container App. Se usar managed identity, confirme se a identity system-assigned possui o papel RBAC correto no Cosmos DB.

## `az containerapp ingress update` trava por mais de 2 minutos
**Sintoma:** O comando parece preso após executar.
**Causa:** O CLI espera a nova revisão do Container Apps ativar.
**Correção:** Isso e esperado. Aguarde concluir — não use Ctrl+C.

## Primeira requisição após deployment da timeout ou resposta lenta
**Sintoma:** `curl` da timeout ou leva mais de 10 segundos na primeira requisição.
**Causa:** A nova revisão está ativando (cold start). `minReplicas: 1` está configurado, mas a ativacao inicial ainda leva tempo.
**Correção:** Aguarde ~15 segundos após concluir o deployment e tente novamente.

## Consulta KQL sem resultados no Cenário 4
**Sintoma:** Consultas retornam tabelas vazias.
**Causa:** Ingestão no Log Analytics tem latência de ~5 minutos. Métricas tem latência de ~15 minutos.
**Correção:** Aguarde 5 minutos após o Cenário 3 e execute novamente.

## `az monitor scheduled-query create` falha com "command not found"
**Sintoma:** O CLI não reconhece o comando `scheduled-query`.
**Causa:** A extensão preview do CLI não está instalada.
**Correção:** Execute `az extension add --name scheduled-query --yes`

## Build Docker falha durante `azd up`
**Sintoma:** Deployment falha com erro relacionado ao Docker.
**Causa:** Docker Desktop não está em execução.
**Correção:** Inicie o Docker Desktop e valide com `docker version`. Depois execute `azd up` novamente.

## Instalacao de dependências Python falha
**Sintoma:** `pip install -r requirements.txt` falha com erros.
**Causa:** Dependencias de sistema ausentes ou versão Python incorreta.
**Correção:** Verifique se está usando Python 3.13+ com `python --version`. Tente `pip install --upgrade pip` antes.

## Escape de aspas no PowerShell em consultas KQL
**Sintoma:** KQL `where Reason_s == "ProbeFailed"` falha com erro de sintaxe no PowerShell.
**Causa:** O PowerShell trata aspas duplas de forma diferente do bash.
**Correção:** Use operador `has`: `where Reason_s has "ProbeFailed"`. A IA geralmente lida com isso automaticamente.

## Divergencia de porta do gunicorn após deployment
**Sintoma:** O Container App retorna 503 mesmo após deployment novo.
**Causa:** A porta alvo do ingress não corresponde a porta de bind do gunicorn (8000).
**Correção:** Garanta que a porta alvo do ingress do Container App está em `8000` (igual ao CMD do `Dockerfile`: `gunicorn --bind 0.0.0.0:8000`).

---

**Voltar para:** [Visão Geral](00-overview.md) | [Próximos Passos →](09-whats-next.md)




