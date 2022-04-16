Desafio Final - AppService-SQLdb-DevOps
# IGTI - MBA Cloud: criação de ambiente no Azure para Implantação de Web App utilizando App Service, SQL Database e Pipeline CI/CD

## Configuração do ambiente da Aplicação Web

**1. Fork do repositório (código fonte da aplicação) para o repositório Github pessoal (MBACloud-az-AppService-SQLdb-DevOps)**

https://github.com/Azure-Samples/msdocs-app-service-sqldb-dotnetcore

**2. Criação de um Azure Web App S1 (App Service) com dois Slots para implantação, sendo um para produção e outro para desenvolvimento**
- criação do App Service com slot de produção (production):
  - App Services -> Create (waMyTodoList)
- criação de um slot de implantação para desenvolvimento separado do slot de produção:
  - App Service - Deployment slots -> Add Slot (dev)

**3. Criação de dois bancos Azure SQL Database Standard S0 chamado coreDB, sendo um para produção e um para desenvolvimento**
- criação da instância SQL Server:
    - SQL Servers -> Create
- criação da base de dados:
  - SQL Databases -> Create

- Instância e base de dados para produção (coredbserver-m05/coreDB) 
- Instância e base de dados para desenvolvimento (coredbserver-m05dev/coreDB):

## Configuração do ambiente do Azure DevOps

**4. Na sua organização dentro do Azure DevOps, criar um Novo Projeto**
- \+ New project

**5. Importação do repositório no Gihub para o Azure DevOps – Repos.**
(temos no repositório o código da aplicação e o schema do banco de dados)

- Azure DevOps -> Repos -> Import
- Clonar repositório para realizar testes de alterações (repo "AppService-SQLdb-dotnetcore"):
```
git clone https://[clone_URL]
```
- Realizar commit inicial no VSCode (local):
```
git add .
git commit -m "commit inicial VSCode"
git push
```
## Deploy do Web App - configuração do processo de DevOps CI/CD

**6. Deploy do App Service (ambiente de desenvolvimento):**
- App Service -> Deployment slots -> selecione o slot para desenvolvimento
- Slot-dev -> Deployment Center: configurar CI/CD para fonte (source) no Azure Repos

## Configuração da conexão com o banco de dados e geração das bases de dados

**7. Conectando o WebApp ao banco de dados SQL**
- cópia da Connection strings (ADO.NET) das Bases de Produção e Desenvolvimento

prod:
```
Server=tcp:coredbserver-m05.database.windows.net,1433;Initial Catalog=coreDB;Persist Security Info=False;User ID={your_user};Password={your_password};MultipleActiveResultSets=False;Encrypt=True;TrustServerCertificate=False;Connection Timeout=30;
```
dev:
```
Server=tcp:coredbserver-m05dev.database.windows.net,1433;Initial Catalog=coreDB;Persist Security Info=False;User ID={your_user};Password={your_password};MultipleActiveResultSets=False;Encrypt=True;TrustServerCertificate=False;Connection Timeout=30;
```
- configuração do App Service no slot de Produção:
  - Configuration -> Connection strings -> New Connection string (MyDbConnection)
  (marcar opção "Deployment slot setting")
- configuração do App Service no slot de Desenvolvimento:
  - Configuration -> Connection strings -> New Connection string (MyDbConnection)
  (marcar opção "Deployment slot setting")

**8. Geração das Bases de Dados de Produção e Desenvolvimento**
- CLI: rodar comando para instalar "Entity Framework Core"
```
dotnet tool install -g dotnet-ef
```
- VSCode: edição local do arquivo appsettings.json adicionando a Connection string (Prod)
- CLI: rodar comandos para criação inicial da base de dados de produção
```
dotnet ef migrations add InitialCreate
dotnet ef database update
```
- VSCode: edição local do arquivo appsettings.json adicionando a Connection string (Dev)
- CLI: rodar comando para criação inicial da base de dados de desenvolvimento
```
dotnet ef database update
```
## Testes de alteração no código e Deploy em desenvolvimento

**9. Alterações no código da aplicação e deploy em desenvolvimento**
- VS Code: realizar alterações no código da aplicação
- CLI: rodar comandos para commit e deploy das alterações (Azure Repos)
```
git add .
git commit -m "alteracoes de teste"
git push
```
(a partir daqui o pipeline de CI/CD do *Deployment Center* será executado para deploy no Slot de Desenvolvimento)

## Deploy do Web App em Produção

**10. Deploy do App a partir do Slot de desenvolvimento para o Slot de produção**
- Swap manual:
  - App Service -> Deployment Slots -> Swap
- Auto Swap - realizar configuração no App Service para realizar swap automático (slot dev -> slot prod):
  - App Service -> Deployment slots -> selecione o slot para desenvolvimento
  - Slot-dev -> Configuration -> General settings -> Auto swap enabled: On ; deployment: production

**11. Rotear manualmnete o tráfego entre o slot de produção e desenvolvimento a partir da URL**
- utilizar na URL a query: `?x-ms-routing-name={slot_name}`
```
URL: <webappname>.azurewebsites.net/?x-ms-routing-name={slot_name}
```
- por exemplo:
  - slot de desenvolvimento (dev):
    `https://wamytodolist.azurewebsites.net/?x-ms-routing-name=dev`
  - slot de produção (self):
    `https://wamytodolist.azurewebsites.net/?x-ms-routing-name=self`
    
*Seria um cenário interessante aonde você pode ocultar o slot de desenvolvimento para o público mas possibilitar que sua equipe interna realize testes nesse slot.*

##
**André Carlucci**
