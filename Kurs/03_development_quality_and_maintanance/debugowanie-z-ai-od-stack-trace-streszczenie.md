### Wprowadzenie

Pipeline z M3L1–M3L4 łapie znane ryzyka proaktywnie; produkcja ujawnia nieprzewidziane. Ta lekcja: agent zbiera dowody z wielu źródeł, formułuje hipotezę, potwierdza testem — diagnostyka domykająca lukę po testach. Struktura dochodzenia jest przenośna (Sentry/Datadog, `wrangler tail`/`vercel logs`, Playwright/Cypress). Bug w 10xCards: ocena „Dobre" nie przesuwa harmonogramu powtórki.

### Parsowanie ticketa z agentem

Ticket = najtrudniejszy punkt wejścia — jedno nieprecyzyjne zdanie zamiast stack trace. Zasada z M3L2: uporządkowany kontekst. Daj agentowi surowy ticket, poproś o strukturę: kroki reprodukcji, zakres, częstotliwość, możliwy obszar (np. trasa oceniania → `review_states`).

### Triage — gdzie szukać najpierw

„Sukces API, brak efektu" → monitoring (cichy błąd). Różne symptomy → różne źródła (unit test, Sentry, ticket). Tu: 200 OK, harmonogram się nie zmienia → Sentry.

### Sentry MCP — dane z produkcji

Sentry: wyjątki, stack trace, breadcrumbs. **MCP** do przeszukiwania issues (CLI = release/source maps, nie search). Przy `captureConsoleIntegration` `console.warn` trafia do Sentry mimo 200 — issue `review/rate: update_failed` w `rate.ts`. Warstwa 1: błąd istnieje, lokalizacja, kontekst oceniania.

### Runtime logs jako uzupełniający sygnał

```bash
npx wrangler pages deployment tail --project-name 10xcards-30 \
  --format json --search "update_failed"
```

Logi potwierdzają systematyczny błąd przy każdej ocenie. Sentry = lokalizacja; logi = skala. Wzorzec: filtruj strumień logów platformy po komunikacie symptomu.

### Reprodukcja lokalna

Playwright CLI/MCP do diagnostyki (M3L4): `browser_console_messages` (brak błędów JS), `browser_network_requests` (POST 200, GET powtórki — ta sama karta). Problem serwerowy w danych, nie UI. Antywzorzec: czytanie kodu „w ciemno" — zawodzi przy produkcji, niereprodukowalności, losowości; dowody wskazują plik i oddzielają front od backendu.

### Sygnał wizyjny: reprodukcja problemów UI

Bugi layoutu (z-index, nachodzenie) — DOM milczy. Screenshot + VLM z pytaniem otwartym (`--caps=vision`) vs weryfikacja w testach E2E (wąska asercja). Kategorie modeli: frontier, budżetowy (np. Gemini Flash), open-weight. Ograniczenia: spatial bias, halucynacje — traktuj jako hipotezę. W testach: `toMatchSnapshot`, Argos; vision w debugowaniu, nie domyślnie w E2E.

### Synteza — dowody się zbiegają

Root cause w `rate.ts`:

```typescript
if (updateError) {
  console.warn(`review/rate: update_failed ...`);
  return jsonResponse(200, { ok: true }); // błąd połknięty
}
```

UPDATE pada, frontend widzi 200 i kartę z pamięci; `due` nie zmienia się → ta sama karta wraca. Żadne źródło samo nie wystarczyło: Sentry (gdzie), logi (skala), Playwright (200 vs brak zapisu), kod (połknięty błąd).

### Najpierw test, który reprodukuje buga

**Test-driven bugfixing** (Kent Beck): symptom → czerwony test → fix → zielony. Test integracyjny na **zapisanym stanie w bazie**, nie odpowiedzi API. Prompt-wzorzec:

```text
Mamy buga: ocena fiszki zwraca 200, ale harmonogram się nie zapisuje i karta wraca do powtórki.
Napisz test integracyjny dla endpointu oceniania (`rate`), który:
- zaseeduje wiersz w `review_states` z terminem `due` w przeszłości (karta od razu „do powtórki"),
- wywoła ocenę „Dobre" przez endpoint,
- odczyta z bazy ZAPISANY wiersz (nie odpowiedź API) i sprawdzi, że `due` przesunęło się
  w przyszłość, a `reps` wzrosło.
Test ma na tym etapie padać — to reprodukcja buga, nie jego naprawa.
```

`/10x-tdd` dla unit/integration; `/10x-e2e` gdy bug tylko w UI.

### Fix i weryfikacja wielowarstwowa

Fix: propaguj błąd (`500 { error: "rate_failed" }`) + napraw zapis UPDATE. Weryfikacja lustrzana: każdy kanał diagnostyczny potwierdza brak problemu.

### Połknięte błędy — klasa problemów

**Swallowed error** — try/catch loguje i zwraca sukces (OWASP A10:2025). Dlaczego pipeline nie złapał: M3L2 — inna trasa (zapis sesji), `rate.ts` bez testów; M3L3 — typy OK, runtime odrzuca wartość; M3L4 — E2E bez ścieżki oceniania; M3L1 — ryzyko SRS w planie, faza jeszcze nie zbudowana. Monitoring = siatka pod proaktywnymi testami.

### Jeden workflow, cztery punkty wejścia

Ten sam proces: synteza wielu źródeł → test → weryfikacja warstwowa. Wejście zmienia start (unit test = od razu stack trace; ticket = parsowanie). Moduł 3: test plan → unity → hooki → E2E → diagnostyka produkcyjna.

### Deep dive: Sentry — konfiguracja

> **Deep dive** — Developer plan: 5000 zdarzeń/mies. Astro+Cloudflare: `@sentry/astro`, `@sentry/cloudflare`, `captureConsoleIntegration` — warningi zużywają limit; przy ruchu zawęź do `error`. Brak DSN = no-op. Setup przez `/10x-new` → `/10x-plan` → `/10x-implement`.

### Deep dive: Sentry MCP

> **Deep dive** — `npx @sentry/mcp-server@latest --access-token=...`. `search_issues` (pass-through query: `message:update_failed is:unresolved`) → `get_sentry_resource` (stack trace, breadcrumbs).

### Deep dive: Runtime logs i Playwright diagnostyka

> **Deep dive** — `wrangler tail`: brak HTTP status w output; alternatywy Fly/Vercel/CloudWatch. Playwright: `console`, `requests`, `request <index>` — core tools, nie `--caps=network`.
