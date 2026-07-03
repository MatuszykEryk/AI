### Co czyni stack „agent-friendly"
Kursowy stack (Astro, React, TypeScript, Tailwind, Supabase, Cloudflare) to **rekomendacja** i najlepiej wspierana ścieżka — nie warunek ukończenia ani lista technologii do opanowania przed startem. Antywzorzec: „muszę się tego nauczyć przed kursem". Model przewiduje tokeny ze wzorców treningowych; stack agent-friendly daje mu czytelne sygnały. Cztery kryteria: **typowany** — jawne kontrakty (TypeScript, Java, C#, Python z type hints) zmniejszają zgadywanie pól i kształtów danych; **bazujący na konwencjach** — przewidywalne miejsca na routing, komponenty, backend, testy, dojrzały sposób tworzenia projektu zamiast improwizacji; **popularny w danych treningowych** — dużo publicznych przykładów i repozytoriów (egzotyczny stack jak mapa bez nazw ulic); **dobrze udokumentowany** — oficjalna dokumentacja i aktualne przykłady, żeby agent trzymał się prawdziwych API zamiast wymyślać metody. Agent działa najlepiej z jasnym kontekstem, a stack jest jego częścią.

### Gdzie zły wybór potrafi dać w kość
Najgorszy wybór to nie „inny niż kursowy". Sygnał ostrzegawczy: często poprawiasz agenta nie dlatego, że źle rozwiązał problem, lecz że nie potrafi sprawnie korzystać z twoich technologii. Zamiast „czy da się w tym pisać z AI?" pytaj: „czy agent dostanie typy, konwencje, starter i aktualną dokumentację?" Luki uzupełnisz przez `AGENTS.md`, `CLAUDE.md` lub rules, ale to dodatkowa praca i słabszy efekt niż stack dostarczający te elementy naturalnie.

### Nasz rekomendowany stack
Stack pokrywa warstwy od UI po deployment; domyślna ścieżka z pełnym wsparciem kursu i starterem [10x-astro-starter](https://github.com/przeprogramowani/10x-astro-starter), ale nie jedyna.

| Warstwa               | Rekomendacja       | Dlaczego u nas świetnie działa                                                                                                                                                                                                                            |
| --------------------- | ------------------ | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Meta-framework + API  | **Astro 6**        | Dużo konwencji, architektura islands, endpointy API i natywna integracja z Cloudflare                                                                                                                    |
| Komponenty UI         | **React 19**       | Ogromna baza wzorców i dobre wsparcie w narzędziach AI                                                                                                                                                                                                    |
| System typów          | **TypeScript**     | Jawne kontrakty, które agent czyta przy każdej decyzji                                                                                                                                                                                                    |
| Stylowanie            | **Tailwind CSS 4** | Style w klasach HTML — agent widzi strukturę i styl w jednym miejscu                                                                                                                                                                                                     |
| Backend + Baza danych | **Supabase**       | PostgreSQL + auth + storage + SDK w TypeScript; nie zastępuje API w Astro, tylko dostarcza usługi backendowe dla endpointów                                                                                                                          |
| Deployment            | **Cloudflare**     | Pages, Workers, KV + natywny `@astrojs/cloudflare`; krótka ścieżka od lokalnego projektu do wersji online                                                                                                                                          |

**Astro** zarządza routingiem, budowaniem, SSR i endpointami API (formularze, webhooki, pośrednictwo między frontendem a backendem); domyślnie HTML, interaktywne fragmenty jako izolowane „wyspy". **React** tylko tam, gdzie statyczny HTML Astro nie wystarcza. **TypeScript** obejmuje komponenty, endpointy i logikę backendową — główny sygnał nawigacyjny dla agenta. **Cloudflare** odpowiada za deployment i edge runtime.

### Jeśli preferujesz własny stack
Nie musisz porzucać produkcyjnego ekosystemu — większość lekcji dotyczy workflow z agentem, nie składni frameworka. Warunek: stack czytelny dla agenta i znany tobie. Sprawdź:
- Czy ma typy albo przynajmniej jawne schematy danych?
- Czy projekt startuje z oficjalnego startera lub dojrzałego szablonu?
- Czy struktura folderów, routing, testy i konfiguracja są oparte na konwencjach?
- Czy dokumentacja jest aktualna i łatwa do podlinkowania agentowi?
- Czy ty sam rozpoznasz, kiedy agent robi coś niezgodnego z praktyką twojego ekosystemu?
Jeśli większość odpowiedzi to „tak" — realizuj projekt w swoim środowisku; kilka „nie wiem" → kursowy stack bezpieczniejszy na start. Własny stack daje większy transfer do pracy, ale ciężar decyzji technicznych zostaje po twojej stronie.

### Porównaj dwie ścieżki
Nie wybieraj ze sympatii do logo — porównaj przez cztery kryteria agent-friendly:

| Kryterium               | Kursowy stack                                        | Mój stack                                                    |
| ----------------------- | ---------------------------------------------------- | ------------------------------------------------------------ |
| Typy / kontrakty danych | TypeScript w całym projekcie                         | Czy mam typy, schematy albo inne jawne kontrakty?            |
| Konwencje projektu      | Gotowy starter i ustalona struktura                  | Czy startuję z dojrzałego szablonu, a nie z pustego folderu? |
| Popularność wzorców     | Bardzo dużo przykładów webowych w publicznych danych | Czy agent ma z czego czerpać podobne wzorce?                 |
| Dokumentacja            | Oficjalne docs dla każdej warstwy                    | Czy mogę podlinkować agentowi aktualne docs i przykłady?     |
| Decyzja                 | Najbezpieczniejsza opcja                             | Dobry wybór, jeśli większość odpowiedzi brzmi „tak"          |

Decyzję domkniesz w module 1 (stack + projekt); na teraz wystarczy orientacja — nie musisz się jeszcze deklarować.

### Ile musisz wiedzieć na start
Liczy się orientacja, nie biegłość — agent pomaga pracować w środowisku, którego nie znasz perfekcyjnie; stack to narzędzie, nie cel nauki. Przed M1 warto mieć: orientację w jakimś systemie typów (rozumieć, dlaczego typy pomagają agentowi); ogólne pojęcie o HTTP i API (endpoint, request, response, JSON); umiejętność uruchomienia startera (`git clone`, instalacja zależności, komenda `dev`). Nie musisz znać składni konkretnego frameworka, konfiguracji linterów (działają w starterach), wdrożenia na chmurę (M1) ani zaawansowanych wzorców ekosystemu. Kurs skupia się na prowadzeniu agenta, weryfikacji jego pracy i bezpiecznym workflow.

## Zasady i procesy
- **Reguła decyzji:** bez silnej preferencji — kursowy stack; przy produkcyjnym stacku zostań, gdy spełnia cztery kryteria agent-friendly.
- **Warto zapamiętać:** nie zaczynaj od pustego folderu — wybierz starter z typami, lintingiem, formatowaniem i działającą komendą `dev`.
- **Akcja:** porównaj kursowy i własny stack w tabeli z tej lekcji; nie musisz finalizować wyboru, ale wiedz, skąd bierze się decyzja.
