### Wprowadzenie

Antywzorzec po roadmapie: „zrób teraz cały Stream A" — brzmi profesjonalnie, jest za szerokie. Następny krok:

```
roadmap item -> change-id -> plan.md -> plan review -> implementacja fazy -> Progress
```

**Plan to mechanizm kontroli nad przyszłą pracą agenta** — nie biurokracja. Agent zapisuje założenia, fazy, kontrakt i warunki zakończenia; ty mówisz „tak" lub „stop". Najpierw plan, potem kod.

### Roadmapa jako mapa streamów

```bash
npx @przeprogramowani/10x-cli@latest get m2l2
```

Paczka: `/10x-new`, `/10x-plan`, `/10x-plan-review`, `/10x-implement`. `roadmap.md` to mapa zależności i streamów, nie lista odhaczeń. Stream A 10xCards: F-01 → S-01 → S-02 → opcjonalnie S-03. S-04 (SRS) czeka — wybór biblioteki, `ReviewState`, skala ocen, polityka edycji. S-05 (compliance) poza głównym torem tej lekcji. Ograniczenie celowe.

### Zakres tej lekcji

Jedna rzecz dobrze: fragment roadmapy przez pętlę plan → implementacja z fazami i weryfikacją. Antywzorce: **vibe coding** („zrób wszystko z roadmapy" → duży diff) vs **sztuka dla sztuki** (nadmiar plików markdown przed kodem). Podejście pośrednie: slice'y zrozumiałe, niski koszt, plan → korekta → implementacja.

| Roadmap item | Rola w lekcji |
| ------------ | ---------------- |
| F-01 | podstawa techniczna — szybki plan |
| S-01, S-02, S-03 | plan + implementacja |
| S-04 | lekcja o researchu |
| S-05 | zadanie domowe |

### Change-id jako intencja pracy

Operacyjny identyfikator zmiany:

```text
gate-product-routes
first-gated-generation
atomic-save-to-deck
```

Spina: pozycję z roadmapy, `context/changes/<change-id>/`, `change.md`, `plan.md`, `plan-brief.md`, `/10x-implement <change-id> phase 1`, `## Progress`. Strategia **Write** z context engineeringu — pamięć w repo, nie tylko w rozmowie.

### Skąd się wziął ten workflow

Efekt iteracji: wbudowany Plan Mode ginie z sesją; `/10x-plan` → plan w repo, stała struktura; `/10x-implement` → fazy, testy, commit, `## Progress`. Traktuj skille jako punkt wyjścia, nie ostateczną konfigurację.

### F-01 jako fundament techniczny

```text
/10x-new gate-product-routes F-01 z @roadmap.md
/10x-plan gate-product-routes
/10x-implement gate-product-routes phase 1
```

Workflow dla zadań niskiej/średniej złożoności bez dodatkowego researchu.

### S-01 i S-02 jako pierwsze slice'y funkcjonalne

```text
/10x-plan atomic-save-to-deck
```

`plan.md` minimum: **end state**, **phases** (nie „implement everything"), **Intent + Contract per file**, **Success Criteria**, **Risks / Open Questions**, **`## Progress`**. Drugi artefakt: **`plan-brief.md`** — lekki hand-off do review i korekty kierunku. Różnica od wbudowanego Plan Mode: fazy z weryfikowalnym kontraktem per krok.

### Plan review przed kodem

```text
/10x-plan-review atomic-save-to-deck
```

Kontrola gotowości: zgodność z roadmapą, konkretny end state, wykonalne fazy, nazwane powierzchnie kontraktu, format `## Progress` dla `/10x-implement`, success criteria sprawdzające zachowanie nie tylko istnienie plików. Uwagi na review = najtańszy moment poprawki.

### Implementacja jednej fazy

```text
/10x-implement atomic-save-to-deck phase 1
```

Agent bierze `plan.md`, wykonuje jedną fazę, weryfikuje, manual gate gdzie trzeba, commit, aktualizuje `## Progress`. Nie wymyśla od nowa — trzyma się fazy, raportuje pliki, uruchamia komendy weryfikacji.

### Checkpoint progressu

Sukces lekcji: wybrany stream, change folder, `plan.md` + `plan-brief.md`, plan po review, ≥1 faza zaimplementowana lub checkpoint, zaktualizowane `## Progress`. Nie „całe 10xCards gotowe".

### Deep dive: Dlaczego plan nie jest biurokracją

> **Deep dive** — Plan w pracy z agentem to punkt synchronizacji przed edycją plików (shift-left), nie prognoza. Bez planu widzisz efekt dopiero po diffie — drift. `plan.md` pozwala pytać „czy te pliki są w scope?" zanim agent zinterpretuje S-02 jako przebudowę modelu. Im większe ryzyko i więcej plików, tym ważniejszy plan przed kodem.

### Plan Mode a `/10x-plan`

| Cecha | Wbudowany Plan Mode | `/10x-plan` |
| ----- | ------------------- | ----------- |
| Gdzie żyje | UI / rozmowa | `context/changes/<change-id>/plan.md` |
| Trwałość | sesja | repo |
| Wznowienie | historia narzędzia | change.md, plan.md, plan-brief.md, Progress |
| Roadmapa | ręcznie | change-id |

Historia rozmowy jest krucha; plik w repo trwały. Plan Mode może być interfejsem — finalny kontrakt trafia do repo.

### Architekt i koder

Model mentalny: **architekt** (zakres, ryzyka, kontrakty, fazy), **koder** (egzekucja fazy), **człowiek** (zatwierdzenie kierunku). W prostych zadaniach jedna sesja/model; w trudnych — rozdzielenie planowania i implementacji. Granica artefaktu ważniejsza niż branding modelu.

### Co robić z `Unknowns`

Nie zamykaj wszystkich unknowns przed planowaniem. **Lokalne unknowns** — `/10x-plan` rozstrzyga po przeczytaniu kodu. **Zewnętrzne unknowns** (np. rating scale biblioteki SRS) — research. **Podejrzane założenia** — framing. **Bez wpływu na slice** — `Open Questions` / `Parked`. Chroni przed researchowaniem na zapas.

### Kiedy zatrzymać implementację

Fakt zmieniający kontrakt planu → stop. Przykłady: brak modelu `Deck`, dwie niespójne ścieżki auth, decyzja produktowa w walidacji, przebudowa 10 modułów zamiast 3 plików. Mały mismatch (literówka, inny eksport) — adaptuj. Zmiana end state lub powierzchni kontraktu → wróć do planu, popraw jawnie, kontynuuj.
