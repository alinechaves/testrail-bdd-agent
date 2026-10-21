# TestRail → BDD com Agentes de IA e MCP

Automação do fluxo **caso de teste no TestRail → feature file Gherkin → step definitions C#** usando um agente de IA (GitHub Copilot em modo *agent*) conectado ao TestRail por meio de um servidor **MCP (Model Context Protocol)**.

Em vez de copiar manualmente os passos de um caso do TestRail e escrever o Gherkin e o scaffold em C#, basta informar o **ID do caso**: o agente busca o caso, gera o `.feature` com as tags de rastreabilidade e cria os step definitions prontos para implementação.

## O que o projeto faz

1. **Busca o caso no TestRail** via MCP (`getBdd` para casos com template BDD, `getCase` para casos com template de Steps).
2. **Gera o feature file** em Gherkin para [Reqnroll](https://reqnroll.net/) (BDD em C#), com as tags `@{id}` e `@MAT` para rastrear o cenário até o caso no TestRail.
3. **Gera os step definitions** em C# (`[Binding]`, métodos `async Task`, `PendingStepException` como placeholder), inferindo as dependências necessárias (WebDriver, HttpClient, banco etc.) e reaproveitando steps que já existem no projeto.
4. **Informa o que falta**: lista os arquivos criados e os steps que ainda precisam de implementação real.

```
 ID do caso ──► [MCP TestRail] ──► bdd-generator ──► .feature
                  getBdd/getCase                        │
                                                        ▼
                                    step-definition-generator ──► *StepDefinitions.cs
```

## Estrutura

```
.github/
├── copilot-instructions.md              # Padrões de automação QA aplicados a todo o repositório
├── prompts/
│   └── gen-feature.prompt.md            # Prompt que orquestra o fluxo completo a partir de um ID
└── skills/
    ├── bdd-generator/SKILL.md           # Skill: caso do TestRail → feature file Reqnroll
    └── step-definition-generator/SKILL.md  # Skill: feature file → step definitions C#
```

| Arquivo | Papel |
|---|---|
| `copilot-instructions.md` | Regras gerais: Reqnroll, tags obrigatórias, local dos arquivos, sem `Thread.Sleep` (usar async/await + Polly). |
| `gen-feature.prompt.md` | Prompt reutilizável que recebe o `case_id` e encadeia as duas skills. |
| `bdd-generator` | Converte o caso em Gherkin. Mapeia pré-condições → `Given`, primeiro passo → `When`, demais → `And`, resultado esperado → `Then`. Usa `Scenario Outline` quando há dados parametrizados. |
| `step-definition-generator` | Extrai os steps únicos, tipa os parâmetros (`string`, `int`, `Table`), evita duplicação e injeta as dependências adequadas no construtor. |

## Pré-requisitos

- VS Code com **GitHub Copilot** (modo *agent* habilitado)
- Um **servidor MCP do TestRail** configurado no VS Code, expondo as ferramentas `getCase` e `getBdd`
- Projeto de testes em **.NET** com **Reqnroll**

## Como usar

1. Copie a pasta `.github/` para a raiz do seu repositório de testes.
2. Ajuste os caminhos em `copilot-instructions.md` e nas skills para o nome do seu projeto de testes (por padrão `MyProject.SystemTests/Features/`).
3. Configure o servidor MCP do TestRail no VS Code.
4. No chat do Copilot, em modo *agent*, execute o prompt:

   ```
   /gen-feature
   ```

   e informe o ID do caso no TestRail (ex.: `1234`).

### Exemplo de saída

```gherkin
Feature: Login

  @1234 @MAT
  Scenario: Login com credenciais válidas
    Given que o usuário está na página de login
    When ele informa usuário e senha válidos
    And clica em "Entrar"
    Then deve ser redirecionado para a página inicial
```

```csharp
[Binding]
public sealed class LoginSteps
{
    private readonly IWebDriver _driver;

    public LoginSteps(IWebDriver driver) => _driver = driver;

    [Given("que o usuário está na página de login")]
    public async Task GivenQueOUsuarioEstaNaPaginaDeLogin()
        => throw new PendingStepException();

    // ...
}
```

## Regras de geração

- As tags `@{id}` (somente o número, sem `C` ou zeros à esquerda) e `@MAT` são sempre adicionadas.
- O idioma do cenário segue o idioma do caso no TestRail.
- Arquivos `.feature` ou de steps existentes nunca são sobrescritos sem aviso.
- Até 2 tentativas por método MCP; se `getBdd` falhar, o agente recorre ao `getCase`.

## Tecnologias

GitHub Copilot (agent mode) · Model Context Protocol (MCP) · TestRail · Reqnroll · C# / .NET · Gherkin
