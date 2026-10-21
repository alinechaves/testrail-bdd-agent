---
name: step-definition-generator
description: Gera step definitions C# com Reqnroll a partir de um arquivo
             .feature existente no projeto. Use quando o usuário pedir para
             gerar, criar ou fazer o scaffold dos steps de um cenário BDD.
---

## O que fazer

1. Leia o arquivo .feature indicado pelo usuário (ou o arquivo aberto no editor).
2. Extraia todos os steps únicos: Given, When, Then, And, But.
3. Para cada step, identifique:
   - Parâmetros entre aspas "valor"  → string
   - Parâmetros numéricos 123        → int ou decimal
   - Tabelas DataTable               → Table table
   - Scenario Outline                → parâmetros via <coluna>
4. Verifique se já existe um arquivo StepDefinitions/{secao}StepDefinitions.cs:
   - Se existir, leia os métodos já implementados para evitar duplicação
   - Se não existir, crie um novo arquivo com a estrutura básica
5. Após gerar, liste os steps que precisam de implementação real
   e pergunte se deve gerar também o Page Object ou Service helper.

## Padrões obrigatórios

- Namespace: deve corresponder ao namespace do projeto (leia o .csproj)
- Classe: sealed, com sufixo Steps
- Construtor: recebe dependências via injeção (IWebDriver, ScenarioContext, etc.)
- Métodos: sempre async Task, nunca void
- Atributos: [Given], [When], [Then] do namespace Reqnroll
- Regex dos steps: use expressões simples, prefira {string} e {int}
  em vez de regex manual sempre que possível
- Corpo dos métodos: throw new PendingStepException() como placeholder
- Steps compartilhados entre features: extraia para SharedSteps.cs separado

## Mapeamento de contexto

Analise o conteúdo dos steps para inferir quais dependências injetar:

| Steps mencionam          | Injetar no construtor              |
|--------------------------|------------------------------------|
| página, botão, campo     | IWebDriver driver                  |
| API, endpoint, requisição| HttpClient httpClient              |
| banco, registro, tabela  | IDbConnection dbConnection         |
| usuário, login, sessão   | ScenarioContext scenarioContext     |
| arquivo, upload, download| IFileService fileService           |
