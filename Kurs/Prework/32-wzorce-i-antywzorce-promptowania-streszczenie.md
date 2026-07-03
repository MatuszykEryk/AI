### Dwa tryby pracy z AI
Chatbot: jedna tura, brak efektów ubocznych. Agent: autonomiczna pętla przez dziesiątki iteracji — ten sam tekst to kontrakt wysokiej wagi, nie kwestia „lepszego modelu”. Antywzorzec: „napraw ten błąd” bez granic. Trzy różnice: **czas trwania** (dwuznaczność z jednej tury rośnie w dziesiątej iteracji), **efekty uboczne** (edycja plików, testy, repo — określ co robić i czego unikać: pliki, publiczne API, migracje, budżet wydajności; bez granic agent improwizuje), **kaskada błędów** (agent w pętli naprawia nieistotne względem oryginalnego celu). W dobrym środowisku prompt bywa krótki — instrukcje żyją w warstwach konfigurowanych raz.

### Anatomia prompta agentowego
W chacie prompt to zwykle wszystko, co model widzi; w agencie wiadomość użytkownika to ostatnia warstwa hierarchicznego stosu instrukcji. Od dołu: **system prompt harnessu** (bezpieczeństwo, kompresja kontekstu, narzędzia — producent, nie ty), **reguły projektowe** (`CLAUDE.md`, `AGENTS.md` — konwencje, ograniczenia, architektura, raz na projekt), **pamięć** (preferencje i decyzje z sesji, często wyłączona — ważne decyzje i tak do plików projektu), **skille** (`/commit`, `/code-review`, `/plan` — standaryzacja powtarzalnych zadań), **twój prompt** (cel tu i teraz).

| Warstwa           | Kto konfiguruje             | Jak często                            | Przykład                                     |
| ----------------- | --------------------------- | ------------------------------------- | -------------------------------------------- |
| System prompt     | Producent narzędzia         | Nigdy (z twojej strony)               | Bezpieczeństwo, kompresja kontekstu          |
| Reguły projektowe | Ty lub zespół               | Od czasu do czasu — zmiany ewolucyjne | CLAUDE.md: "testy Vitest, Svelte, nie React" |
| Pamięć            | Narzędzie albo konfiguracja | Zależnie od środowiska                | "User stosuje conventional commits"          |
| Skille            | Ty lub zespół               | Reużywalnie, wg potrzeby              | /commit, /code-review                        |
| Prompt            | Ty, tu i teraz              | Każde zadanie                         | "Napraw bug w calculateDiscount()"           |

Najpierw konfigurujesz warstwy, potem cel i kryteria sukcesu — w chacie tłumaczysz _jak_ odpowiedzieć, w agencie _po czym_ system pozna ukończenie (build, testy, linter). Reguły w `AGENTS.md` / `CLAUDE.md`, procedury w skillach.

### Modele rozumujące w praktyce
Modele rozumujące rozkładają problemy wewnętrznie — kroki, przykłady rozwiązań i narzucona technika często dodają szum. Optymalnie: cel + żelazne ograniczenia; chodzi o cel, nie mikrozarządzanie.

**1. Nie narzucaj kroków, jeśli nie musisz.**
> ❌ _„Przeanalizuj tę funkcję krok po kroku. Najpierw sprawdź typy argumentów, potem prześledź przepływ danych, następnie zidentyfikuj błąd i zaproponuj poprawkę."_
> ✅ _„Napraw błąd powodujący podwójne naliczanie rabatu w calculateDiscount(). Testy w discount.test.ts muszą przechodzić."_

Wyjątki: migracje danych, bezpieczeństwo, płatności, dostępność, twarde konwencje zespołu — wtedy kroki pilnują procesu, którego nie wolno pominąć.

**2. Nie dawaj przykładów, które zamykają problem.**
> ❌ _„Oto jak zoptymalizowałem podobną funkcję: [15 linii kodu]. Zoptymalizuj renderowanie w userList.ts w analogiczny sposób."_
> ✅ _„Zoptymalizuj renderowanie listy użytkowników w userList.ts — przy 5000 rekordów czas renderowania nie powinien przekraczać 200ms."_

Przykłady OK dla stylu, formatu raportu lub DSL — nie jako najlepsze rozwiązanie techniczne.

**3. Nie narzucaj techniki — podaj ograniczenia.**
> ❌ _„Użyj wzorca Observer do synchronizacji stanu między komponentami. Stwórz klasę EventBus z interfejsem Subscriber."_
> ✅ _„Komponenty Dashboard i Sidebar muszą reagować na zmianę filtrów w czasie rzeczywistym. Aktualnie stan jest niespójny po szybkim przełączaniu. Napraw synchronizację — bez zmiany publicznego API komponentów."_

Po 1–2 nieudanych próbach — bądź konkretniejszy albo poproś o trzy podejścia, potem wybierz jedno.

### Popularne antywzorce promptowania
**1. Brak granic.** Prosta zmiana IF → refaktor, aktualizacja zależności, zmiana konwencji. Brakuje: „edytuj tylko `src/billing/*`”, „nie zmieniaj publicznego API”, „nie dodawaj zależności”, „najpierw pokaż plan”, „bez mojej zgody nie uruchamiaj migracji”.
**2. Przeładowanie kontekstu na start.** Cały kod w pierwszym prompcie gubi model — wąski zakres albo swoboda eksploracji.
**3. Brak obiektywnych kryteriów.** „Upewnij się, że działa” bez komend w terminalu; potrzebny linter, build, testy. Test od agenta może przykryć ten sam błąd.
**4. Kompresja hierarchii.** Wszystko w `CLAUDE.md` — procedury do skilli, preferencje do pamięci, w pliku tylko fundamentalne reguły.
**5. Ufanie poprawnej składni.** Kompilacja ≠ poprawna semantyka; poprawna składnia to start, nie koniec — myśl jak inżynier.

### Kontrakt prompta
Dobry prompt agentowy odpowiada na pytania (część pomijasz, gdy masz `AGENTS.md` / `CLAUDE.md`): **cel** — co ma być prawdą po zakończeniu; **zakres** — które pliki, moduły, ścieżki; **granice** — czego nie zmieniać bez zgody; **kontekst** — skąd prawda: issue, logi, test, dokumentacja, fragment kodu; **weryfikacja** — komenda lub scenariusz sprawdzenia; **raport** — diff summary, ryzyka, testy, decyzje; **uprawnienia** — edycja vs tylko eksploracja i plan.

### Gotowe szablony
Wzorzec: najpierw eksploruj, potem edytuj.

**Bugfix**
```text
Naprawiamy krytyczny incydent. Oto zgłoszenie z supportu: 
<TICKET>
[krótki opis objawu].
</TICKET>

Prawdopodobny zakres zmian dotyczy @{module/directory/file}
Najpierw znajdź przyczynę i wskaż minimalny plan, potem wprowadź poprawkę.
Nie zmieniaj publicznego API bez mojej zgody.

Weryfikacja: uruchom [komenda testu/builda] i opisz wynik.
Na końcu podsumuj zmienione pliki i ryzyka.
```

**Refaktor**
```text
Zrefaktoryzuj [obszar], zachowując obecne zachowanie.

Cel: [np. uproszczenie duplikacji / rozdzielenie odpowiedzialności].
Zakres: [pliki/moduły].
Nie zmieniaj kontraktu publicznych funkcji ani formatu danych.
Jeśli zauważysz potrzebę zmiany publicznego API, zatrzymaj się i poinformuj mnie.

Weryfikacja: [komenda] oraz krótki opis, jak potwierdziłeś brak zmiany zachowania.
```

**Code review**
```text
Przejrzyj zmiany pod kątem błędów, regresji i brakujących testów.

Oto reguły zespołowe: {rules}

Skup się na zachowaniu, bezpieczeństwie, edge case'ach i zgodności z konwencjami projektu.
Zwróć tylko konkretne uwagi z plikiem i linią.
Na końcu dodaj krótką listę testów, których brakuje albo których nie da się ocenić z diffu.
```

**Eksploracja bez edycji**
```text
Zbadaj, jak w tym projekcie działa [obszar/funkcja].

Znajdź najważniejsze miejsca w kodzie, zależności i przepływ danych między składowymi.
Na końcu zaproponuj 2-3 możliwe podejścia do rozwiązania [problem], z ryzykami.
Zatrzymaj się przed implementacją i nie edytuj plików.
```

**Implementacja po planie**
```text
Zaimplementuj zatwierdzony plan z [link/sekcja].

Jeśli plan okaże się nieaktualny, zatrzymaj się i wyjaśnij konflikt.
Weryfikacja: [komenda].
```

### Najważniejsze zasady
Kryteria sukcesu, nie ścieżka; warstwy zamiast jednego pliku; cel i granice zamiast mikrozarządzania modelami rozumującymi. Niejasne lub wieloplikowe zadanie → eksploracja i plan przed edycją. Prompt z prawem edycji: zakres, granice, komenda weryfikująca. Akcja: co przenieść do `AGENTS.md` / `CLAUDE.md`, a co podawać w prompcie.
