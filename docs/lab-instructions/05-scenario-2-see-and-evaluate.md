# Cenario 2 — Visualizar e Avaliar (~10 min)

Diagramas de arquitetura costumam estar desatualizados, errados ou nem existir. A IA pode gera-los na hora — mas estao corretos?

## Parte A — Gerar o diagrama (~4 min)

**Diga ao Copilot:**

```
Visualize the resources in my resource group as an architecture diagram.
```

### 5️⃣ `azure-resource-visualizer` ativa

Observe como ele:
- Consulta o Azure Resource Graph para inventariar todos os recursos no seu resource group
- Mapeia relacoes: Container App → Container Apps Environment → Log Analytics, Container App → ACR
- Gera um diagrama Mermaid com subgrafos rotulados, tipos de recurso e setas de conexao
- Entrega markdown renderizavel para colar em qualquer visualizador Mermaid

> 💡 **Destaque da skill:** O visualizer nao apenas lista recursos — ele infere relacionamentos pelas propriedades dos recursos (por exemplo, `environmentId` conecta o Container App ao Environment). Ele le o modelo ARM de recursos, nao adivinha por nomes. Observe que algumas conexoes — como a dependencia de Cosmos DB — sao descobertas apenas pelas variaveis de ambiente do Container App, nao pelas propriedades ARM, entao o visualizer pode nao captura-las automaticamente.

> 💡 **E o Cosmos DB?** Seu app conecta em um Cosmos DB pre-provisionado que pode estar em outro resource group. Verifique se o visualizer captura essa dependencia cross-resource-group ou se mostra apenas os recursos do resource group do deployment. Esta e uma lacuna comum em diagramas auto-gerados.

---

## Parte B — Avaliar o diagrama (~6 min)

Abra o markdown gerado e revise de forma critica:

- Capturou os recursos implantados (Container App, Environment, ACR, Log Analytics)?
- As relacoes estao corretas? Mostra o pull ACR → Container App?
- Mostra a dependencia de Cosmos DB? Se nao, esta faltando algo importante — o app nao funciona sem isso.
- O que esta faltando para uma revisao de arquitetura de producao?

**Diga ao Copilot:**

```
What's missing from this architecture for a production deployment? The app also connects to an existing Cosmos DB for its data.
```

Compare as recomendacoes da IA com seus achados do Cenario 1B.

✅ **Checkpoint:** Voce tem um diagrama Mermaid mostrando recursos implantados com setas de conexao. Para renderizar: copie o bloco Mermaid da saida do Copilot e cole em [mermaid.live](https://mermaid.live), use VS Code com extensao Mermaid, ou publique o markdown em um repositorio GitHub — o GitHub renderiza Mermaid nativamente em arquivos markdown.

**Aprendizado:** `azure-resource-visualizer` e excelente para descoberta ("o que existe agora?") mas exige revisao especializada para documentacao ("isso esta completo e correto?"). O diagrama reflete o estado implantado dentro de um resource group, nao o quadro completo — dependencias cross-resource-group como seu Cosmos DB sao parte da sua responsabilidade de documentar.

---

**Proximo:** [Cenario 3 — Quebrar e Triar →](06-scenario-3-break-and-triage.md)
