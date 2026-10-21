## Padrões de automação QA

- Todo arquivo .feature usa Reqnroll (C# BDD)
- Sempre inclua as tags @{id} e @MAT para rastreabilidade com TestRail
- Feature files ficam em `MyProject.SystemTests/Features/`
- Step definitions ficam em `MyProject.SystemTests/Features/StepDefinitions/` com sufixo `StepDefinitions.cs`
- Use `[Binding]` e `[Given]`, `[When]`, `[Then]` do namespace Reqnroll
- Nunca use Thread.Sleep — use async/await com Polly para retry
