# Cenário 3 — Quebrar e Triar (~10 min)

São 2 da manhã. Seu app retorna 503. Você abre o terminal. Preste atenção na cadeia de raciocínio diagnóstico da IA, não apenas na resposta.

## Introduza a falha

Substitua `<app>` e `<rg>` pelo nome real do seu Container App e pelo resource group do Cenário 1A (execute `azd env get-values` se precisar localizar). Em seguida, em uma nova aba do PowerShell, execute:

```powershell
az containerapp ingress update --name <app> -g <rg> --target-port 9999
```

> ⏱️ **Este comando leva ~30 segundos a 2 minutos** enquanto a nova revisão do Container Apps ativa. Isso e esperado — não use Ctrl+C.

Acesse o endpoint — você receberá `503 Service Unavailable`.

---

## Diagnostique com IA

**Diga ao Copilot:**

```
My Container App is returning 503. What's wrong?
```

### 6️⃣ `azure-diagnostics` ativa

Observe a cadeia de triagem:

1. **Formação de hipóteses** — a skill considera vários modos de falha: crash da app? erro de ingress? imagem ruim? ambiente unhealthy?
2. **Coleta de logs** — busca logs de sistema do Container App com `az containerapp logs show --type system`
3. **Correlação de logs** — encontra `Reason: ProbeFailed` — *"Probe of StartUp failed with status code: 1"* (a startup probe falha porque o container não está ouvindo na porta 9999)
4. **Verificação de configuração** — cruza config de ingress (porta 9999) com a porta real da app (8000, definida pelo gunicorn no Dockerfile)
5. **Causa raiz + correção** — entrega o comando CLI exato para restaurar a porta correta

> 💡 **Destaque da skill:** `azure-diagnostics` não apenas procura erros em logs — ela segue uma cadeia de raciocínio diagnóstico. Começa amplo (o que pode causar 503?), afunila por evidência (logs de sistema mostram ProbeFailed) e confirma com dados de configuração. E o mesmo padrão de triagem que um SRE sênior seguiria.

---

## Aplique a correção

Execute o comando de correção sugerido. Deve ser algo como:

```powershell
az containerapp ingress update --name <app> -g <rg> --target-port 8000
```

Verifique a recuperação → `200 OK`.

✅ **Checkpoint:** `curl <your-endpoint-url>` volta a retornar o HTML do navegador de conjuntos LEGO.

**Aprendizado:** Uma pergunta em linguagem natural → `azure-diagnostics` ativada → causa raiz + correção em ~30 segundos. A skill fez a correlação de logs que você normalmente faria manualmente no portal.

---

**Próximo:** [Cenário 4 — Investigar e Operacionalizar →](07-scenario-4-investigate-and-operationalize.md)




