### GitHub Actions: minimum, żeby ruszyć
Agent CI/CD musi działać na runnerze, nie na laptopie. GHA automatyzuje build/test/deploy bez dedykowanych serwerów — wystarczy plik YAML w `.github/workflows/`.

Hierarchia GHA: **Workflow** (scenariusz YAML) → **Trigger** (`push`, `pull_request`, `workflow_dispatch`) → **Job** (na runnerze) → **Step** → **Action** (reużywalny plugin). **Runner**: `ubuntu-latest` (Node.js), `macos-latest`, `windows-latest`. Publiczne repo: darmowe minuty na standardowych runnerach.

Minimalny workflow — PR do `master` + ręczny trigger, sekrety zamiast klucza w pliku:

```yaml
# .github/workflows/review.yml
name: AI Code Review

on:
  pull_request:
    branches: [master]
  workflow_dispatch:

jobs:
  review:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout repository
        uses: actions/checkout@v4

      - name: Configure node
        uses: actions/setup-node@v4
        with:
          node-version-file: '.nvmrc'

      - name: Install dependencies
        run: npm ci

      - name: Run agent
        run: npm run review
        env:
          LLM_PROVIDER_API_KEY: ${{ secrets.LLM_PROVIDER_API_KEY }}
```

Każdy PR odpala agenta; wynik trafia do pull requesta. Kolejny krok: Definition of Done + bramka merge.

### Composite Action: agent jako reużywalny plugin
Antywzorzec **kopiowania `review.yml` do wielu repo** — kopie się rozjeżdżają. **Composite Action** wyciąga kroki do parametryzowanej jednostki wpinanej w wiele scenariuszy (vs reusable workflows dla całych procesów).

Hosting: (1) osobne repo → `uses: twoj-zespol/ai-reviewer@<sha>`; (2) lokalnie `.github/actions/<nazwa>/` → `uses: ./.github/actions/ai-reviewer`. Na start: opcja 2.

Serce: `action.yml` z `using: composite`. Krok `run` wymaga jawnego `shell`. Skrypty przez `${{ github.action_path }}`. `inputs` — traktuj jak nieufny input. `outputs` — werdykt pod bramkę merge.

```yaml
# action.yml
name: AI Reviewer
description: Run the team's code review agent

inputs:
  api-key:
    description: API key
    required: true

outputs:
  verdict:
    description: Pass/fail verdict from the agent
    value: ${{ steps.agent.outputs.verdict }}

runs:
  using: composite
  steps:
    - name: Run agent
      id: agent
      run: node ${{ github.action_path }}/dist/review.js
      shell: bash
      env:
        LLM_PROVIDER_API_KEY: ${{ inputs.api-key }}
```

Konsument kurczy się do jednego kroku. **Bezpieczeństwo:** przypinaj akcje do commita `@<sha>`, nie do ruchomego `@v1` — cudzy kod z dostępem do sekretów.

### Integracja GitHub Actions z Agentem do Code Review
10x-workflow: research → plan → implementacja. Zamiast pełnego shape'ingu wystarczy `requirements.md` w folderze zmiany.

```text
/10x-new ci-cd-code-review introducing first ci/cd workflow for pr code reviews
```

MVP wymagań (wąski zakres jak MVP aplikacji):
- **Koncepcja** — workflow na każdy PR do `master`, review w composite action.
- **Wejście** — tytuł PR, opis (świadomy kompromis kosztowy), `git diff`.
- **Kryteria** — sześć wymiarów 1–10 (poprawność, idiomatyczność, złożoność, testy/ryzyko, dokumentacja, bezpieczeństwo); stany na „1" i „10".
- **Na później** — dopasowanie biznesowe, architektoniczne (wymagają szerszego kontekstu).
- **Efekty** — komentarz w PR, etykiety `ai-cr:passed` / `ai-cr:failed`.
- **Retry** — etykieta `ai-cr:review`.

Tytuł i opis z `github.event.pull_request.*`; diff na runnerze. Pułapka: domyślny checkout jest shallow — bez `fetch-depth: 0` diff będzie pusty.

```yaml
- uses: actions/checkout@v4
  with:
    fetch-depth: 0

- id: diff
  run: echo "value=$(git diff origin/${{ github.base_ref }}...HEAD)" >> "$GITHUB_OUTPUT"

- uses: twoj-zespol/ai-reviewer@<sha>
  with:
    api-key:  ${{ secrets.LLM_PROVIDER_API_KEY }}
    pr-title: ${{ github.event.pull_request.title }}
    pr-body:  ${{ github.event.pull_request.body }}
    diff:     ${{ steps.diff.outputs.value }}
```

```text
/10x-research ci-cd-code-review based on requirements from '@context/changes/ci-cd-code-review/requirements.md'
/10x-plan ci-cd-code-review Plan the implementation of ci/cd workflow based on provided research and @context/changes/ci-cd-code-review/requirements.md
```

### Claude w GHA - Claude Action
Ręczny agent (SDK + composite action) = pełna kontrola nad promptem, schematem, bramką. **Claude Code Action** (`anthropics/claude-code-action@v1`) — gotowy harness z dostępem do repo, komentarzy, review; sterowanie przez `prompt`. Intelligent mode detection: `@claude` w komentarzu vs automatyczny `pull_request`.

Tryb interaktywny:

```yaml
on:
  issue_comment:
    types: [created]
  pull_request_review_comment:
    types: [created]
steps:
  - uses: anthropics/claude-code-action@v1
    with:
      anthropic_api_key: ${{ secrets.ANTHROPIC_API_KEY }}
      github_token: ${{ secrets.GITHUB_TOKEN }}
```

Automatyczne review:

```yaml
on:
  pull_request:
    types: [opened, synchronize]
steps:
  - uses: anthropics/claude-code-action@v1
    with:
      prompt: |
        Zrób review tego PR-a pod kątem poprawności, bezpieczeństwa i czytelności.
        Dla każdego zmienionego pliku podaj konkretne sugestie.
      anthropic_api_key: ${{ secrets.ANTHROPIC_API_KEY }}
      github_token: ${{ secrets.GITHUB_TOKEN }}
```

W `prompt` wklejasz kryteria DoD albo skill `10x-impl-review-ci` (Deep Dive). Zysk: czas, czytanie repo, skilli z `.claude/skills`, runner własny, modele przez Bedrock/Vertex. Ręczny agent gdy: wymuszony schemat werdyktu, twarda bramka na `score`, niestandardowe narzędzia, tani model spoza Anthropic. Zasada: zacznij od gotowca, schodź do SDK tam, gdzie gotowiec ogranicza.

### Testowanie zmian w Agentach z promptfoo
Antywzorzec **„chyba lepiej" po jednym PR-ze** — to wróżenie, nie testowanie. **promptfoo** porównuje prompty/agenty na zestawie prerenderowanych przypadków.

Trzy elementy w `promptfooconfig.yaml`: **prompts**, **providers** (modele obok siebie), **tests** (wejścia + **assert**). OpenRouter: jeden klucz, prefiks `openrouter:vendor/model`.

```yaml
providers:
  - id: openrouter:anthropic/claude-haiku-4.5
  - id: openrouter:anthropic/claude-sonnet-4.6
  - id: openrouter:openai/gpt-5-mini

prompts:
  - file://prompts/review.txt

tests:
  - vars:
      diff: file://fixtures/sql-injection.diff
    assert:
      - type: is-json
      - type: llm-rubric
        value: Werdykt odrzuca zmianę i wskazuje podatność SQL injection
      - type: javascript
        value: JSON.parse(output).score <= 3
```

```bash
npx promptfoo eval
```

Asercje: `is-json` (struktura pod bramkę), `llm-rubric` (drugi model ocenia treść), `javascript`/`python` (twarde progi). Wynik: macierz pass/fail + koszt + czas. Cicha regresja promptu wyłapie się na zestawie, nie na jednym PR. Domyślnie jeden fail = czerwono; poluzowanie: `PROMPTFOO_PASS_RATE_THRESHOLD=95`.

```text
/10x-research code-review-evals Analyze the current state of '@packages/code-reviewer' in the context of potential eval introduction...
/10x-plan code-review-evals Plan how to introduce promptfoo within '@packages/code-reviewer'...
```

Research: jeśli promptfoo nie pasuje — szukaj alternatywy OSS. Plan: ten sam prompt na 3 modelach, złożony diff (React 16→19+ z 3 błędami), LLM-as-judge + statyczny test fail.

### Więcej możliwości dla agenta
MVP agenta = jeden strzał: prompt + diff → `Output.object` (scorer). `ToolLoopAgent` bez `tools` to ceremonia — ale świadoma, bo **ToolLoopAgent** umożliwia pętlę narzędziową bez przepisywania architektury.

Narzędzie w AI SDK: **`description`** (kiedy wołać), **`inputSchema`** (Zod, walidacja dwukierunkowa), **`execute`** (robotę). Rejestracja: pole `tools`.

```ts
const readPlan = tool({
  description:
    'Read an implementation plan from context/changes/<change-id>/plan.md. ' +
    'Returns the plan contents, or { found: false } when none exists.',
  inputSchema: z.object({
    target: z.string().describe("A change-id (e.g. 'oauth-login') or a plan.md path."),
  }),
  execute: async ({ target }) => {
    const contents = await readPlanFromContext(target);
    return contents ?? { found: false };
  },
});

const agent = new ToolLoopAgent({
  model: provider(modelId),
  instructions: REVIEW_RUNNER_INSTRUCTIONS,
  tools: { readPlan },
});
```

Pętla: model → odpowiedź albo tool call → walidacja → `execute` → wynik do kontekstu → kolejna tura. **Koszt:** każdy krok = tokeny; domyślnie `stopWhen: stepCountIs(20)`. Antywzorzec **`isLoopFinished()` bez limitu** na CI — koszt × liczba PR. `onStepFinish` mierzy tokeny krok po kroku.

Drabina sprawczości: **read** lokalny (`./context`, DoD) → **write** GitHub (komentarz, etykiety) → całe repo (issues, checks, historia) → **read/write** poza GitHub (Linear, Slack). Agent staje się aktorem procesu, nie funkcją tekst→werdykt.

### Deep dive: Review implementacji na CI/CD - `10x-impl-review-ci`
> **Deep dive** — skill `10x-impl-review-ci` oddziela mechanikę uruchomienia od kryteriów oceny; reużywalny w Claude Action i w agencie AI SDK.

Skill porównuje PR z planem, ocenia w siedmiu wymiarach, werdykt: `APPROVED` / `NEEDS ATTENTION` / `REJECTED`. Podział plików:
- **`SKILL.md`** — hydraulika: plan, diff, podagenci, raport, commit `[skip ci]`, status check blokujący merge.
- **`references/impl-review-instructions.md`** — rubryka: plan jako źródło prawdy, trzy analizy (dryf, bezpieczeństwo/jakość, testy), gramatyka findingów. Progresywne ujawnianie — kryteria nie w `SKILL.md`.

**Ścieżka 1 (Claude Action):** skopiuj skill do `$HOME/.claude/skills`, wywołaj `/10x-impl-review-ci` w `prompt`.

**Ścieżka 2 (AI SDK):** wyciągnij kryteria narzędziem `readReviewCriteria` z `impl-review-instructions.md`; mechanikę piszesz sam, schemat odpowiedzi wymuszasz `Output.object`. Kryteria przenośne między harnessami; hydraulika wymienna.

### Deep dive: Pod maską skilla
> **Deep dive** — trzy narzędzia w `ToolLoopAgent`: `readPlan`, `readImplReviewCriteria`, `postPrComment` (wtedy usuń komentowanie z workflow YAML).

Prompt dwuetapowy:
1. **STEP 1 — zawsze:** ocena diff wg sześciu kryteriów + `postPrComment({ kind: "code" })` — także przy czystym pass.
2. **STEP 2 — warunkowo:** plan review tylko gdy PR wskazuje plan (`context/changes/<id>/plan.md`, marker `Plan: <id>`, change-id).

Ścieżki: tylko Code Review vs Code Review + Plan review. Plan review = wyższy koszt + niedeterminizm. Przed wdrożeniem rozważ: jakie PR-y trafiają do agenta, co filtrować z diffa, czy plan review tylko po zielonym code review, jeden vs dwa modele.

### Deep dive: Eksperymentalne wsparcie dla skilli po stronie Vercela
> **Deep dive** — `experimental_createSkillTool` w paczce `bash-tool` ładuje cały folder skilli do sandboxa, nie tylko kryteria.

```ts
const { skill, files, instructions } = await createSkillTool({
  skillsDirectory: './skills',
});
const { tools } = await createBashTool({ files, extraInstructions: instructions });
const agent = new ToolLoopAgent({ model, tools: { skill, ...tools } });
```

Różnica vs ścieżka 2: cały skill w sandboxie z bashem (bliżej Claude harness) vs wyciągnięcie prozy kryteriów. Zastrzeżenia: `experimental_` = niestabilne API; wymaga sandboxa; skill trudniej przenosić między runtime'ami. Dla CI review wystarcza wyciągnięcie kryteriów.

### Deep dive: Code review w skali zespołu
> **Deep dive** — po miesiącach w produkcji liczy się proces, nie tylko kod agenta.

AI w review = **przepustowość**, nie zastępstwo recenzenta. Cztery wątki:
- **Triage** — poziom ważności PR (pliki, rozmiar diffa, etykiety) decyduje, kiedy i jak głęboko odpalać AI.
- **Kto ma ostatnie słowo** — niskie ryzyko + zielony werdykt → auto-merge; krytyczne ścieżki → człowiek + opinia agenta.
- **Koszt i czas jako metryka** — `onStepFinish` + dashboard (Grafana); koszt per PR i per poziom triage.
- **Pętla uczenia** — gdzie człowiek był zbędny vs gdzie agent przepuścił bug → przypadek do promptfoo, korekta DoD, progi triage.

DoD i eval promptfoo to żywe artefakty — wiedza z produkcji wraca do systemu.
