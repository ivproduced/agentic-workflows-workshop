<!-- l10n-sync: source-file="01-first-exercise.md" -->
# Ejercicio 1 — Inicio Rápido con Agentic Workflows

En este ejercicio configurarás las herramientas de `gh aw` en tu repositorio y crearás tu primer flujo de trabajo agéntico: un **resumen diario** que resume los issues abiertos y los pull requests.

**Tiempo estimado:** 20 minutos

## Objetivos

- Inicializar Agentic Workflows en tu repositorio con `gh aw init`
- Comprender los archivos y la configuración que se crean
- Crear y compilar un flujo de trabajo de resumen diario para issues y pull requests
- Ejecutar el flujo de trabajo manualmente y leer el issue generado

## Cómo Funcionan los Agentic Workflows

Los flujos de trabajo agénticos son **archivos markdown en lenguaje natural** (`.github/workflows/<name>.md`) con un pequeño bloque de frontmatter YAML en la parte superior. El frontmatter declara cosas como el disparador, las herramientas requeridas y los permisos. El cuerpo del markdown es un prompt en inglés simple que instruye al agente de IA sobre qué hacer.

Antes de que un flujo de trabajo pueda ejecutarse en GitHub Actions, debe ser **compilado** en un archivo lock (`.github/workflows/<name>.lock.yml`). Haces commit de ambos archivos: el `.md` (legible por humanos) y el `.lock.yml` (legible por máquinas).

## Parte 1 — Inicializar Agentic Workflows

Desde la raíz de tu repositorio, ejecuta:

```bash
gh aw init
```

El comando configura tu repositorio para flujos de trabajo agénticos. Crea varios archivos, incluyendo:

- `.gitattributes` — marca los archivos lock compilados como generados
- `.github/aw/github-agentic-workflows.md` — la documentación de referencia completa
- `.github/agents/agentic-workflows.agent.md` — un asistente de IA para crear y editar flujos de trabajo
- `.vscode/settings.json` y `.vscode/mcp.json` — configuración del editor

> [!NOTE]
> `gh aw init` requiere acceso de escritura al repositorio. Asegúrate de estar trabajando en tu fork.

### Inspeccionar los Archivos Generados

Después de que `init` se complete, explora lo que se creó:

```bash
git status
ls .github/aw/
ls .github/agents/
```

> [!TIP]
> El archivo `.github/aw/github-agentic-workflows.md` es la referencia completa para todas las opciones de frontmatter. Ábrelo siempre que necesites verificar disparadores, herramientas o permisos soportados.

## Parte 2 — Crear un Resumen Diario para Issues y Pull Requests

Crea tu primer flujo de trabajo usando `gh aw new`:

```bash
gh aw new daily-digest
```

Esto abre una sesión interactiva con el agente de IA `agentic-workflows`. Describe lo que quieres usando lenguaje natural — por ejemplo:

```
Every weekday, create a GitHub issue that summarises all open issues
and pull requests in this repository. Group them by label. Include the
total count, the title, the author, and how long each item has been
open. Title the issue "Daily Digest – <date>".
```

El agente hará preguntas de clarificación (como qué disparador usar y si se necesitan permisos de escritura) y luego generará el archivo de flujo de trabajo por ti.

> [!TIP]
> También puedes ejecutar `gh aw new daily-digest` de forma no interactiva proporcionando tu descripción a través de GitHub Copilot Chat — abre Copilot Chat, escribe `/agent` y selecciona **agentic-workflows**.

### Qué Se Crea

Después de que el agente termine, tendrás:

- `.github/workflows/daily-digest.md` — el flujo de trabajo legible por humanos con frontmatter YAML y tu prompt
- `.github/workflows/daily-digest.lock.yml` — el archivo compilado legible por máquinas para GitHub Actions

Abre el archivo markdown para ver lo que el agente escribió:

```bash
cat .github/workflows/daily-digest.md
```

El frontmatter se verá similar a:

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

Y el cuerpo es una descripción en lenguaje natural de lo que el agente debe hacer.

> [!NOTE]
> Los archivos de flujos de trabajo agénticos son markdown normal — haz commit de ellos en el control de versiones como cualquier otro código. El `.lock.yml` se genera automáticamente y no debe editarse manualmente.

## Parte 3 — Configurar el Token de Copilot

Los flujos de trabajo agénticos necesitan un PAT (token de acceso personal) de grano fino para llamar a la API de Copilot. Ejecuta el comando de bootstrap interactivo, apuntando a **tu fork**:

```bash
gh aw secrets bootstrap --repo <tu-usuario>/agentic-workflows-workshop
```

Cuando se te solicite, abre el enlace para crear un nuevo PAT de grano fino con la siguiente configuración:

| Ajuste | Valor |
|---|---|
| **Nombre del token** | `gh-aw-copilot` (o cualquier nombre que prefieras) |
| **Propietario del recurso** | Tu cuenta **personal** de GitHub |
| **Acceso al repositorio** | Todos los repositorios públicos |
| **Permisos → Copilot → Copilot Requests** | Solo lectura |

Genera el token, cópialo y pégalo en el prompt del terminal.

> [!IMPORTANT]
> El propietario del recurso debe ser la cuenta de GitHub que tiene tu **suscripción activa de Copilot** (Individual, Pro, Business o Enterprise). Si usas una cuenta de organización, selecciona la organización como propietario del recurso.

> [!TIP]
> Solo necesitas ejecutar `gh aw secrets bootstrap` una vez por repositorio. El secreto se almacena como un secreto de GitHub Actions llamado `COPILOT_GITHUB_TOKEN` en tu fork.

## Parte 4 — Ejecutar el Flujo de Trabajo Manualmente

Haz commit y push de los archivos generados, luego ejecuta el flujo de trabajo inmediatamente para probarlo:

```bash
git add .gitattributes .github/workflows/daily-digest.md .github/workflows/daily-digest.lock.yml
git commit -m "Add daily digest workflow"
git push
```

Una vez subido, ejecuta una ejecución manual apuntando a **tu fork**:

```bash
gh aw run daily-digest --repo <tu-usuario>/agentic-workflows-workshop
```

Después de que la ejecución se complete (generalmente en menos de un minuto), abre GitHub y revisa la pestaña **Issues**. Deberías ver un nuevo issue titulado **Daily Digest – \<fecha de hoy\>**.

> [!NOTE]
> Si el repositorio aún no tiene issues o PRs abiertos, el resumen lo indicará. Esto es esperado — el flujo de trabajo sigue funcionando correctamente.

## Criterios de Éxito

- [ ] `gh aw init` se completó y creó archivos en `.github/aw/` y `.github/agents/`
- [ ] `.github/workflows/daily-digest.md` existe en tu repositorio
- [ ] `.github/workflows/daily-digest.lock.yml` existe en tu repositorio
- [ ] El flujo de trabajo se subió y se ejecutó sin errores
- [ ] Se creó un nuevo issue de GitHub titulado **Daily Digest – \<fecha de hoy\>** en tu repositorio

---

Una vez terminado, continúa con el [Ejercicio 2: Resumen Diario de Hacker News](./02-second-exercise.md).
