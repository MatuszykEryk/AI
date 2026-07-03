### Co już masz na dysku

Po M1L2 masz `prd.md` i `tech-stack.md` — kontrakt biznesowy i techniczny, ale projekt jeszcze nie istnieje. Łańcuch `/10x-shape → /10x-prd → /10x-tech-stack-selector → /10x-bootstrapper` operuje na plikach w `/context`. W tej lekcji `tech-stack.md` trafia do `/10x-bootstrapper`, który tworzy szkielet projektu i raport `verification.md`. Istniejący projekt → `/10x-health-check` zamiast bootstrappera.

### Uruchomienie bootstrappera

```text
/10x-bootstrapper @tech-stack.md
```

Skill najpierw weryfikuje, czy istnieje `context/foundation/tech-stack.md`. Brak pliku → odmowa wykonania i przekierowanie do `/10x-tech-stack-selector` — nie wyciąga stacku z historii rozmowy, nie improwizuje. Hand-off jako warunek wstępny to pierwsza z trzech bramek: **pre-execution** — zanim agent cokolwiek wykona, sprawdza mandat (tu: plik wejściowy; w innych skillach: testy, brak niezacommitowanych zmian).

### Dlaczego agent nie pisze projektu sam

Bootstrapper czyta `starter_id` z `tech-stack.md`, wyszukuje starter w rejestrze i wykonuje `cmd_template` (np. `npm create astro@latest -- --template minimal`). Oficjalne CLI zna aktualne wersje, konwencje i integracje; agent z pamięci dowiózłby pół-działający boilerplate (np. `"astro": "^4.0.0"` przy Astro 6.x).

> **Jeśli istnieje CLI, które potrafi zrealizować zadanie, agent powinien do niego delegować, zamiast działać na podstawie danych treningowych.**

Ta sama logika: `docker init` przed ręcznym Dockerfile, `ng generate component` zamiast komponentu „od zera", `npx prisma migrate dev` zamiast SQL z głowy. Pewność siebie modelu jest najbardziej kosztowna tam, gdzie obok stoi narzędzie z autorytatywną odpowiedzią.

### Trzy bramki egzekucji

Każda egzekucja agenta przechodzi przez: (1) **Pre-execution** — mandat przed działaniem na dysku/sieci (hand-off `tech-stack.md`, recency check startera); (2) **In-execution** — interakcja agent–harness (Claude Code, Cursor, Codex); harness pyta lub wykonuje wg polityki; (3) **Post-execution** — audyt wyniku (w bootstrapperze: `npm audit` + `verification.md`). Właściciele: kontrakt skilla / harness i twoja polityka / narzędzia zewnętrzne. Trzy pytania do każdego skilla: co sprawdza przed startem? co harness chce w trakcie? co dostaniesz na wyjściu?

### Uprawnienia w trakcie pracy

Harness pyta o zgodę na operacje dotykające dysku lub sieci; bez pytania puszcza odczyt (`ls`, `cat`, `grep`, `git status`). Przy każdym prompcie: **Yes** (jednorazowo), **Yes, don't ask again** (reguła w `.claude/settings.json`), **No** (feedback, inna ścieżka). Antywzorzec: klikanie Yes wszędzie → wieczorne potwierdzanie rutyny; wszystko „don't ask again" → agent może wykonać dowolną komendę bash.

Filtr: **co ten wzorzec może popsuć?** „Nic" (np. `npm install`) → dodaj regułę; „Wszystko, co `git push` może wyrzucić na remote" → indywidualna zgoda; „Potencjalnie cały dysk" (np. `Bash(*)`) → nie zgadzaj się. Kolejność ewaluacji: **deny → ask → allow** — pierwsza pasująca wygrywa. Wbudowane narzędzia (`Read`, `Edit`, `Write`) i bash to różne ścieżki: `Read(./.env)` w deny nie blokuje `cat .env`. Polityka filtruje wywołania po nazwie i argumentach, nie po efekcie — **nie jest absolutna**. Analogicznie: Codex `--sandbox workspace-write --ask-for-approval on-request` w `~/.codex/config.toml`; Cursor allowlist w `~/.cursor/permissions.json`.

### Rekomendowana konfiguracja startowa

Po pierwszym bootstrapie wklej do `.claude/settings.json`:

```json
{
  "permissions": {
    "allow": [
      "Bash(npm *)",
      "Bash(npx *)",
      "Bash(node *)",
      "Bash(git add *)",
      "Bash(git commit *)",
      "Bash(git diff *)",
      "Bash(git log *)",
      "Bash(git status *)",
      "Bash(git branch *)",
      "Bash(git checkout *)",
      "Bash(git stash *)",
      "Read",
      "Edit",
      "Write"
    ],
    "ask": [
      "Bash(curl *)",
      "Bash(wget *)",
      "Bash(git push *)",
      "Bash(git push)"
    ],
    "deny": [
      "Bash(rm -rf *)"
    ]
  }
}
```

Allow: operacje lokalne w projekcie (`npm`/`npx`/`node`, git bez push, `Read`/`Edit`/`Write`). Ask: pobieranie z sieci (`curl`, `wget`), push (decyzja publikacyjna). Deny: `rm -rf` — bez prompta. Lista to punkt wyjścia: allow rośnie o wzorce przeszłe filtr; deny o zaskoczenia; reszta zostaje w ask.

### Polityka uprawnień i YOLO mode

Stan pośredni: rutyna bez pytania, destrukcja zablokowana, reszta wymaga zgody. Przeciwległy kraniec: **YOLO mode** (`bypassPermissions`, `--dangerously-skip-permissions`) — wszystko bez pytania. Anthropic: tylko w izolowanych środowiskach (kontenery, VM, dev containery bez dostępu do sieci hosta). Dev container: agent widzi tylko zamontowane repo; użytkownik nie-root, ograniczona sieć — chroni hosta, nie zwalnia z kontroli plików w repo.

**Polityka uprawnień jest kontrolą probabilistyczną, nie absolutną** — podnosi koszt pomyłki, nie zeruje go (udokumentowane obejścia denylisty, prośby o wyłączenie sandboxu). Zalecenie: **polityka jako punkt wyjścia, YOLO jako świadoma decyzja z warunkami brzegowymi, defense in depth**. Tryb **auto** (research preview) redukuje prompty bez pełnego wyłączenia kontroli.

### verification.md jako raport z bootstrappera

Po scaffoldzie: `context/changes/bootstrap-verification/verification.md` — post-execution, raport plikowy. Trzy fazy ze statusem `passed` / `warned` / `failed` / `skipped`: (1) pre-scaffold — czy starter nie jest porzucony; (2) scaffold and merge — exit code CLI (`failed` = HARD-STOP); (3) post-scaffold — `npm audit` (info/low/moderate/high/critical; niezerowy exit nie zatrzymuje bootstrappera). `warned` = wiesz, gdzie szukać; `failed` = interwencja. Bootstrapper v1 nie robi `git init`.

### Health-check: audyt istniejącego projektu

Brownfield → `/10x-health-check`, nie bootstrapper. Bramki: (1) **Pre-check** — audyt zależności (`npm audit`, `pip-audit`, `cargo audit`), lockfile, podatności; (2) **In-check** — read-only: test runner, CI/CD, brakująca konfiguracja (`tsconfig` strict, `.prettierrc`); (3) **Post-check** — werdykt `healthy` / `needs-attention` / `critical-issues`, fixy Category A (teraz: brak testów, krytyczne podatności) i B (CI w M1L5, AGENTS.md w M1L4). Czyta `stack-assessment.md` z `/10x-stack-assess`. Wynik: `context/foundation/health-check.md`. Obie ścieżki zbiegają się w M1L4.

### Deep dive: cztery tryby awaryjne

> **Deep dive** — Antywzorce pierwszego tygodnia: (1) **Approve-everything Stockholm** — ręczne Yes wszędzie → antidotum: kilka reguł allow + filtr „co może popsuć poza repo"; (2) **YOLO bez warunków** — `Bash(*)` w allow → antidotum: deny na niebezpieczne wzorce, brak prod-data, repo w gicie; (3) **AI tworzy projekt od zera** — pominięcie skilla → antidotum: deleguj do CLI; (4) **Brownfield-as-greenfield** — bootstrapper na istniejącym kodzie → antidotum: `/10x-health-check`.

### Deep dive: YOLO mode w wykonaniu 10xDevs

> **Deep dive** — Prowadzący używają `--dangerously-skip-permissions` w bezpiecznym środowisku (bez prod MCP, git, branchowanie); jeden incydent skasowanej lokalnej bazy — zatrzymany regułą w `CLAUDE.md`. Rekomendacja kursowa: stan pośredni, dopóki nie masz instynktu, kiedy warunki brzegowe przestają obowiązywać.

### Deep dive: Twój kod w chmurze

> **Deep dive** — Kod wysłany do API opuszcza maszynę. Trzy tiery: (1) darmowa aplikacja — dane domyślnie trenują model; (2) płatna subskrypcja — zależy od dostawcy (Google nie zmienia polityki treningu); (3) API/plany biznesowe — domyślnie nie trenują. Anthropic/OpenAI: wyłącz trening w ustawieniach na tierach konsumenckich; Codex ma osobne ustawienie. Chińscy dostawcy (DeepSeek, Qwen): dane w PRC, brak porównywalnego DPA — sprawdź politykę pracodawcy przed kodem firmowym. Checklist: tier, wyłączenie treningu, compliance (DPA, region).

### Deep dive: co jeśli delegacja zawodzi

> **Deep dive** — Niezerowy exit code CLI = **HARD-STOP**: skill się zatrzymuje, nie naprawia od strony agenta. Zostaje `.bootstrap-scaffold/`, częściowy `verification.md` ze statusem `failed`, nietknięty `tech-stack.md`. Napraw na poziomie systemu (sieć, npm, konflikt katalogu) i uruchom `/10x-bootstrapper` ponownie. **Kiedy autorytatywne CLI mówi nie, agent też mówi nie.**

### Deep dive: jak bootstrapper traktuje istniejące pliki

> **Deep dive** — Nie nadpisuje: istniejący `package.json` zostaje, wersja ze scaffoldu → `package.json.scaffold`. Katalog `context/` zachowywany verbatim (`prd.md`, `tech-stack.md`). `.gitignore` — merge lokalnych wpisów ze scaffoldu. Mechanizm in-execution: wbudowane reguły zamiast wymuszania promptem.
