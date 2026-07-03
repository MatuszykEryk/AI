### Kontekst modułu piątego
Moduł 5 zmienia perspektywę z „jak poprawić mój projekt?" na „co w pracy zespołu regularnie traci czas, uwagę i energię?". Gdy AI przyspiesza pisanie kodu, wąskim gardłem stają się review, status projektu, rozproszone kryteria release'u, brak kontekstu do PR-a. Antywzorzec **„weekendowy zamiennik SaaS"**: kusząco szybko zbudować własny dashboard/system triage, ale łatwo przejąć niewidoczną odpowiedzialność platformy. Potrzebny filtr: kiedy użyć gotowej funkcji, kiedy cienki helper, kiedy poczekać na pełniejszy przepływ zespołowy.

### Tarcie zespołu jako sygnał
**Tarcie zespołu** = powtarzalny koszt koordynacji: ustalać, sprawdzać, przepisywać lub synchronizować tak często, że zespół traci tempo. Przykłady: status wymaga sześciu narzędzi; review stoi bez właściciela domeny; Linear, CI i notatka release'owa mówią różne rzeczy; sygnał z supportu nie trafia do backlogu w użytecznej formie. To sygnały, nie pomysły na produkt — powtarzalność ≠ automatycznie „budujmy narzędzie". Z modułu 4 (F. Brooks): rozróżnij **złożoność przypadkową** (da się skrócić narzędziem) od **istotnej** (wynika z procesu, ograniczenia lub decyzji niewidocznej z bliska). Warstwa zespołowa dokłada istniejące systemy, procesy i odpowiedzialności — AI łączy, ale nie zastępuje decyzji, czy połączenie ma sens.

### SaaS to nie tylko funkcje
Antywzorzec **„brakuje jednej funkcji → budujemy lepszy system w weekend"**: porównujesz widoczną funkcję z niewidoczną odpowiedzialnością. SaaS = platforma z infrastrukturą, bezpieczeństwem, uprawnieniami, audytami, niezawodnością, onboardingiem, wsparciem, rozliczeniami, dostępnością i ryzykiem dostawcy. Pełny zamiennik przejmuje też dostęp do danych, konfigurację, logi, błędy API, utrzymanie integracji po zmianach dostawcy, onboarding i ryzyko zbyt szerokiego pokazania danych klienta. Pytanie startowe: nie „czy umiemy zbudować?" (AI coraz częściej: tak), lecz **„czy powinniśmy przejąć tę odpowiedzialność?"**.

### Kup, uzupełnij albo zbuduj
Klasyczne build vs buy rozbijamy na trzy kategorie — w erze AI trzecia jest najczęściej najlepsza:

- **Kup / domyślna funkcja** — problem generyczny: podsumowanie ticketa, sugestia etykiety, widok zaległych issue, raport statusu, dashboard CI; dostawcy coraz częściej to rozwiązują w produkcie.
- **Uzupełnij (complement)** — problem z lokalnego połączenia systemów (GitHub + Linear/Jira + CI + wewnętrzna baza); zespół skleja prawdę ręcznie w głowie; cienki helper ma sens.
- **Zbuduj pełniejszy przepływ** — dopiero gdy helper przetrwa kontakt z rzeczywistością: ktoś używa, oszczędza realny czas, ma właściciela, jasny zakres i nie jest jednorazowym raportem.

Uzupełnienie **nie udaje systemu źródłowego** — nie zastępuje GitHuba, Lineara, Jiry ani CI; czyta sygnały, porządkuje i odsyła ludzi tam, gdzie decyzja powinna paść. Cienki helper najpierw odpowiada: czy problem zasługuje na stałą obsługę — nie od razu aplikacja, agent w SDK, pipeline czy rejestr artefaktów.

### Mapa okazji dla internal buildera
**Internal builder** = programista widzący lokalne tarcie i budujący najmniejsze narzędzie usuwające ból, bez przebudowy firmy. **Mapa okazji** = krótka klasyfikacja jednego sygnału (notatka robocza, nie backlog produktu); chroni przed „nic nie zrobimy" i „własny system do wszystkiego". Cztery pola:

| Pole | Pytanie |
| --- | --- |
| Sygnał tarcia | Co powtarza się tak często, że kosztuje czas lub uwagę? |
| SaaS / domyślna odpowiedź | Czy istniejące narzędzia rozwiązują to wystarczająco? |
| Cienki helper | Jaka najmniejsza warstwa połączy lokalne sygnały? |
| Pierwsza użyteczna wersja | Jak sprawdzimy wartość bez produktu, logowania i wdrożenia? |

Trzy wzorce: (1) **Poranny status** — digest łączący zaległe issue, ryzykowne PR-y, nieudane CI, brak właściciela, status release'u → statyczny HTML z JSON/CSV/API. (2) **Review bez kryteriów** — klasyfikacja ryzyka PR + sugestia recenzentów → komentarz agenta pod PR na przykładach. (3) **Artefakty AI kopiowane ręcznie** — jedno źródło prawdy → repo z paczką testową, ręczna instalacja w jednym projekcie. Wzorzec: kwalifikacja przed kodem. Ograniczenie: szybki prototyp ≠ nieograniczona przepustowość — lepiej 2–3 przemyślane helpery niż 10 prototypów bez utrzymania. Po mapie: większy/niejasny temat → `/10x-shape` → `/10x-prd` → `/10x-roadmap`; wąski sygnał z jasną pierwszą wersją → `/10x-new` → `/10x-research` → `/10x-plan` → `/10x-implement`. Przed budową: rozmowa z menedżerem i zespołem (często wyjaśnia złożoność istotną vs przypadkową). Walidacja według _The Mom Test_: pytaj o fakty z przeszłości i realne obejścia, nie „czy używalibyście narzędzia" — skill `/10x-mom-test` krytykuje założenia i przygotowuje pytania 1:1 oraz krótką ankietę.

### Przykład: digest tarcia zespołowego
Case study: GitHub, Linear/Jira, CI, notatki release'owe, wewnętrzna baza — każde działa, ale codzienny status wymaga ręcznego skakania. GitHub nie zna logiki klientów enterprise; tracker nie widzi czerwonego CI; CI nie zna kontekstu ticketa; baza ma domenę niewidoczną dla SaaS. Idealny kandydat na **uzupełnienie**, nie zamiennik.

```text
Poranny digest zespołu

Ryzykowne PR-y:
PR #1842 dotyka billing, ma czerwone CI i ticket z klientem enterprise.
PR #1839 czeka 3 dni bez review; właściciel domeny: Marta.

Zablokowane tickety:
PAY-412 ma status "In Review", ale nie ma otwartego PR-a.
AUTH-91 ma zmergowany PR, ale ticket nadal jest "In Progress".

Release:
Release 2026.06.11 ma 2 nieudane joby i 1 ticket z flagą klienta.

Decyzje na dziś:
Dopytać właściciela PAY-412, sprawdzić nieudany job w pipeline'ie wydania i przypisać recenzenta do PR #1839.
```

AI pomaga streścić chaos i nazwać wzorce („brak właściciela", „rozjazd statusu", „ryzyko release'u"), ale największa wartość = **lokalne połączenie sygnałów**, których pojedynczy SaaS z definicji nie widzi. Helper pozostaje skromny: czyta, streszcza, linkuje do źródeł, pomaga podjąć decyzję.

### Pierwsza użyteczna wersja
Najlepszy pierwszy helper jest nudny: bez panelu admina, logowania, wdrożenia, backlogu funkcji i własnej bazy, jeśli wystarczy plik lub statyczny raport. Kolejność: mock Markdown z przykładów → lokalny skrypt na JSON/CSV → statyczny HTML do kanału → digest uruchamiany przez jedną osobę; agent, aplikacja, pipeline i harmonogram dopiero gdy wcześniejsza wersja faktycznie pomaga. AI obniża koszt prototypu, nie zeruje utrzymania — codzienny użytek wymaga właściciela, reakcji na błędy, zachowania przy braku danych i decyzji o widoczności wyniku. Mocki, eksporty, sygnały read-only = start lekki; prawdziwe dane firmowe/klienckie/produkcyjne = wcześniej dostęp, uprawnienia, audytowalność (higiena inżynierska, nie straszenie compliance).

### Dźwignia małych usprawnień
Reputacja w zespołach technicznych często rośnie wokół widocznych usprawnień („to już nie boli"), nie prezentacji o produktywności. Usunięcie wspólnego tarcia buduje zaufanie do decyzji technicznych i rozumienia przepływu pracy — nie magiczna ścieżka awansu, lecz praktyczna dźwignia: klasyfikuj odpowiedzialnie, buduj małą warstwę tam, gdzie ma sens, nie udawaj zamiennika platformy.

### Krok 1: wypisz sygnały tarcia
Wybierz 3–5 powtarzalnych problemów (projekt, zespół, PoC, scenariusz na mockach). Dobre sygnały: konkretne sytuacje („codziennie ręcznie sprawdzamy PR-y blokujące release", „tickety bez zmian w kodzie", „skille AI kopiowane między repo", „te same uwagi review bez twardej bramki"). Słabe = życzenia („dashboard", „agent", „automatyzacja wszystkiego") — hasła do rozpakowania, nie problemy. Skill `/10x-opportunity-map` prowadzi przez klasyfikację i sugeruje następny krok.

### Krok 2: wypełnij mapę okazji
Dla każdego sygnału cztery pola (tabela powyżej). W „SaaS / domyślna odpowiedź" bądź uczciwy — jeśli Linear, Jira, GitHub, Slack lub Notion wystarczają, nie buduj helpera tylko dlatego, że możesz.

### Krok 3: wybierz jeden helper
Wybierz sygnał: regularny, łączy ≥2 źródła informacji lub role, da się sprawdzić bez pełnego produktu. Szablon pierwszej wersji:

```text
Helper:
[nazwa robocza]

Czyta:
[źródła, np. GitHub export, Jira CSV, CI logs, mock data]

Zwraca:
[krótki opis raportu / widoku / digestu]

Nie robi:
[czego świadomie nie budujesz teraz]

Ryzyko danych:
[mock / tylko do odczytu / dane niewrażliwe albo realne dane firmowe lub klienckie; przy realnych danych opisz, jak wcześniej ograniczysz dostęp]
```

Opcjonalnie przed budową: `/10x-mom-test` na mapie okazji.

### Krok 4: czego się spodziewać po ścieżce 10xChampion
Moduł 5 opcjonalny względem certyfikatu Builder (M1–3); ścieżka zespołowa → odznaka **10xChampion**. Dowód = zrzuty ekranu działającego przepływu w twoim kontekście/PoC, nie publiczny kod firmowy. Wystarczy jeden z dwóch projektów M5: (1) **Pipeline CI/CD do review** (M5L2, M5L3) — widok pipeline + job, logi, screenshot komentarza LLM na PR; (2) **Rejestr artefaktów zespołowych** (M5L4) — repo/rejestr, definicja paczki, lista wydanych wersji.

### Deep dive: Kup, uzupełnij albo zbuduj bez popadania w skrajności
> **Deep dive** — AI zmienia ekonomię tylko częściowo: obniża koszt pierwszej wersji, iteracji i klejenia danych, ale nie usuwa utrzymania, odpowiedzialności za dane, obsługi błędów, zmian API dostawcy ani wyjaśniania, dlaczego raport milczy. **Uzupełnij** = systemy źródłowe zostają, obok cienka warstwa pod lokalny problem.

### Deep dive: Niewidzialna praca SaaS-u
> **Deep dive** — W SaaS widzisz ekran, nie tenant isolation, audyty, backupy, uprawnienia, onboarding, wersjonowanie API, wsparcie i niezawodność. Prototyp może wygrać w wąskim przypadku, bo nie niesie ciężaru platformy — to nie znaczy, że wygrał jako system. Helper jak najdłużej zależny od źródeł: linkuje do PR/ticketa/job/rekordu, nie udaje nowego miejsca prawdy.

### Deep dive: Kiedy helper staje się produktem
> **Deep dive** — Jednorazowy raport może być luźny; codzienny używany przez zespół — nie. Sygnały dojrzewania: planowanie dnia na podstawie wyniku; błąd/brak raportu wywołuje zamieszanie; prośby o nowe źródła, kontrolę dostępu, działanie bez ręcznego uruchamiania. Nie przepisuj wszystkiego natychmiast — świadoma zmiana statusu z eksperymentu na utrzymywany przepływ. Moduł 5 = sekwencja: kwalifikacja → agent → pipeline → artefakty po zespole → praca asynchroniczna.
