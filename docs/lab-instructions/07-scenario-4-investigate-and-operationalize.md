# Cenario 4 — Investigar e Operacionalizar (~15 min)

O incidente foi resolvido. Agora: "quanto tempo ficou fora?" e "como evitamos na proxima vez?"

> ⏱️ **Latencia de ingestao de logs:** Logs de sistema do Container App podem levar ~5 minutos para aparecer no Log Analytics. Se voce concluiu o Cenario 3 rapidamente, os dados ja devem estar disponiveis — se nao, aguarde um minuto e tente novamente.

## Parte A — Post-mortem via KQL (~7 min)

**Diga ao Copilot:**

```
Query the Log Analytics workspace for my Container App. Show me what happened during the port mismatch incident.
```

### 7️⃣ `azure-diagnostics` ativa

Observe como ela monta a investigacao:

1. **Descoberta do workspace** — localiza seu Log Analytics workspace a partir do resource group
2. **Exploracao de tabelas** — consulta `ContainerAppSystemLogs_CL` para encontrar tipos de evento disponiveis
3. **Distribuicao de eventos** — executa KQL `summarize count() by Reason_s` para mostrar o resumo: eventos ProbeFailed (startup probe falhando por porta errada), impacto ReplicaUnhealthy, recuperacao RevisionUpdate
4. **Linha do tempo do incidente** — escreve consulta KQL com `earliest(TimeGenerated)` e `latest(TimeGenerated)` para calcular duracao exata da indisponibilidade
5. **Confirmacao de recuperacao** — verifica eventos `RevisionReady` para provar que a correcao funcionou

> 💡 **Destaque da skill:** `azure-diagnostics` escreve KQL *para voce* com base em linguagem natural. Revise as consultas geradas — voce as escreveria de forma diferente? A skill usa `has` em vez de `==` para comparacao de strings em KQL, o que e mais resiliente a mudancas no formato dos logs.

**Revise o KQL que a IA escreveu.** Copie uma consulta e modifique — tente adicionar `| where TimeGenerated > ago(1h)` ou trocar o `summarize` para incluir `bin(TimeGenerated, 5m)` e ver em serie temporal. Execute as consultas alteradas no Copilot CLI ou cole no editor de consultas do Log Analytics no Azure Portal.

✅ **Checkpoint:** Voce viu consultas KQL mostrando eventos ProbeFailed (causados pela divergencia de porta), linha do tempo do incidente e confirmacao de recuperacao.

---

## Parte B — Operacionalizar (~8 min)

> ⚠️ **Nota:** Os passos abaixo podem falhar em alguns ambientes de lab por restricoes de politica existentes. Tente assim mesmo — se funcionar, voce tera uma regra de alerta ativa. Se falhar, foque em entender o padrao de KQL e da configuracao do alerta.

**Diga ao Copilot:**

```
Create a KQL alert rule that fires when ProbeFailed events appear in the Container App system logs.
```

### `azure-diagnostics` continua

Ela:
- Escreve a consulta KQL do alerta direcionando `ContainerAppSystemLogs_CL`
- Gera o comando completo `az monitor scheduled-query create` com limite, frequencia, severidade e action group
- Explica cada parametro para ajuste (por exemplo, frequencia de avaliacao, numero de violacoes antes de disparar)

> ⚠️ **Pre-requisito:** A extensao `scheduled-query` do Azure CLI precisa estar instalada: `az extension add --name scheduled-query --yes`

**Depois, pergunte:**

```
What other alert rules should I have for a production Container App backed by Cosmos DB?
```

A IA sugere: saude de replicas, loops de restart, alta latencia, picos de 5xx, uso de memoria, consumo de RUs do Cosmos DB e throttling (429) — cada um com o padrao KQL necessario.

✅ **Checkpoint:** `az monitor scheduled-query list -g <rg> -o table` mostra sua regra de alerta.

**Aprendizado:** Dois prompts, uma skill (`azure-diagnostics`), e voce saiu de "incidente encerrado" para "esta classe de incidente vai me alertar na proxima vez". O valor nivel 300: agora voce consegue ler e modificar essas consultas KQL sozinho.

> 💡 **Dica:** O parametro `--condition` em `az monitor scheduled-query create` usa um formato DSL especifico, nao KQL puro. A condicao referencia o nome da tabela da consulta KQL (por exemplo, `ContainerAppSystemLogs_CL`), enquanto a consulta completa vai em `--condition-query`. Se o comando falhar, confira se esses dois parametros estao consistentes.

---

**Proximo:** [Solucao de Problemas →](08-troubleshooting.md)
