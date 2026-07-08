### Wprowadzenie

Antywzorzec po opanowaniu cyklu zmian: „odpalmy sześciu agentów" na resztę roadmapy. Przez pięć minut wygląda jak przyszłość — potem kilka sesji, branchy, PR-y i pytanie kto to zrozumie. **Nadal ty.** Więcej agentów bez zarządzania = więcej przypadkowego kodu na twoją odpowiedzialność. Lekcja: równoległa praca zorganizowana — osobny zakres, git worktree, osobny agent. Prework: worktrees i Isolate — tu praktyka.

### Zacznij od niezależnych slice'ów

Dwa slice'y dotykające tych samych obszarów → konflikt, nie przyspieszenie. Wybierz zadania bez wchodzenia sobie w drogę. 10xCards: **S-05** (usuwanie konta, RODO) i **S-06** (poprawki UX odkryte w trakcie — bulk actions, reset sesji, loading states) — osobne obszary, S-06 równolegle z S-05.

Prompt dodania slice'a do roadmapy:

```text
During work on S-01 through S-04 I noticed missing bulk actions on candidate review, no way to reset a review session, and poor loading states. Add a new slice S-06 "UX improvements" to the roadmap — status planned, prerequisites F-01, parallel with S-05. Keep it concise and consistent with existing slice format.
```

Weryfikacja przed implementacją:

```text
Check the roadmap and plans for S-05 and S-06. Assess whether they can be implemented in parallel. Pay attention to shared files, contracts, layers and other elements that may cause conflicts.
```

### Worktree to osobny pas ruchu

Git worktree — osobny katalog roboczy, ten sam repo; własny branch, `HEAD`, index.

```bash
git worktree add ../10xcards-account-deletion -b feature/account-deletion
git worktree add ../10xcards-ux-improvements -b feature/ux-improvements
```

Trzy foldery: główny (roadmapa, koordynacja), worktree S-05, worktree S-06. Strategia **Isolate**: katalog, branch, sesja, kontekst zadania. Granica: worktree nie izoluje portu dev servera, lokalnej bazy ani sandboxa płatności — to osobna konfiguracja.

### Dwa tryby: rozmowa albo delegowanie

Złożone zmiany z decyzjami w trakcie → interaktywna sesja, `/10x-implement`, pauzy, kontrola na bieżąco. Plan konkretny, zakres zamknięty, warunki mierzalne → delegowanie bez interakcji. **`/goal`** (Claude Code, Codex):

```text
/goal Use 10x-implement skill to implement all phases of context/changes/ux-improvements/plan.md. Each phase is committed separately. All phases marked done in plan progress. Stop after 20 turns if not complete.
```

Ewaluator po każdej turze sprawdza warunek zakończenia. Kontrola przesuwa się na PR, review i twoją decyzję — celowe obniżenie interakcji, nie odpowiedzialności.

### Wiele sesji w jednym widoku

Cursor Agents, Antigravity, Superset, Conductor, Claude Code Agent View — ten sam wzorzec: (1) izoluj kontekst per zadanie, (2) deleguj cel, (3) recenzuj w jednym miejscu. Wybór narzędzia osobisty; wzorzec się nie zmienia.
