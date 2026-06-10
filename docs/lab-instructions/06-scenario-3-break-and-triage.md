# Cenario 3 — Quebrar e Triar (~10 min)

Sao 2 da manha. Seu app retorna 503. Voce abre o terminal. Preste atencao na cadeia de raciocinio diagnostico da IA, nao apenas na resposta.

## Introduza a falha

Substitua `<app>` e `<rg>` pelo nome real do seu Container App e pelo resource group do Cenario 1A (execute `azd env get-values` se precisar localizar). Em seguida, em uma nova aba do PowerShell, execute:

```powershell
az containerapp ingress update --name <app> -g <rg> --target-port 9999
```

> ⏱️ **Este comando leva ~30 segundos a 2 minutos** enquanto a nova revisao do Container Apps ativa. Isso e esperado — nao use Ctrl+C.

Acesse o endpoint — voce recebera `503 Service Unavailable`.

---

## Diagnostique com IA

**Diga ao Copilot:**

```
My Container App is returning 503. What's wrong?
```

### 6️⃣ `azure-diagnostics` ativa

Observe a cadeia de triagem:

1. **Formacao de hipoteses** — a skill considera varios modos de falha: crash da app? erro de ingress? imagem ruim? ambiente unhealthy?
2. **Coleta de logs** — busca logs de sistema do Container App com `az containerapp logs show --type system`
3. **Correlacao de logs** — encontra `Reason: ProbeFailed` — *"Probe of StartUp failed with status code: 1"* (a startup probe falha porque o container nao esta ouvindo na porta 9999)
4. **Verificacao de configuracao** — cruza config de ingress (porta 9999) com a porta real da app (8000, definida pelo gunicorn no Dockerfile)
5. **Causa raiz + correcao** — entrega o comando CLI exato para restaurar a porta correta

> 💡 **Destaque da skill:** `azure-diagnostics` nao apenas procura erros em logs — ela segue uma cadeia de raciocinio diagnostico. Comeca amplo (o que pode causar 503?), afunila por evidencia (logs de sistema mostram ProbeFailed) e confirma com dados de configuracao. E o mesmo padrao de triagem que um SRE senior seguiria.

---

## Aplique a correcao

Execute o comando de correcao sugerido. Deve ser algo como:

```powershell
az containerapp ingress update --name <app> -g <rg> --target-port 8000
```

Verifique a recuperacao → `200 OK`.

✅ **Checkpoint:** `curl <your-endpoint-url>` volta a retornar o HTML do navegador de conjuntos LEGO.

**Aprendizado:** Uma pergunta em linguagem natural → `azure-diagnostics` ativada → causa raiz + correcao em ~30 segundos. A skill fez a correlacao de logs que voce normalmente faria manualmente no portal.

---

**Proximo:** [Cenario 4 — Investigar e Operacionalizar →](07-scenario-4-investigate-and-operationalize.md)
