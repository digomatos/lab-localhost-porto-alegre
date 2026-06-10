# Primeiros passos — Configure o App Inicial

Configure o app inicial usando as instruções abaixo.

## 1. Clone o repositório do lab

Abra uma nova sessão PowerShell, clone o repositório do lab e navegue ate ele:

```powershell
git clone https://github.com/microsoft/Build26-LAB501.git
```
```powershell
cd Build26-LAB501
```

## 2. Copie o app inicial

O diretório `src/` contem uma aplicação Python Flask pronta para uso — um navegador de conjuntos LEGO com dados no Azure Cosmos DB. Copie para um novo diretório de trabalho `lego-set-browser` e inicialize como um repositório Git proprio:

```powershell
Copy-Item -Recurse src lego-set-browser
```
```powershell
cd lego-set-browser
```
```powershell
git config --global user.name "Seu Nome"
```
```powershell
git config --global user.email "você@exemplo.com"
```
```powershell
git init
```
```powershell
git add -A
```
```powershell
git commit -m "init"
```
Se você encontrar algum problema, tente digitar os comandos manualmente e execute um comando por vez.

Todos os comandos seguintes devem ser executados a partir do diretório `lego-set-browser`.

> 💡 **O que existe no app inicial?** `app.py` e uma aplicação web Flask com rotas para navegar, pesquisar e visualizar conjuntos LEGO. Ela se conecta ao Azure Cosmos DB para consultar dados dos conjuntos. `requirements.txt` define as dependências Python (Flask, azure-cosmos, azure-identity, gunicorn). Um `Dockerfile` está incluido para deployment em container. O app usa `DefaultAzureCredential` para autenticação sem senha no Cosmos DB.

## 3. Teste local

Vamos pular intencionalmente os testes locais durante os labs presenciais para ter mais tempo de concluir o lab.

---

**Próximo:** [Cenário 1 — Publicar e Fortalecer →](04-scenario-1-ship-and-harden.md)




