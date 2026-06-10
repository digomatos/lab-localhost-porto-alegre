# Antes de começar — Login e Inicialização

## 1. Faça login no Azure

Abra um terminal — use o atalho (Ctrl + Shift + 4) para abrir o PowerShell — e conclua estes passos.

```bash
az login
```

Quando o pop-up de login aparecer, selecione **Work or school account** e clique em **Continue**. Informe o usuário encontrado na aba **Resources** da sua VM Skillable clicando no ícone de teclado e selecione **Next**. Depois, informe o TAP encontrado na mesma aba clicando no ícone de teclado para concluir o login. No diálogo **Sign in to all apps and websites on this device?**, clique em **Yes**.

Quando o terminal pedir a seleção da subscription, pressione **Enter** para não alterar.

> ⚠️ **NÃO selecione "Microsoft account" (pessoal/consumidor).** A página de login pode mostrar múltiplas opções — selecione sempre **Work or school account**. Escolher a opção errada resulta em erros de acesso negado.

## 2. Faça login no Azure Developer CLI

```bash
azd auth login
```

Selecione a conta Azure do passo anterior e conclua a autenticação.

## 3. Faça login no GitHub

Abra este link no navegador: <a href="https://github.com/enterprises/skillable-events/sso" target="_blank" rel="noopener noreferrer">https://github.com/enterprises/skillable-events/sso</a>. Selecione **Continue** quando solicitado para single sign-on no Skillable Events. Selecione a conta Azure que você acabou de autenticar. Siga os prompts para concluir a autenticação.

## 4. Faça login no GitHub Copilot CLI

Digite o comando abaixo para iniciar o GitHub Copilot CLI:

```bash
copilot
```

Isso abre a sessão interativa do Copilot CLI. Todos os prompts "Say to Copilot" deste lab são digitados aqui. **Mantenha está sessão aberta durante todo o lab** — e aqui que você interage com as skills de IA.

> 💡 **Terminal vs. Copilot:** Durante este lab, você vai executar comandos em dois lugares. O **Copilot CLI** e para prompts orientados por IA (por exemplo, "Deploy my app to Azure"). **Comandos de terminal** (prefixados com `!` no Copilot) são para operacoes de shell como `curl`, `az` e `git`. Em caso de duvida, você pode executar qualquer comando de terminal dentro do Copilot prefixando com `!`.

```bash
/login
```

Quando for perguntado em qual conta fazer login, selecione GitHub.com. O Copilot pedirá para você pressionar qualquer tecla para abrir o navegador e concluir o login. Siga as instruções no Copilot para concluir a autorização usando a conta autenticada.

## 5. Desabilite o agente Rubberduck

Use o prompt abaixo no Copilot para desabilitar o agente rubberduck no Copilot CLI, pois ele não e necessario para está sessão de lab:

Say to Copilot
```
 Update the settings.json for Copilot CLI to disable rubber duck with this, "builtInAgents": {"rubberDuck": false},
```

![Desabilitando o agente Rubber Duck no Copilot CLI](images/disablingRubberDuck.png)

## 6. Instale o Azure Skills Plugin

1. Adicione o marketplace Microsoft:
   ```
   /plugin marketplace add microsoft/azure-skills
   ```

2. Instale o plugin Azure:
   ```
   /plugin install azure@azure-skills
   ```

3. Recarregue o Azure MCP:
   ```
   /mcp reload
   ```
4. **FECHE O TERMINAL** para que as alterações nas configurações do Copilot sejam carregadas na próxima vez que você abrir o Copilot.

> 💡 **MCP tools vs. Azure skills:** O Azure MCP server fornece **MCP tools** — operacoes de baixo nível como listar recursos, consultar logs e gerenciar deployments. As **skills** do Azure são instruções de nível mais alto que encadeiam essas ferramentas com conhecimento de dominio (por exemplo, `azure-diagnostics` sabe seguir uma cadeia de raciocínio de triagem). Este lab usa ambos: as skills conduzem o fluxo, e os MCP tools executam as operacoes no Azure.

> 💡 **Dica:** Para atualizar o plugin depois, execute:
> ```
> /plugin update azure@azure-skills
> ```

✅ **Checkpoint:** Você está autenticado no GitHub e no Azure, o Copilot CLI está em execução e as skills do Azure e o Azure MCP Server estão instalados.

---

**Próximo:** [Configurar o App Inicial →](03-getting-started.md)




