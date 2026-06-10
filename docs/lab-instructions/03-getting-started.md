# Primeiros passos — Configure o App Inicial

Configure o app inicial usando as instrucoes abaixo.

## 1. Clone o repositorio do lab

Abra uma nova sessao PowerShell, clone o repositorio do lab e navegue ate ele:

```powershell
git clone https://github.com/microsoft/Build26-LAB501.git
```
```powershell
cd Build26-LAB501
```

## 2. Copie o app inicial

O diretorio `src/` contem uma aplicacao Python Flask pronta para uso — um navegador de conjuntos LEGO com dados no Azure Cosmos DB. Copie para um novo diretorio de trabalho `lego-set-browser` e inicialize como um repositorio Git proprio:

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
git config --global user.email "voce@exemplo.com"
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
Se voce encontrar algum problema, tente digitar os comandos manualmente e execute um comando por vez.

Todos os comandos seguintes devem ser executados a partir do diretorio `lego-set-browser`.

> 💡 **O que existe no app inicial?** `app.py` e uma aplicacao web Flask com rotas para navegar, pesquisar e visualizar conjuntos LEGO. Ela se conecta ao Azure Cosmos DB para consultar dados dos conjuntos. `requirements.txt` define as dependencias Python (Flask, azure-cosmos, azure-identity, gunicorn). Um `Dockerfile` esta incluido para deployment em container. O app usa `DefaultAzureCredential` para autenticacao sem senha no Cosmos DB.

## 3. Teste local

Vamos pular intencionalmente os testes locais durante os labs presenciais para ter mais tempo de concluir o lab.

---

**Proximo:** [Cenario 1 — Publicar e Fortalecer →](04-scenario-1-ship-and-harden.md)
