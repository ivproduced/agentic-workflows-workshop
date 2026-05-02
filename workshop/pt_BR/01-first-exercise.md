<!-- l10n-sync: source-file="01-first-exercise.md" -->
# Exercício 1 — Início Rápido com Agentic Workflows

Neste exercício, você configurará as ferramentas do `gh aw` no seu repositório e criará seu primeiro agentic workflow: um **resumo diário** que sumariza issues abertas e pull requests.

**Tempo estimado:** 20 minutos

## Objetivos

- Inicializar Agentic Workflows no seu repositório com `gh aw init`
- Entender os arquivos e a configuração que são criados
- Criar e compilar um workflow de resumo diário para issues e pull requests
- Disparar o workflow manualmente e ler a issue de saída

## Como Funcionam os Agentic Workflows

Agentic workflows são **arquivos markdown em linguagem natural** (`.github/workflows/<name>.md`) com um pequeno bloco de frontmatter YAML no topo. O frontmatter declara coisas como o gatilho, ferramentas necessárias e permissões. O corpo do markdown é um prompt em linguagem natural que instrui o agente de IA sobre o que fazer.

Antes que um workflow possa ser executado no GitHub Actions, ele deve ser **compilado** em um arquivo lock (`.github/workflows/<name>.lock.yml`). Você faz commit tanto do `.md` (legível por humanos) quanto do `.lock.yml` (legível por máquina).

## Parte 1 — Inicializar Agentic Workflows

A partir da raiz do seu repositório, execute:

```bash
gh aw init
```

O comando configura seu repositório para agentic workflows. Ele cria vários arquivos, incluindo:

- `.gitattributes` — marca arquivos lock compilados como gerados
- `.github/aw/github-agentic-workflows.md` — a documentação de referência completa
- `.github/agents/agentic-workflows.agent.md` — um assistente de IA para criar e editar workflows
- `.vscode/settings.json` e `.vscode/mcp.json` — configuração do editor

> [!NOTE]
> `gh aw init` requer acesso de escrita ao repositório. Certifique-se de que está trabalhando no seu fork.

### Inspecione os Arquivos Gerados

Após a conclusão do `init`, explore o que foi criado:

```bash
git status
ls .github/aw/
ls .github/agents/
```

> [!TIP]
> O arquivo `.github/aw/github-agentic-workflows.md` é a referência completa para todas as opções de frontmatter. Abra-o sempre que precisar verificar gatilhos, ferramentas ou permissões suportados.

## Parte 2 — Criar um Resumo Diário para Issues e Pull Requests

Crie seu primeiro workflow usando `gh aw new`:

```bash
gh aw new daily-digest
```

Isso abre uma sessão interativa com o agente de IA `agentic-workflows`. Descreva o que você deseja usando linguagem natural — por exemplo:

```
Every weekday, create a GitHub issue that summarises all open issues
and pull requests in this repository. Group them by label. Include the
total count, the title, the author, and how long each item has been
open. Title the issue "Daily Digest – <date>".
```

O agente fará perguntas de esclarecimento (como qual gatilho usar e se permissões de escrita são necessárias) e, em seguida, gerará o arquivo de workflow para você.

> [!TIP]
> Você também pode executar `gh aw new daily-digest` de forma não interativa fornecendo sua descrição via GitHub Copilot Chat — abra o Copilot Chat, digite `/agent` e selecione **agentic-workflows**.

### O Que é Criado

Após o agente terminar, você terá:

- `.github/workflows/daily-digest.md` — o workflow legível por humanos com frontmatter YAML e seu prompt
- `.github/workflows/daily-digest.lock.yml` — o arquivo compilado legível por máquina para o GitHub Actions

Abra o arquivo markdown para ver o que o agente escreveu:

```bash
cat .github/workflows/daily-digest.md
```

O frontmatter terá uma aparência semelhante a:

```yaml
---
name: Daily Digest
on:
  schedule: daily on weekdays
  workflow_dispatch:
permissions:
  issues: write
  contents: read
safe-outputs:
  create-issue:
    max: 1
---
```

E o corpo é uma descrição em linguagem natural do que o agente deve fazer.

> [!NOTE]
> Arquivos de agentic workflow são markdown comum — faça commit deles no controle de versão como qualquer outro código. O `.lock.yml` é gerado automaticamente e não deve ser editado manualmente.

## Parte 3 — Configurar o Token do Copilot

Os agentic workflows precisam de um PAT (token de acesso pessoal) de granularidade fina para chamar a API do Copilot. Execute o comando de bootstrap interativo apontando para o **seu fork**:

```bash
gh aw secrets bootstrap --repo <seu-usuario>/agentic-workflows-workshop
```

Quando solicitado, abra o link para criar um novo PAT de granularidade fina com as seguintes configurações:

| Configuração | Valor |
|---|---|
| **Nome do token** | `gh-aw-copilot` (ou qualquer nome que preferir) |
| **Proprietário do recurso** | Sua conta **pessoal** do GitHub |
| **Acesso ao repositório** | Todos os repositórios públicos |
| **Permissões → Copilot → Copilot Requests** | Somente leitura |

Gere o token, copie-o e cole no prompt do terminal.

> [!IMPORTANT]
> O proprietário do recurso deve ser a conta do GitHub que possui sua **assinatura ativa do Copilot** (Individual, Pro, Business ou Enterprise). Se usar uma conta de organização, selecione a organização como proprietário do recurso.

> [!TIP]
> Você só precisa executar `gh aw secrets bootstrap` uma vez por repositório. O segredo é armazenado como um secret do GitHub Actions chamado `COPILOT_GITHUB_TOKEN` no seu fork.

## Parte 4 — Disparar o Workflow Manualmente

Faça commit e push dos arquivos gerados, depois dispare o workflow imediatamente para testá-lo:

```bash
git add .gitattributes .github/workflows/daily-digest.md .github/workflows/daily-digest.lock.yml
git commit -m "Add daily digest workflow"
git push
```

Após o push, dispare uma execução manual apontando para o **seu fork**:

```bash
gh aw run daily-digest --repo <seu-usuario>/agentic-workflows-workshop
```

Após a execução ser concluída (geralmente em menos de um minuto), abra o GitHub e verifique a aba **Issues**. Você deve ver uma nova issue intitulada **Daily Digest – \<data de hoje\>**.

> [!NOTE]
> Se o repositório ainda não tiver issues ou PRs abertos, o resumo informará isso. Isso é esperado — o workflow ainda está funcionando corretamente.

## Critérios de Sucesso

- [ ] `gh aw init` foi concluído e criou arquivos em `.github/aw/` e `.github/agents/`
- [ ] `.github/workflows/daily-digest.md` existe no seu repositório
- [ ] `.github/workflows/daily-digest.lock.yml` existe no seu repositório
- [ ] O workflow foi enviado por push e disparado sem erros
- [ ] Uma nova issue no GitHub intitulada **Daily Digest – \<data de hoje\>** foi criada no seu repositório

---

Quando terminar, prossiga para o [Exercício 2: Resumo Diário do Hacker News](./02-second-exercise.md).
