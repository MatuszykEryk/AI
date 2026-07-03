### Wprowadzenie

Antywzorzec po implementacji: „skoro działa, to merge". Kod może działać lokalnie, a zmiana nie być gotowa — agent rozwiązał problem źle, rozszerzył zakres, ominął konwencje albo napisał test pod własne błędne założenie. Tryb „approve bez obrony" z preworku. Solo review kodu AI: szybki, systematyczny, wystarczająco ostry — bez klasycznego review zespołowego ani planowania testów (moduł 3).

### Kod działa, ale czy jest gotowy?

Punkt wyjścia: zaimplementowany slice, `plan.md`, `## Progress` z commitami. Review bez planu = diff „na wyczucie". AI pisze rozsądnie wyglądający kod przy niewłaściwym problemie. Pytanie: **czy zmiana dowozi plan zgodnie z wymaganiami?** `/10x-impl-review` — przegląd względem planu, zakresu, reguł projektu i success criteria. Ostateczna decyzja o merge — twoja.

```bash
npx @przeprogramowani/10x-cli@latest get m2l3
```

### Scorecard zamiast intuicji

Sześć wymiarów `/10x-impl-review`:

| Wymiar | Pytanie |
| ------ | ------- |
| Plan Adherence | Czy realizuje zatwierdzony plan? |
| Scope Discipline | Czy bez zmian spoza zakresu? |
| Safety & Quality | Czy bez ryzyk bezpieczeństwa, danych, stabilności? |
| Architecture | Czy pasuje do architektury projektu? |
| Pattern Consistency | Czy lokalne wzorce, nie własne? |
| Success Criteria | Czy kryteria sukcesu naprawdę spełnione? |

Filtr uwagi: najpierw odchylenia od planu, zakres, bezpieczeństwo, architektura, wzorce — potem drobiazgi. Łatwo zauważyć nazwę zmiennej; trudno nowy format API albo test sprawdzający implementację zamiast zachowania.

### Wyniki to nie lista „napraw wszystko"

Findings przez dwie osie: **severity** (krytyczny/ostrzeżenie/obserwacja) i **impact** (lokalna poprawka vs architektura, produkt, dane, kontrakt). Wysoka severity + niski impact → napraw od razu. Niska severity + wysoki impact (np. rename statusu w bazie) → nie „fix" z rozpędu — wygląda jak porządek w enumie, dotyka frontendu, backendu, danych historycznych i dokumentacji.

Macierz severity × impact to heurystyka, nie sztywna reguła:

| Severity / Impact | Low | Medium | High |
| ----------------- | --- | ------ | ---- |
| Critical | Napraw teraz | Napraw ostrożnie | Zatrzymaj się i zawęź fix |
| Warning | Fix albo skip | Decyzja zależna od kontekstu | Często wymaga planowania |
| Observation | Zwykle skip | Zostaw notatkę albo regułę (`/10x-lesson`) | Zbadaj, czy problem nie dotyka wielu miejsc |

**Nie każdy finding musi być poprawiony.** Decyzje: fix now, fix differently, skip, record as lesson (`/10x-lesson`). Zalecane: zapisz raport, przejrzyj bez presji, potem `/10x-impl-review @ścieżka-do-raportu`.

### Triage: finding po findingu

Cztery reakcje:

1. **Napraw teraz** — prawdziwy, istotny, w zakresie (np. endpoint omija auth).
2. **Napraw inaczej** (`Fix differently`) — lepsze rozwiązanie niż propozycja agenta.
3. **Nie naprawiaj teraz** (`Skip`) — niski wpływ; poważniejsze odkładane → „Other" (accept risk); błędny finding → dismiss ze śladem w raporcie.
4. **Buduj pamięć projektu** (`Record as lesson`) — powtarzalny wzorzec → `context/foundation/lessons.md`; agent pyta, czy zastosować poprawkę od razu.

### Jak czytać diff agenta

Kolejność czytania:

1. **Zakres** — tylko pliki z planu? Slice o zapisie kart, a diff w routingu i deployu → stop.
2. **Granice ryzyka** — auth, zapis danych, migracje, zewnętrzne API, konfiguracja środowiska.
3. **Lokalne wzorce** — helpery, formaty błędów, struktura folderów, testy (Pattern Consistency).
4. **Testy** — czy złapałyby przyszły błąd, nie tylko „czy są". Zielony wynik ≠ dowód sensu zmiany.

### Werdykt przed merge

Trzy warianty: **Approve** (findings rozwiązane/odrzucone/świadomie zaakceptowane), **Request changes**, **Block** (powrót do planu/założeń). `/10x-impl-review` porządkuje raport; werdykt należy do ciebie. Cykl plan → implementacja → review → triage zamyka jedną zmianę.
