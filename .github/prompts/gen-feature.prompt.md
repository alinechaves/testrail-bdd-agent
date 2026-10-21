---
agent: agent
description: Gera feature file e step definitions a partir de um ID do TestRail
---

## Objetivo

Dado o caso de teste de ID {{input:case_id}} no TestRail, execute o fluxo
completo de geração: feature file Reqnroll + step definitions C# scaffolded.

## Passo 1 — buscar o caso no TestRail

- Use a ferramenta MCP `getCase` ou `getBdd` do servidor testrail para
   buscar o caso pelo ID informado.
- Se o template for BDD: chame `getBdd({{input:case_id}})`
- Se o template for Steps: chame `getCase({{input:case_id}})` e leia os campos
  `custom_preconds`, `custom_steps_separated` e `custom_expected`

## Passo 2 — gerar o feature file

Aplique a skill `bdd-generator`:
- Monte o Gherkin a partir dos dados retornados
- Adicione a tag `@{{input:case_id}}` e `@MAT` antes do `Scenario:`
- Use `Scenario Outline` + `Examples` se houver dados parametrizados
-Verifique se já existe um arquivo StepDefinitions/{secao}StepDefinitions.cs:
   - Se existir, leia os métodos já implementados para evitar duplicação
   - Se não existir, salve em `Features/{secao}.feature`


## Passo 3 — gerar os step definitions

Aplique a skill `step-definition-generator` sobre o arquivo recém-criado:
- Leia o .feature gerado no passo anterior
- Analise os steps e infira as dependências necessárias (driver, http, db…)
- Gere `StepDefinitions/{secao}StepDefinitions.cs` com métodos async,
  `[Binding]`, `PendingStepException()` como placeholder
- Se houver steps idênticos a outros já existentes no projeto, reutilize
  o binding existente em vez de duplicar

## Passo 4 — confirmar

Liste os arquivos criados e os steps que ainda precisam de implementação real.
Pergunte se deve gerar também o Page Object ou helper de serviço correspondente.

## Regras gerais

- Namespace: leia o .csproj para usar o namespace correto do projeto
- Idioma dos cenários: mantenha o mesmo idioma do caso no TestRail
- Nunca sobrescreva um .feature ou Steps.cs que já existe sem avisar

<critical>
Nao adicione resultados de testes executados via MCP ou qualquer outro dado que nao seja o caminho do arquivo gerado e os steps que precisam de implementacao. O objetivo desta skill é gerar os arquivos .feature e Steps.cs, nao executar testes ou validar resultados.
</critical>
<critical>Sempre use somente os metodos `getCase`ou `getBdd`</critical>
