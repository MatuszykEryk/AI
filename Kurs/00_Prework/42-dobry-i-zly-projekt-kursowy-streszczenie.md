### Dobry projekt jest mniejszy, niż myślisz
Projekt kursowy w 10xDevs to środowisko do pracy z agentem, artefaktami, testami, CI/CD i decyzjami technicznymi — nie startup, produkt ani case study portfolio. Nie musi być duży, webowy ani w „naszym” stacku; musi dać się dowieźć razem z kursem. **MVP (Minimum Viable Product)** to najmniejszy spójny przepływ dający użytkownikowi konkretną wartość — nie prototyp do wyrzucenia, nie połowa aplikacji. Fiszki: wklej tekst → generuj propozycje → akceptuj i zapisuj (bez organizacji, płatności, PDF, mobile, algorytmu powtórek). Treningi: cel i ograniczenia → plan tygodnia → oznaczanie sesji → aktualizacja rekomendacji. Reguła: **jeśli pierwszy działający przepływ wymaga więcej niż tygodnia pracy po godzinach, zmniejsz MVP** — tydzień to czujnik dymu, nie formalny limit; przy miesiącach dojścia do „czegoś działającego” zabraknie czasu na dokumentację, testy, CI/CD i feedback. Mały rdzeń pomaga agentowi: jasny cel, krótka pętla weryfikacji, łatwiejszy opis w **PRD (Product Requirements Document)**, rozbicie na taski i test end-to-end.

### Co musi mieć projekt zaliczeniowy
Certyfikacja ocenia proces z kursu i minimum techniczne, nie wielkość aplikacji. Sześć elementów: (1) **mechanizm kontroli dostępu** dopasowany do typu — logowanie w webie, lokalny profil lub klucz w desktopie; aplikacja rozróżnia użytkownika, stan lub uprawnienia, nie jest anonimową demonstracją. (2) **zarządzanie danymi** — tworzenie, odczyt, aktualizacja i usuwanie sensowne dla domeny (fiszki, zadania, treningi…), nie sztuczny CRUD „pod checkbox”. (3) **logika biznesowa** — decyzja domenowa (klasyfikuje, rekomenduje, waliduje, generuje propozycję…); jeśli nie opiszesz jej jednym zdaniem, projekt jest za mglisty. (4) artefakty z modułów 1–3: PRD, specyfikacje, plany, kontekst dla AI. (5) co najmniej jeden test kluczowego przepływu z perspektywy użytkownika. (6) **CI/CD (Continuous Integration / Continuous Delivery albo Deployment)** — pipeline budujący aplikację i uruchamiający testy; publiczny deployment, App Store lub pakiet instalowalny to warstwa ponad minimum. Akceptowane: własna aplikacja lub rozbudowa istniejącego projektu o moduł; stack otwarty (web, desktop, mobile, embedded…). Kluczowe pytanie: czy projekt pozwoli przećwiczyć workflow z AI i spełnić wymagania bez topienia czasu na rozruch?

| Moment           | Data             | Informacja zwrotna  |
| ---------------- | ---------------- | ------------------- |
| Koniec szkolenia | 19 czerwca 2026  | nie dotyczy         |
| 1\. termin       | 5 lipca 2026     | do 19 lipca 2026    |
| 2\. termin       | 10 sierpnia 2026 | do 25 sierpnia 2026 |
| 3\. termin       | 14 września 2026 | do 30 września 2026 |

Ostateczny wybór projektu i stacku — w pierwszym tygodniu kursu; prework przygotowuje do decyzji, nie wymusza deklaracji.

### Zły projekt ma wysoki próg zero-to-one
Najgroźniejsze pomysły często brzmią świetnie (automatyzacja firmy, pięć integracji, wieloplatformowy mobile, niszowy framework). **Zero-to-one** wysokie, gdy długo grzebiesz w technice — zewnętrzna autoryzacja, biblioteka słabo znana agentowi, integracja API przed podstawowym przepływem — i po tygodniu nadal nie ma jednej sensownej akcji; w kursie po godzinach to zabija momentum. Drugi błąd: za duże MVP (np. finanse osobiste z importem banku, skanem paragonów, wykresami, budżetami, alertami, rolami rodzinnymi i coachem AI naraz). Trzeci: **pusty CRUD** — rekordy bez decyzji domenowej; lista książek/filmów/zadań wymaga reguły (rekomendacja, plan powtórki, priorytetyzacja, scoring, workflow akceptacji). Antywzorzec: dobry projekt szybko daje efekty; zły to firma, nie side-project.

### Dobry vs zły projekt
| Kryterium        | Dobry projekt                                    | Zły projekt                                     | Pytanie kontrolne                                   |
| ---------------- | ------------------------------------------------ | ----------------------------------------------- | --------------------------------------------------- |
| Użytkownik       | Wiesz, kto używa aplikacji i po co               | „Dla wszystkich”                                | Kto jest adresatem pierwszej akcji?                 |
| Problem          | Jeden konkretny ból lub potrzeba                 | Ogólna platforma do wszystkiego                 | Jaki konkretny problem rozwiązuje aplikacja?        |
| MVP              | 1-2 kluczowe przepływy                           | Lista funkcji jak roadmapa produktu             | Co powinno zadziałać w pierwszym tygodniu?          |
| Dane             | Dane wynikają z domeny                           | Sztuczny CRUD doklejony do wymagań              | Co użytkownik tworzy lub aktualizuje?               |
| Logika biznesowa | Aplikacja podejmuje decyzję domenową             | Rekordy tylko leżą w bazie                      | Jaką regułę działania da się opisać jednym zdaniem? |
| Stack            | Znany, oparty o konwencje, dobrze udokumentowany | Niszowy albo wybrany tylko z ciekawości         | Czy agent i ty macie dość kontekstu?                |
| Test             | Da się przetestować główny przepływ              | Test nie ma czego sensownie sprawdzić           | Co musi przejść, żeby projekt uznać za sprawny?     |
| CI/CD            | Build i testy da się uruchomić automatycznie     | Projekt wymaga postawienia całej infrastruktury | Czy repo może samo sprawdzić podstawową jakość?     |

Przy dwóch pomysłach wybierz krótszą drogę do pierwszego przepływu i bardziej oczywistą logikę biznesową — nie ten, który brzmi efektowniej.

### Jak ocenić swój pomysł
Zanim rozmawiasz z agentem o implementacji, opisz projekt pytaniami kontrolnymi z tabeli. Niepełne pole to sygnał, że jeszcze nie pora kodować — w modułach 1 i 2 dopracujesz PRD i kontekst dla agenta. Inspiracja z Demo Day absolwentów: szukaj czytelnego problemu, działającego przepływu i historii „użytkownik robi X, aplikacja pomaga osiągnąć Y”, nie wielkości projektu. Reguła decyzji: pierwszy przepływ w ~tydzień po godzinach — inaczej zmniejsz MVP przed pisaniem PRD. Test jednego zdania: jeśli brzmi tylko „użytkownik dodaje i usuwa rekordy”, wzmocnij domenę. Akcja: wypełnij tabelę oceny, potem decyzja o stacku — stack obsługuje projekt, nie zastępuje decyzji o projekcie.
