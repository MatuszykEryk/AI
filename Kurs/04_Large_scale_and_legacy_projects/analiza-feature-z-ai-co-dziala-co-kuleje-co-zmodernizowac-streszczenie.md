### Mapa jako kontrakt wejściowy

Po Wide Scan (M4L2) masz `context/map/repo-map.md`. Mapa to **kontrakt wejściowy** dla agenta — nie notatka „do przeczytania". Stała struktura (prompt syntezy z L2):

- `TL;DR` — czym jest repo, gdzie skupia się praca,
- `Teren` — obszary dużej odpowiedzialności vs peryferia, głębokie/płytkie, aktywność w czasie,
- `Realne powiązania` — couplingi, warstwy, cykle,
- `Strefy ryzyka` — kilka obszarów wysokiego ryzyka z jednolinijkowym „dlaczego",
- `Kogo zapytać` — kandydaci per strefa,
- `Pierwszy dzień` — 5–8 plików wejściowych,
- `Ograniczenia` — okno czasowe, metoda, czego mapa NIE mówi.

W Deep Focus najważniejsze: `Strefy ryzyka` (stąd cel), `Pierwszy dzień` (entry pointy), `Ograniczenia` (pierwsze `unknowns` do potwierdzenia/obalenia).

Kolejność: **mapa → wybór celu → zakres → analiza semantyczna → potwierdzenie strukturalne** (ast-grep). W dużym legacy to różnica między pracą _z_ agentem a oglądaniem, jak agent pracuje po swojemu.

### Z mapy do jednego przepływu

Antywzorzec: „agent, przeanalizuj ten moduł” — kilka plików, importy, streszczenie, szybkie sugestie refaktoru bez pewności co do przepływu danych.

Mapa zamienia szerokie „przeanalizuj storage” w kierowaną pracę: gdzie zagęścić research, od którego pliku zacząć, czego spodziewać się po couplingu, jakie pytania otwarte. Zamiast „zrozum moduł” — zawężony cel z dowodami z mapy. Zasada: **wąsko, ale głęboko.**

### Czego mapa nie powie — i dlaczego sięgamy po agenta

Trzy wbudowane ograniczenia mapy: **szerokość, nie głębokość** (gdzie patrzeć, nie jak działa); **struktura, nie zachowanie** (statyczny obraz vs runtime, side-effecty, błędy); **migawka w czasie** (okno historii, białe plamy, `unknowns`). Wide Scan był tani i słuszny; Deep Focus wymaga świeżego wejścia w kod z rygorami `evidence` / `inference` / `unknown`. Agent bierze mapę jako **prior**, nie prawdę objawioną.

### Research bez wymyślania koła na nowo

`/10x-research` z **kontraktem zakresu** i mapą jako wejściem. Sztuczka: jedno zapytanie rozbija się na trzy sub-agenty:

- **trace e2e** — przepływ od entry pointu do zapisu/odczytu,
- **luki w testach** — pokrycie ścieżek i gałęzi,
- **blast radius** — co zmienia się razem (graf + co-change z gita).

Przygotowanie:

```text
/10x-init
/10x-new post-flow-analysis Analiza przepływu danych w obszarze wiadomości.
```

Prompt-wzorzec:

```text
/10x-research post-flow-analysis Przeanalizuj proces zapisu postów, zwracając szczególną uwagę na powiązane z nim obszary zdefiniowane w context/map/repo-map.md

Wykorzystaj trzech równoległych sub-agentów:
1. Trace e2e: odtwórz ścieżkę od entry pointu, przez warstwy, do zapisu/odczytu i z powrotem. Daj sekwencję kroków z file:line oraz diagram Mermaid.
2. Luki w testach: które metody i gałęzie na tej ścieżce mają pokrycie, a które nie.
3. Blast radius: co musi zmienić się razem przy zmianie tego przepływu — szew interfejsu, warstwy generowane, model, migracje, testy. Połącz graf statyczny z co-change z historii gita.

Skup się wyłącznie na analizie i opisie stanu obecnego repozytorium.
Twój raport musi zawierać dwie jawne sekcje: Feature overview, Technical debt.
Zapisz wnioski do context/changes/post-flow-analysis/research.md
```

Wynik: `context/changes/{change-id}/research.md` z sekcjami ② Feature overview i ③ Technical debt. Oddziel `evidence`, `inference`, `unknown`. Na górze pliku: jednolinijkowy cel (co badasz, od którego pliku, dlaczego mapa wskazała obszar). Paczka ćwiczeń: `npx @przeprogramowani/10x-cli@latest get m4l3`. **Krok 4 ćwiczenia: zatrzymaj się przed refaktorem** — wejście do M4L4.

### Feature overview

Cel: **przepływ, nie spis plików** — skąd dane, kto waliduje, gdzie zmienia się stan, co wraca; które kroki potwierdzone w kodzie (`evidence`), które założone (`inference`).

Mattermost — lekcja pokazuje typowe odkrycia trace: **struktura folderów kłamała** (gruby stos warstw vs realna praca w jednym gęstym miejscu); zapis „jednej wiadomości" to fasada nad operacją **wsadową**; dane po zapisie wracają z pamięci, nie z ponownego odczytu DB; **optymistyczny UI** — autor widzi wiadomość przed potwierdzeniem serwera, potem wersje się godzą w jednym punkcie. Tego nie widać z mapy ani drzewa plików — trzeba trace krok po kroku.

### Technical debt

Mapa **kruchości**, nie lista brzydkich plików. Typowe odkrycia: **cichy dług** — dwie połowy modelu (backend/front) bez narzędziowej synchronizacji (historia: rzadko jeden commit); zapis oparty na **kolejności** bez kompilatora; **ścieżki błędów** bez testów (decydują o bezpieczeństwie refaktoru).

Rozróżnij dług **prawdziwy** od **taniego** — sprzężenie wygląda groźnie, ale jest mechaniczne, generowane, łapane przez CI (klasyczny błąd: mylić jedno z drugim).

Agent na końcu raportu **jawnie koryguje** założenia z mapy, które okazały się nieaktualne — nie przepisuje ich po cichu jako faktów. To praktyka `evidence`/`inference`/`unknown`.

### Pogłębianie raportu z narzędziem ast-grep

Reguła: **im pewniej wygląda liczba w raporcie, tym bardziej warto ją sprawdzić.** Agent słaby na twierdzeniach strukturalnych („tylko tutaj”, liczba call-site'ów). Podział: agent = znaczenie i zachowanie; CLI = liczba i pozycja.

AST vs tekst: `grep "Save("` łapie komentarze i `SaveMultiple`; `ast-grep` rozumie węzły składniowe.

```bash
ast-grep -p '$X.Post().Save($$$A)' -l go
```

Instalacja: `npm i -g @ast-grep/cli`, brew, cargo, pip; na Linuksie pełna nazwa `ast-grep` (alias `sg` koliduje). Prompt śledztwa (nie refaktor):

```text
Ulepszamy raport context/changes/{change-id}/research.md.

Najpierw wypisz z niego wszystkie twierdzenia STRUKTURALNE (liczby call-site'ów, "tylko tutaj", "zawsze przez X", liczność metod, powtarzające się kształty wywołań).

Dla każdego zbuduj wzorzec narzędzia ast-grep, wywołaj je i przeanalizuj wyniki.

Wynik podaj jako: twierdzenie -> potwierdzone / doprecyzowane / obalone, z dokładnymi plikami i liniami. Każde zero z ast-grep potwierdź klasycznym grepem, żeby odróżnić realny brak wystąpień od złego wzorca.

Na koniec zaktualizuj i skoryguj raport wynikami z ast-grep.
```

Mechanizm: agent widzi kod jako tokeny, nie węzły AST — liczenia i „tylko tutaj" zostaw narzędziu strukturalnemu. Ta sama filozofia co L2: **tani deterministyczny CLI** jako kontrapunkt dla drogiego probabilistycznego agenta — inna oś (relacje/historia w L2, liczby/pozycje tutaj).

### ast-grep vs klasyczny grep — i czemu chcesz obu

`grep` — tępy, uczciwy (wszędzie, w tym komentarze). `ast-grep` — precyzyjny, wybredny (zły wzorzec = zero). **Reguła: ast-grep dla precyzji, każde zero potwierdź grepem** — zero ze strukturalnego matchera jest podejrzane.

### ast-grep potrafi nie tylko czytać, ale i przepisywać kod

Rewrite (na przyszłość, nie w tej lekcji):

```bash
ast-grep -p 'var $A = $B' -r 'let $A = $B' -l js src/
ast-grep -p 'var $A = $B' -r 'let $A = $B' -l js --interactive src/
```

Reguły YAML z `fix` → `ast-grep scan`. W M4L3 ast-grep = weryfikator; w M4L4 może nieść mechaniczną zmianę.

### Deep dive: Hotspot to złożoność × częstość zmian

> **Deep dive** — Ryzyko = skrzyżowanie **złożoności** i **częstości zmian**. Osobno wprowadzają w błąd: gęsty plik zamrożony vs trywialny plik w każdym commicie. Który proxy złożoności — mniej ważne niż samo skrzyżowanie. Mapa ma częstość z gita; Deep Focus dokłada złożoność na wybranym przepływie.

### Deep dive: Connascence — słownik kruchości

> **Deep dive** — „Tightly coupled” to werdykt bez opisu. Connascence nadaje nazwy: **pozycji/kolejności** (zapis DB bez kompilatora), **znaczenia/konwencji** (backend vs front), **nazwy/typu** (publiczny kontrakt). **Statyczna** — widoczna w kodzie; **dynamiczna** — ujawnia się w runtime (cichy dług). Siła rośnie z **dystansem**. Zamiast „ciasno” pisz np. „connascence znaczenia na dużym dystansie bez bariery narzędziowej”.

### Deep dive: Sprzężenie logiczne: para vs hub

> **Deep dive** — Blast radius łączy graf i współzmiany. Dwa pytania: **która para** jest ciasno związana (ukryty kontrakt) vs **który plik jest hubem** (szeroki promień). Sumowanie sprzężeń per plik wyciąga huby. Zastrzeżenia: współzmiany = dowód w oknie czasowym; masowy refaktor/formatowanie może spiąć niewinne pliki. Rankuj z **dwóch źródeł**; sprzeczności → weryfikacja, nie fakt.
