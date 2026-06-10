# Cenário 2 — Visualizar e Avaliar (~10 min)

Diagramas de arquitetura costumam estar desatualizados, errados ou nem existir. A IA pode gerá-los na hora — mas estão corretos?

## Parte A — Gerar o diagrama (~4 min)

**Diga ao Copilot:**

```
Visualize the resources in my resource group as an architecture diagram.
```

### 5️⃣ `azure-resource-visualizer` ativa

Observe como ele:
- Consulta o Azure Resource Graph para inventariar todos os recursos no seu resource group
- Mapeia relacoes: Container App → Container Apps Environment → Log Analytics, Container App → ACR
- Gera um diagrama Mermaid com subgrafos rotulados, tipos de recurso e setas de conexão
- Entrega markdown renderizavel para colar em qualquer visualizador Mermaid

> 💡 **Destaque da skill:** O visualizer não apenas lista recursos — ele infere relacionamentos pelas propriedades dos recursos (por exemplo, `environmentId` conecta o Container App ao Environment). Ele lê o modelo ARM de recursos, não adivinha por nomes. Observe que algumas conexões — como a dependência de Cosmos DB — são descobertas apenas pelas variáveis de ambiente do Container App, não pelas propriedades ARM, então o visualizer pode não capturá-las automaticamente.

> 💡 **E o Cosmos DB?** Seu app conecta em um Cosmos DB pré-provisionado que pode estar em outro resource group. Verifique se o visualizer captura essa dependência cross-resource-group ou se mostra apenas os recursos do resource group do deployment. Esta é uma lacuna comum em diagramas auto-gerados.

---

## Parte B — Avaliar o diagrama (~6 min)

Abra o markdown gerado e revise de forma crítica:

- Capturou os recursos implantados (Container App, Environment, ACR, Log Analytics)?
- As relacoes estão corretas? Mostra o pull ACR → Container App?
- Mostra a dependência de Cosmos DB? Se não, está faltando algo importante — o app não funciona sem isso.
- O que está faltando para uma revisão de arquitetura de produção?

**Diga ao Copilot:**

```
What's missing from this architecture for a production deployment? The app also connects to an existing Cosmos DB for its data.
```

Compare as recomendacoes da IA com seus achados do Cenário 1B.

✅ **Checkpoint:** Você tem um diagrama Mermaid mostrando recursos implantados com setas de conexão. Para renderizar: copie o bloco Mermaid da saída do Copilot e cole em [mermaid.live](https://mermaid.live), use VS Code com extensão Mermaid, ou publique o markdown em um repositório GitHub — o GitHub renderiza Mermaid nativamente em arquivos markdown.

**Aprendizado:** `azure-resource-visualizer` e excelente para descoberta ("o que existe agora?") mas exige revisão especializada para documentação ("isso está completo e correto?"). O diagrama reflete o estado implantado dentro de um resource group, não o quadro completo — dependências cross-resource-group como seu Cosmos DB são parte da sua responsabilidade de documentar.

---

**Próximo:** [Cenário 3 — Quebrar e Triar →](06-scenario-3-break-and-triage.md)




