---
name: bdd-generator
description: Gera feature files Reqnroll/Gherkin a partir de casos do TestRail.
             Use quando o usuário pedir para gerar, criar ou converter cenários
             de teste do TestRail para BDD ou .feature.
---

## Como executar

1. Use a ferramenta MCP `getCase`ou `getBdd` do servidor testrail para
   buscar o caso pelo ID informado.
2. Se o template for BDD, use `getBdd` para exportar o cenário diretamente.
3. Se o template for Steps, extraia `custom_preconds`, `custom_steps_separated`
   e `custom_expected`, e monte o Gherkin assim:
   - custom_preconds → linha(s) `Given`
   - primeiro step → `When`
   - steps seguintes → `And`
   - expected result → `Then`
4. Salve o arquivo em `MyProject.SystemTests/Features/{secao}.feature`
5. Adicione as tags `@{id}` e `@MAT` antes do `Scenario:`
6. Ajuste o `@{id}` para usar apenas o número do ID, sem prefixos ou zeros à esquerda (ex: `@1234` em vez de `@C1234` ou `@0001234`) 
7. Após gerar o .feature, sinalize o fluxo que o arquivo foi criado e informe o caminho completo para a skill step-definition-generator possa usar no proximo passo.

## Template de feature
```gherkin
Feature: {Titulo da Feature}

  @{id} @MAT
  Scenario: {Titulo do cenario}
    Given {precondition}
    When {acao principal}
    And {acao adicional}
    Then {resultado esperado}
```

## Regras

- Nunca omita as tags @{id} e @MAT
- Use Scenario Outline + Examples se houver dados parametrizados
- Texto em português se o caso original estiver em português
- Somente execute no maximo 2 tentativas por metodo, se falhar 2 vezes o `getBdd` tente pelo `getCase`