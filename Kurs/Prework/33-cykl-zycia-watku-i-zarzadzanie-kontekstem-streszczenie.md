### Czym jest context engineering
Gdy agent po długiej sesji „gubi pamięć" mimo dobrych promptów — prompt i model się nie zmieniły; zmieniła się zawartość okna kontekstowego. Brakuje trzeciego fundamentu preworku: świadomego zarządzania kontekstem w czasie (kiedy czyścić, kompresować, izolować, zaczynać od nowa). `Context engineering` (Karpathy, 2025) to wypełnianie okna dokładnie tym, co potrzebne do następnego kroku; Harrison Chase (LangChain) dodaje: właściwe informacje i narzędzia we właściwym formacie. Różnica względem `prompt engineeringu` nie jest kosmetyczna — prompt to redagowanie instrukcji, context engineering to **projektowanie systemu**, który decyduje, co, kiedy i w jakiej formie trafia do okna; w agencie po dziesiątkach tur sam prompt to punkt startowy, a wynik zależy od całej sesji.

LangChain proponuje cztery strategie jako mapę decyzji (każda tura to świadoma lub nieświadoma decyzja): **Write** — zapis poza kontekstem (notatki, plany markdown, commity cząstkowych wyników, pamięć po restarcie); **Select** — wybór istotnego na następny krok zamiast 20 plików „na wszelki wypadek" (grep, semantic search, odczyt na żądanie, `@`-mentions); **Compress** — redukcja objętości z zachowaniem sensu (kompakcja historii, streszczenia, przycinanie starszych wiadomości); **Isolate** — separacja odpowiedzialności (subagent eksploruje we własnym oknie, do głównej rozmowy wraca podsumowanie).

### Jak żyje wątek z agentem
Cykl życia wątku: (1) świeży wątek — reguły projektu, cel, pierwszy kontekst; (2) eksploracja — pliki, logi, dokumentacja; (3) plan — zakres, ograniczenia, weryfikacja; (4) implementacja — modyfikacje lub diff; (5) weryfikacja — testy, build, lint, scenariusz ręczny; (6) kompakcja lub przekazanie — zapis decyzji, redukcja historii; (7) zamknięcie lub restart. Każdy wątek zaczyna od świeżego okna (`AGENTS.md`, `Cursor Rules`, pierwsza wiadomość) — moment najwyższej jakości; z każdą turą dochodzą odpowiedzi, wyniki tool calli, odczyty i błędy, a przypadkowa historia utrudnia skupienie na bieżącej decyzji.

Kluczowa umiejętność to rozpoznanie **sygnałów degradacji** (nie magiczna liczba tokenów): agent powtarza ustalenia i „odkrywa" to, co sam znalazł kilka tur wcześniej; wymyśla ścieżki, komendy lub pliki niepasujące do repo; ignoruje zaakceptowane ograniczenia (np. „nie ruszaj publicznego API"); łata wokół testu zamiast wracać do przyczyny; podsumowania rosną, ale decyzji jest mniej; zaprzecza wcześniejszym ustaleniom. Przy dwóch lub więcej sygnałach — decyzja, nie kolejny prompt; trzeci „skup się bardziej" rzadko ratuje sytuację.

| Sytuacja                                    | Narzędzie                                           | Dlaczego                                                                |
| ------------------------------------------- | --------------------------------------------------- | ----------------------------------------------------------------------- |
| Sesja się rozrosła, ale kierunek jest dobry | Kompakcja, np. /compact <co zachować>               | Redukcja historii z zachowaniem decyzji, ścieżek i otwartych ryzyk      |
| Zmiana tematu lub koniec zadania            | Czyszczenie, np. /clear                             | Reset rozmowy przy zachowaniu reguł projektu ładowanych przez narzędzie |
| Ostatni krok był zły                        | Cofnięcie ostatnich tur, jeśli narzędzie to wspiera | Powrót przed błędną decyzję bez resetu całej sesji                      |
| Sesja jest nie do uratowania                | Nowy wątek                                          | Commitnij wiedzę do pliku, zacznij od zera z lepszym promptem           |

`/clear` odzyskuje najwięcej miejsca, ale tracisz szczegóły bieżącej rozmowy — gdy sesja służyła tylko prototypowaniu albo osiągnąłeś zadanie i przechodzisz do nowego tematu. `/compact` to „miękki reset" z określeniem, co zachować; stara historia może być niewidoczna lub mocno streszczona — nie traktuj kompakcji jak pełnego archiwum. Reguła Anthropic: jeśli poprawiałeś agenta więcej niż dwa razy w tej samej sprawie, często lepiej `/clear` i nowy prompt niż brnięcie w zaśmieconym kontekście. Lee Robinson (Cursor): nowy wątek przy zmianie zadania, powtarzających się błędach lub zakończeniu logicznej jednostki pracy.

### Co zapisać przed resetem
Największy błąd przy `/clear` lub nowym wątku: usuwanie historii, którą właśnie wypracowałeś. Przed resetem zapisz w pliku, commicie lub podsumowaniu: aktualny cel; zaakceptowane decyzje techniczne; zmienione lub do zmiany pliki; uruchomione komendy i wyniki; status testów, buildu lub weryfikacji ręcznej; otwarte ryzyka i rzeczy, których agent nie powinien ruszać; następny prompt do wznowienia. To strategia `Write` w najprostszej wersji — jeśli coś ma przetrwać reset, zapisz poza rozmową.

Przy kompakcji użyj szablonu:

```text
Zachowaj: finalne decyzje, ścieżki plików, wykonane komendy, status testów, zaakceptowane ograniczenia i nierozwiązane ryzyka.
Usuń: nieudane próby, duplikaty logów, pośrednie hipotezy, powtórzone wyjaśnienia i szczegóły, które nie wpływają na następny krok.
Następny krok: <jedno konkretne działanie, od którego agent ma zacząć po kompakcji>.

```

### Izolacja i pamięć zewnętrzna
`Compact` i `clear` działają w jednym wątku; czasem sam wątek nie jest właściwym miejscem na operację. **Subagenty** to wzorzec izolacji kontekstu: pracują we własnym oknie (odczyty, wyszukiwania, analiza zostają u nich), do głównej rozmowy wraca tylko podsumowanie — zamiast zaśmiecać główny kontekst eksploracją codebase'u. Tryb foreground blokuje głównego agenta do wyniku (gdy potrzebujesz odpowiedzi przed kolejnym krokiem); background to praca równoległa (długa eksploracja, testy, logi w tle). Role definiujesz plikami markdown z frontmatterem YAML (nazwa, opis roli, model, instrukcje) — klucz delegacji to opis, kiedy dana specjalizacja ma sens. Powtarzalne wzorce: weryfikator, orkiestrator (planista + implementer + tester), debugger. Ograniczenie: każdy subagent zużywa tokeny niezależnie — pięć równoległych to ~pięciokrotny koszt; deleguj celowo, nie „na wszelki wypadek".

**Pamięć zewnętrzna** przeżywa restart: `CLAUDE.md` wstrzykiwany w każdą rozmowę, `Cursor Rules` ładowane przez glob patterns, plany markdown, commity, scratchpady — wszystko to `Write`. Anthropic zaleca w połowie długiej sesji: commit postępów, `/clear`, wznowienie z czystym kontekstem i wiedzą w repo. Reguły na jutro: jeden wątek = jedna logiczna jednostka pracy (zmiana tematu → nowy wątek); przy powtarzaniu ustaleń, wymyślaniu ścieżek lub ignorowaniu ograniczeń — zatrzymaj pracę i wybierz `compact`, `clear`, cofnięcie lub nowy wątek; przed resetem zapisz cel, decyzje, pliki, status weryfikacji i następny prompt poza rozmową.
