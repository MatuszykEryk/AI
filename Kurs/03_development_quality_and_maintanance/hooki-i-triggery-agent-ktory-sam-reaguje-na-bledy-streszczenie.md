### Wprowadzenie

Antywzorzec: ręczne lint/typecheck/test po każdej edycji agenta — kumuluje się w godziny. **Hooki** harnessu uruchamiają weryfikację lokalnie we właściwym momencie, bez czekania na CI.

### Cykl życia hooka

Każdy hook: **Trigger** (np. zapis pliku) → **Matcher** (narzędzie, typ pliku) → **Handler** (shell) → **Sygnał** (exit code, stdout do kontekstu). Ten sam wzorzec: Claude Code, Cursor, Codex, Windsurf, Copilot.

### Pierwszy hook w praktyce

```bash
npx @przeprogramowani/10x-cli@latest get m3l3
```

Paczka: reguła `CLAUDE-m3l3`. Przykład lint po `Write|Edit`:

```json
{
  "hooks": {
    "PostToolUse": [
      {
        "matcher": "Write|Edit",
        "hooks": [
          {
            "type": "command",
            "command": "npx eslint --fix . --quiet",
            "timeout": 10000
          }
        ]
      }
    ]
  }
}
```

Konfiguracja Claude Code: `~/.claude/settings.json`, `.claude/settings.json` (commit), `.claude/settings.local.json` (gitignore).

### Typecheck i pytanie o szybkość

Hooki **blokują** pętlę agenta. Heurystyka: check > kilka sekund → pre-commit/pre-push. Lint/format per-edit; typecheck per-edit w małych projektach; pełne testy → commit/push. Irytacja = sygnał zmiany warstwy.

### Testy tylko tam, gdzie trzeba

```bash
vitest related src/server/auth.ts --run
```

`related` = subkomenda, nie flaga; `--run` bez watch. Parsowanie ścieżki ze stdin: `jq -r .tool_input.file_path`. Nie każda edycja wymaga testów — `test-plan.md` wskazuje obszary ryzyka. PostToolUse = raz na użycie narzędzia (3 pliki → 3 uruchomienia). Vitest 4.1+: `AI_AGENT=1` — kompaktowe wyjście.

### Agent widzi i reaguje

Exit codes: **0** sukces, **2** błąd blokujący (agent widzi feedback), **inny** log bez blokady. Agent naprawi format/import/typ; przy padającej logice biznesowej — tylko sygnał „coś nie tak". Złożone problemy → `/10x-new` → research → plan → implement.

### Trzy warstwy lokalnej jakości

**Per-edit** — format, typy, related testy; jedyna warstwa z feedbackiem w trakcie pracy agenta. **Pre-commit** — staged files, ręczne edycje. **Pre-push** — cięższe checki. **CI** — integracja, infrastruktura. Lokalne warstwy nie zastępują CI.

### Bramka na poziomie commita

Przykład Lefthook (agnostyczny stack):

```yaml
pre-commit:
  parallel: true
  commands:
    lint:
      glob: "*.{ts,tsx,js,jsx}"
      run: npx eslint --fix {staged_files} && git add {staged_files}
    typecheck:
      run: npx tsc --noEmit
    test:
      glob: "*.{ts,tsx}"
      run: npx vitest related {staged_files} --run
```

Node: Husky+lint-staged; Python: pre-commit.com; uniwersalnie: Lefthook. `lefthook install` → automatyczny `pre-commit`.

### Ten sam wzorzec w każdym narzędziu

| Narzędzie | Context injection | Konfiguracja |
| --------- | ----------------- | ------------ |
| Claude Code, Cursor, Codex, Copilot (VS Code) | tak | różne pliki |
| Windsurf | **nie** — exit 2 bez feedbacku | `.windsurf/hooks.json` |

Codex: hooki z repo wymagają `/hooks` (hash trust). Copilot: ignoruje matchery, camelCase `tool_input.filePath`, inne nazwy narzędzi (`create_file` vs `Write`). 1Password `agent-hooks` — jeden skrypt, wiele narzędzi.

### Hooki a test-plan.md

Hooki zamieniają quality gates z `test-plan.md` w automatyczną weryfikację. Pytanie przy każdej bramce: per-edit, commit, pre-push czy CI? Przykład 10xCards: lint+typecheck pre-commit; unit+integration po fazie 1; e2e pre-push po fazie 6; post-edit hooki odroczone — **menu, nie nakaz**. Hooki nie łapią layoutu/nawigacji → E2E (M3L4).

### Deep dive: Performance hooków

> **Deep dive** — Per-edit: formatter <1s; related testy zależą od grafu. Pre-commit: `vitest --changed`. Claude Code: `async: true` dla hooków informacyjnych bez blokady.

### Deep dive: Inne typy handlerów

> **Deep dive** — Claude Code: `http`, `mcp_tool`, `prompt`, eksperymentalny `agent`. PreToolUse: `updatedInput`, `allow`/`deny`/`ask`. Cursor/Copilot: command+prompt; Codex/Windsurf: głównie command.

### Deep dive: Niezawodność hooków

> **Deep dive** — PostToolUse+command = najpewniejsza kombinacja. Pułapki: Codex pomija repo hooks bez `/hooks`; Copilot ignoruje matchery; Windsurf bez context injection. Hooki deterministyczne — przetrwają kompresję kontekstu lepiej niż reguła w `CLAUDE.md`.
