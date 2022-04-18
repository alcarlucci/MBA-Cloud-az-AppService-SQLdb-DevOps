Desafio Final - AppService-SQLdb-DevOps
# IGTI - MBA Cloud: criação de ambiente no Azure para Implantação de Web App utilizando App Service, SQL Database e Pipeline CI/CD

## Configuração do ambiente da Aplicação Web

**1. Fork do repositório (código fonte da aplicação) para o repositório Github pessoal (MBACloud-az-AppService-SQLdb-DevOps)**

https://github.com/Azure-Samples/msdocs-app-service-sqldb-dotnetcore

**2. Criação de um Azure Web App (App Service) no plano S1 (App Service plan) com dois Slots para implantação, sendo um para produção e outro para desenvolvimento**
- criação do App Service com slot de produção (production):
  - App Services -> Create (waMyTodoList)
- criação de um slot de implantação para desenvolvimento separado do slot de produção:
  - App Service - Deployment slots -> Add Slot (dev)

**App Service plan - App e Slot:**
![image](https://user-images.githubusercontent.com/101406714/163691800-82b37187-c01b-4d3d-bef0-f04a56150d64.png)

**Slots de Implantação do App Service:**
![image](https://user-images.githubusercontent.com/101406714/163692002-7a2ceaaa-8a18-48e5-943b-221b834d7516.png)

**3. Criação de dois bancos Azure SQL Database Standard S0 chamado coreDB, sendo um para produção e um para desenvolvimento**
- criação da instância SQL Server:
    - SQL Servers -> Create
- criação da base de dados:
  - SQL Databases -> Create

**Instância e base de dados para produção (coredbserver-m05/coreDB):**
![image](https://user-images.githubusercontent.com/101406714/163692112-bbfb6b65-f8d8-4409-989d-514289205bc6.png)

**Instância e base de dados para desenvolvimento (coredbserver-m05dev/coreDB):**
![image](https://user-images.githubusercontent.com/101406714/163692173-2cf8e6e3-2bd1-4171-9187-e60228d738c4.png)

## Configuração do ambiente do Azure DevOps

**4. Na sua organização dentro do Azure DevOps, criar um Novo Projeto**
- \+ New project

![image](https://user-images.githubusercontent.com/101406714/163692481-0a40ed1a-c568-44c8-a7f9-39c95d1276a7.png)

**5. Importação do repositório no Gihub para o Azure DevOps – Repos.**
(temos no repositório o código da aplicação e o schema do banco de dados)

- Azure DevOps -> Repos -> Import

![image](https://user-images.githubusercontent.com/101406714/163692677-3d942687-518d-4996-8421-dedd1962a908.png)

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
![image](https://user-images.githubusercontent.com/101406714/163692771-4606e24f-12b1-40df-937c-9690bd1f32e3.png)

## Deploy do Web App - configuração do processo de DevOps CI/CD

**6. Deploy do App Service (ambiente de desenvolvimento):**
- App Service -> Deployment slots -> selecione o slot para desenvolvimento
- Slot-dev -> Deployment Center: configurar CI/CD para fonte (source) no Azure Repos

![image](https://user-images.githubusercontent.com/101406714/163692898-110bec43-b704-4659-b7be-f626571e206a.png)

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
- configuração da conexão do App Service no slot de Produção:
  - Configuration -> Connection strings -> New Connection string (MyDbConnection)
  (marcar opção "Deployment slot setting")

![image](https://user-images.githubusercontent.com/101406714/163693378-761502a7-ed8e-4f4d-a6e3-b8afc830223a.png)

- configuração da conexão do App Service no slot de Desenvolvimento:
  - Configuration -> Connection strings -> New Connection string (MyDbConnection)
  (marcar opção "Deployment slot setting")
  
![image](https://user-images.githubusercontent.com/101406714/163693436-5f9a042a-bc5f-490a-83c6-a3f1d6508a81.png)

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
git commit -m "index - alteracoes de teste"
git push
```
**Execução do pipeline de CI/CD do *Deployment Center* para o deploy no Slot de Desenvolvimento:**

![image](https://user-images.githubusercontent.com/101406714/163693054-0182060f-4563-48fd-a0f8-4842ffbf9809.png)

**Detalhes da execução do pipeline:**

![image](https://user-images.githubusercontent.com/101406714/163693189-38f39712-6ce5-4e6d-9578-5869ded5ece7.png)

## Deploy do Web App em Produção

**10. Deploy do App a partir do Slot de desenvolvimento para o Slot de produção**
- Swap manual:
  - App Service -> Deployment Slots -> Swap

![image](https://user-images.githubusercontent.com/101406714/163840876-5ed7170b-963d-4e26-91e5-67593063a438.png)

- Auto Swap - realizar configuração no App Service para realizar swap automático (slot dev -> slot prod):
  - App Service -> Deployment slots -> selecione o slot para desenvolvimento
  - Slot-dev -> Configuration -> General settings -> Auto swap enabled: On ; deployment: production

![image](https://user-images.githubusercontent.com/101406714/163841244-7dddf91c-9f2a-4947-8ede-e2d0fc4ed1ee.png)

**Exemplo do aplicação publicada já com algumas alterações realizadas e implantadas:**
![image](https://user-images.githubusercontent.com/101406714/163839098-624ae0e1-8a79-4b5a-ae04-3a1918d65ecf.png)

**11. Rotear manualmnete o tráfego entre o slot de produção e desenvolvimento a partir da URL**
- adicionar na URL a query: `?x-ms-routing-name={slot_name}`
```
URL: <webappname>.azurewebsites.net/?x-ms-routing-name={slot_name}
```
- por exemplo:
  - slot de desenvolvimento (dev): `https://wamytodolist.azurewebsites.net/?x-ms-routing-name=dev`
  ![image](https://user-images.githubusercontent.com/101406714/163839679-22c56b3f-b6f9-425b-acbb-a92f7110f0c4.png)

  - slot de produção (self): `https://wamytodolist.azurewebsites.net/?x-ms-routing-name=self`
  ![image](https://user-images.githubusercontent.com/101406714/163839974-f3197379-933e-4bf5-9365-8e754aec3f2b.png)
    
*Seria um cenário interessante aonde você pode ocultar o slot de desenvolvimento para o público mas possibilitar que sua equipe interna realize testes nesse slot.*

##
**André Carlucci**
