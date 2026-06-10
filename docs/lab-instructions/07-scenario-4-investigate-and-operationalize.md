# Cenário 4 — Investigar e Operacionalizar (~15 min)

O incidente foi resolvido. Agora: "quanto tempo ficou fora?" e "como evitamos na próxima vez?"

> ⏱️ **Latência de ingestão de logs:** Logs de sistema do Container App podem levar ~5 minutos para aparecer no Log Analytics. Se você concluiu o Cenário 3 rapidamente, os dados já devem estar disponíveis — se não, aguarde um minuto e tente novamente.

## Parte A — Post-mortem via KQL (~7 min)

**Diga ao Copilot:**

```
Query the Log Analytics workspace for my Container App. Show me what happened during the port mismatch incident.
```

### 7️⃣ `azure-diagnostics` ativa

Observe como ela monta a investigação:

1. **Descoberta do workspace** — localiza seu Log Analytics workspace a partir do resource group
2. **Exploração de tabelas** — consulta `ContainerAppSystemLogs_CL` para encontrar tipos de evento disponíveis
3. **Distribuicao de eventos** — executa KQL `summarize count() by Reason_s` para mostrar o resumo: eventos ProbeFailed (startup probe falhando por porta errada), impacto ReplicaUnhealthy, recuperação RevisionUpdate
4. **Linha do tempo do incidente** — escreve consulta KQL com `earliest(TimeGenerated)` e `latest(TimeGenerated)` para calcular duração exata da indisponibilidade
5. **Confirmacao de recuperação** — verifica eventos `RevisionReady` para provar que a correção funcionou

> 💡 **Destaque da skill:** `azure-diagnostics` escreve KQL *para você* com base em linguagem natural. Revise as consultas geradas — você as escreveria de forma diferente? A skill usa `has` em vez de `==` para comparacao de strings em KQL, o que e mais resiliente a mudancas no formato dos logs.

**Revise o KQL que a IA escreveu.** Copie uma consulta e modifique — tente adicionar `| where TimeGenerated > ago(1h)` ou trocar o `summarize` para incluir `bin(TimeGenerated, 5m)` e ver em serie temporal. Execute as consultas alteradas no Copilot CLI ou cole no editor de consultas do Log Analytics no Azure Portal.

✅ **Checkpoint:** Você viu consultas KQL mostrando eventos ProbeFailed (causados pela divergencia de porta), linha do tempo do incidente e confirmacao de recuperação.

---

## Parte B — Operacionalizar (~8 min)

> ⚠️ **Nota:** Os passos abaixo podem falhar em alguns ambientes de lab por restricoes de politica existentes. Tente assim mesmo — se funcionar, você tera uma regra de alerta ativa. Se falhar, foque em entender o padrão de KQL e da configuração do alerta.

**Diga ao Copilot:**

```
Create a KQL alert rule that fires when ProbeFailed events appear in the Container App system logs.
```

### `azure-diagnostics` continua

Ela:
- Escreve a consulta KQL do alerta direcionando `ContainerAppSystemLogs_CL`
- Gera o comando completo `az monitor scheduled-query create` com limite, frequencia, severidade e action group
- Explica cada parâmetro para ajuste (por exemplo, frequencia de avaliacao, numero de violacoes antes de disparar)

> ⚠️ **Pre-requisito:** A extensão `scheduled-query` do Azure CLI precisa estar instalada: `az extension add --name scheduled-query --yes`

**Depois, pergunte:**

```
What other alert rules should I have for a production Container App backed by Cosmos DB?
```

A IA sugere: saude de replicas, loops de restart, alta latência, picos de 5xx, uso de memória, consumo de RUs do Cosmos DB e throttling (429) — cada um com o padrão KQL necessario.

✅ **Checkpoint:** `az monitor scheduled-query list -g <rg> -o table` mostra sua regra de alerta.

**Aprendizado:** Dois prompts, uma skill (`azure-diagnostics`), e você saiu de "incidente encerrado" para "está classe de incidente vai me alertar na próxima vez". O valor nível 300: agora você consegue ler e modificar essas consultas KQL sozinho.

> 💡 **Dica:** O parâmetro `--condition` em `az monitor scheduled-query create` usa um formato DSL específico, não KQL puro. A condição referência o nome da tabela da consulta KQL (por exemplo, `ContainerAppSystemLogs_CL`), enquanto a consulta completa vai em `--condition-query`. Se o comando falhar, confira se esses dois parametros estão consistentes.

---

**Próximo:** [Solução de Problemas →](08-troubleshooting.md)




