### Gotowy agent kontra agent do złożenia

SDK dzielą się na dwie kategorie rozwiązujące różne problemy — pytanie „które SDK jest najlepsze?" jest źle postawione; właściwe: „chcę gotowego agenta czy framework do samodzielnego złożenia?".
**SDK z gotowym agentem** (Claude Agent SDK, Codex SDK, Cursor SDK) — cały harness: pętla, narzędzia do plików/basha/sandboxingu, czytanie repo. Podajesz cel i opcje deklaratywnie. Cena: przywiązanie do modeli, runtime'u i cennika producenta.
**SDK do złożenia agenta** (OpenRouter Agent SDK, Vercel AI SDK) — składowe i punkty montażu: zawołaj model, uruchom narzędzie, dołącz wynik, powtórz, zatrzymaj. Harness składasz sam, narzędzia własne, interfejs niezależny od modelu.
Antywzorzec: wybór SDK po znajomości marki („używam Claude, to wezmę Claude SDK") — traktowanie kategorii jak kosmetyki, gdy zadanie wymagało drugiej.
Kryterium wyboru: ile elastyczności (model, dostawca, narzędzia) vs szybkie wdrożenie z zaufanym dostawcą. W 10xCards start od Vercel AI SDK — żonglowanie modelami; potem refaktor pod CI/CD.

### Ten sam agent na dwóch SDK

Minimalny agent code review: wejście = diff z gita, wyjście = JSON z pięcioma kryteriami 1-10 (poprawność implementacji, idiomatyczność, złożoność, pokrycie testami względem ryzyka, bezpieczeństwo), werdykt pass/fail, podsumowanie Markdown. Uruchomienie: `git diff | npx tsx review.ts`. Pełne przykłady w [agent-sdk-examples](https://github.com/przeprogramowani/agent-sdk-examples).
Wspólny moduł (`common/review-schema.ts`): prompt systemowy i schemat `zod` jako jedno źródło prawdy. AI SDK bierze zod wprost; Claude Agent SDK dostaje JSON Schema przez `z.toJSONSchema()`.

```typescript
const SYSTEM_PROMPT = `Jesteś precyzyjnym, konstruktywnym recenzentem kodu oceniającym pull request.
Oceń podany diff w pięciu kryteriach w skali 1-10 (1 = poważne braki, 10 = wzorowo):
poprawność implementacji, idiomatyczność, złożoność, pokrycie testami względem ryzyka, bezpieczeństwo.
Następnie wydaj wiążący werdykt (pass/fail) dla całej zmiany i dołącz krótkie podsumowanie (2-3 zdania)
w Markdown, na podstawie którego autor PR-a będzie mógł działać.`;
```

`.describe()` na polach schematu to istotna dźwignia sterowania modelem — nie kosmetyka. Zakres 1-10 wymuszamy opisem i promptem, bo structured output Anthropica odrzuca minimum/maximum na integer. Rubryka skrajnych ocen w `.describe()` kalibruje model iteracyjnie.

#### Wersja z gotowym agentem (Claude Agent SDK)

Setup: `npm init -y && npm pkg set type=module`, `npm install @anthropic-ai/claude-agent-sdk zod`, `npm install -D tsx @types/node`. Uruchomienie: `git diff | npx tsx review.ts`.
Cały workflow w `query()` — deklaratywnie: `systemPrompt`, `model: "claude-sonnet-4-6"`, `tools: []`, `maxTurns: 2`, `outputFormat: { type: "json_schema", schema: REVIEW_JSON_SCHEMA }`. Za darmo z harnessu: pętla agenta, walidacja structured output, kilkanaście narzędzi (tu wyłączone). `maxTurns: 2` bo: tura 1 = analiza diffa, tura 2 = structured JSON. `structured_output` typowane jako `unknown` — końcowa walidacja przez `REVIEW_SCHEMA.safeParse()`. Błędy: gdy `subtype !== "success"`, czytasz `errors` sam — inaczej surowy wyjątek. Aktywna sesja Claude Code = start bez jawnego klucza (zmienia się w CI/CD).

#### Wersja do złożenia (Vercel AI SDK 6)

Trzy paczki: `ai`, `@openrouter/ai-sdk-provider`, `zod`. `ToolLoopAgent` z `openrouter("z-ai/glm-5.1")`, `instructions: SYSTEM_PROMPT`, `tools: {}`, `Output.object({ schema: REVIEW_SCHEMA })`, `stopWhen: stepCountIs(2)`. Domyślny limit AI SDK to `stepCountIs(20)` — świadomie skracamy. Podmiana dostawcy = jedna linijka importu (`@ai-sdk/anthropic`, `@ai-sdk/openai`). `OPENROUTER_API_KEY` wymagany jawnie. AI SDK 5: eksperymentalny `Experimental_Agent` z polem `system`; w 6 stabilny `ToolLoopAgent` z `instructions` (stabilny od grudnia 2025).
Wybór: gotowy agent = szybkie wdrożenie na systemie plików z zaufanym dostawcą; Vercel AI SDK = kontrola modelu, dostawcy i narzędzi.

### Co tak naprawdę robi pętla agenta

Workflow agenta = **tool loop** (pętla narzędziowa) — core skryptowego agenta; wpływa na koszty, możliwości i odporność.
Pięć kroków (Anthropic):

1. **Wejście** — prompt + system + definicje narzędzi + historia.
2. **Decyzja** — tekst, wywołanie narzędzi, lub oba.
3. **Wykonanie narzędzi** — uruchomienie i zebranie wyników.
4. **Powrót** — wyniki do kolejnej rundy; kroki 2-3 w kółko.
5. **Wynik** — tekst bez żądania narzędzi = koniec pętli.
   Jedno okrążenie = jedna **tura** (`maxTurns`, `stepCountIs`). Review świadomie 2-3 tury.
   Kluczowe pytanie: **kto pisze kroki 3 i 4?** Gotowy agent (`query()`, `ToolLoopAgent`) chowa pętlę. W AI SDK 5 pisałeś ręcznie:

```typescript
const messages: ModelMessage[] = [{ role: "user", content: prompt }];
let step = 0;
while (true) {
  const result = await generateText({ model, messages, tools });
  messages.push(...result.response.messages);
  if (result.finishReason !== "tool-calls") break;
  for (const call of result.toolCalls) {
    const output = await runTool(call);
    messages.push({
      role: "tool",
      content: [
        {
          type: "tool-result",
          toolCallId: call.toolCallId,
          toolName: call.toolName,
          output,
        },
      ],
    });
  }
  if (++step >= MAX_STEPS) break;
}
```

Kontrolujesz sam: `messages`, pętlę `while`, `finishReason`, wywołanie narzędzi, format `tool`, warunek stopu — pomyłka = agent ignorujący wynik narzędzia lub nieskończona pętla. `ToolLoopAgent` w AI SDK 6 zawija ten szkielet pod `generate()`; deklarujesz `model`, `tools`, `output`, `stopWhen`. Różnica kategorii: co _poza_ pętlą dostajesz (sandbox, compaction, narzędzia) vs co budujesz sam (model, narzędzia, persystencja).

### Czy agent pamięta poprzedni przebieg sesji

Lokalny `review.ts` = pamięć ulotna; każde uruchomienie od zera — OK dla jednorazowej recenzji diffa. W obiegu pracy (poprawki, dopytywanie) wraca pytanie: gdzie mieszka stan?
**Gotowy agent trzyma sesję sam.** Claude Agent SDK: `session_id` z `init`, potem `resume: sessionId` — kontekst wraca bez ponownego wczytywania diffa:

```typescript
let sessionId: string | undefined;
for await (const message of query({
  prompt: `Zrecenzuj ten diff:\n\n${diff}`,
  options,
})) {
  if (message.type === "system" && message.subtype === "init")
    sessionId = message.session_id;
}
query({
  prompt:
    "Autor naniósł poprawki. Sprawdź, czy adresują twoje wcześniejsze uwagi.",
  options: { ...options, resume: sessionId },
});
```

Możliwy `fork` alternatywnej ścieżki bez psucia oryginału. Codex: wątki w `~/.codex/sessions`, `resumeThread`.
**Wersja do złożenia nie ma sesji.** Każde `generate()` = osobny strzał; `ToolLoopAgent` trzyma konfigurację, nie historię. Kontynuacja = własna tablica `messages` przekazywana przy każdym wywołaniu. Uwaga: `output` ustawiasz w konstruktorze agenta — druga tura ze structured output wymusi schemat; na pytanie uzupełniające tekstem użyj osobnego agenta bez `output` z tą samą tablicą `messages`.
Różnica kluczowa przy agentach asynchronicznych (M5L5): agent wraca po godzinach i musi wiedzieć, na czym skończył.

### Co twój agent dziedziczy z repo: reguły, skille, bezpieczeństwo

Reguły repo (`CLAUDE.md`, `AGENTS.md`, skille, hooki) to pliki, które _ktoś_ musi wczytać — zależy od kategorii SDK.
**Gotowy agent dziedziczy konfigurację repo** — ten sam harness co terminal:

| Co                        | Skąd ładuje (Claude Agent SDK)     |
| ------------------------- | ---------------------------------- |
| Pamięć projektu           | CLAUDE.md lub .claude/CLAUDE.md    |
| Skille                    | .claude/skills/\*/SKILL.md         |
| Komendy (legacy)          | .claude/commands/\*.md             |
| Hooki                     | konfiguracja + callbacki w opcjach |
| Uprawnienia / tryb edycji | allowedTools, permissionMode       |
| Serwery MCP, pluginy      | mcpServers, plugins                |

Niuans w `review.ts`: własny `systemPrompt` + `tools: []` = świadome odcięcie od pamięci projektu (wąska, przewidywalna recenzja). Aby dziedziczyć reguły repo:

```typescript
const result = query({
  prompt: `Zrecenzuj ten diff:\n\n${diff}`,
  options: {
    systemPrompt: { type: "preset", preset: "claude_code" },
    settingSources: ["project"],
    skills: "all",
    allowedTools: ["Read", "Glob", "Grep"],
  },
});
```

`skills: "all"` to jedyne miejsce udostępniające skille modelowi; `settingSources` ładuje `.claude/` z `cwd` — jeśli skille leżą gdzie indziej, ustaw `cwd` jawnie. Codex SDK = subprocess CLI (`AGENTS.md`, `~/.codex/config.toml`, nadpisania przez `config`). Cursor SDK = `.cursor/skills/`, `.cursor/hooks.json`.
**Agent do złożenia nie dziedziczy niczego** — reguły wczytujesz własnym kodem (`readFileSync("CLAUDE.md")` → `instructions`). Skille, hooki, pamięć projektu = wzorce do samodzielnej implementacji.
**Bezpieczeństwo** tym samym podziałem: gotowy agent — `allowedTools`, `permissionMode`, hook `PreToolUse`, presety sandboxa Codexa. Wersja do złożenia — tylko narzędzia, które sam napisałeś; bramka = `needsApproval` (Vercel) lub tryby HITL (OpenRouter). Przy inwestycji w reguły ekosystemu — gotowy agent z tego samego środowiska; przy niezależności od dostawcy — wspólny rejestr zespołowy (M5L4).

### Kto teraz widzi twój diff

Polityka prywatności wynika ze **ścieżki uwierzytelnienia**, nie z SDK.
Claude Agent SDK — dwa reżimy:

- **Klucz komercyjny** — brak treningu na treści, retencja 30 dni.
- **Login subskrypcyjny** (Claude Code bez klucza) — warunki konsumenckie; od 28.08.2025 trening domyślnie włączony, retencja do 5 lat.
  Reguła: **najtańsza i najwygodniejsza ścieżka bywa najbardziej „przeciekająca"** — login konsumencki Claude, darmowy ChatGPT w Codexie, darmowe modele OpenRoutera.
  Vercel AI SDK domyślnie nie stoi na drodze danych — żądanie idzie wprost do providera. Vercel AI Gateway opcjonalnie wpina Vercela (tylko USA). Cursor: brak samoobsługowej rezydencji UE. Wersja do złożenia = kontrola po twojej stronie.

### Kontrola kosztów

Ta sama ścieżka uwierzytelnienia decyduje o rachunku.
**Tokeny:** Claude Agent SDK — `message.usage`, `total_cost_usd`, `modelUsage` w `result`. AI SDK — `totalUsage` obok `output` z `generate()`.
**Zużycie narastająco:** `onStepFinish` w AI SDK — `stepNumber`, `usage`, `finishReason` po każdej turze.
**Koszt w USD:** Claude Agent SDK liczy za ciebie (`total_cost_usd`). AI SDK daje tokeny; OpenRouter jako _provider_ zwraca koszt w `providerMetadata.openrouter.usage.cost` przy `usage: { include: true }`.
**Twardy limit:** `maxBudgetUsd` (Claude Agent SDK) → `error_max_budget_usd`. `maxCost(0.5)` w **OpenRouter Agent SDK** (osobne SDK, nie provider pod Vercel!) — ta sama marka, dwie różne rzeczy. Vercel AI SDK 6: brak nazwanego `maxCost` — własny `StopCondition` zliczający koszt po krokach.

```typescript
// Claude Agent SDK: options: { maxBudgetUsd: 0.5, ... }
// OpenRouter Agent SDK: stopWhen: maxCost(0.5)
const { output, totalUsage } = await reviewer.generate({ prompt });
// totalUsage.inputTokens, totalUsage.outputTokens
```

`onStepFinish` po każdej turze: `stepNumber`, `usage`, `finishReason` — telemetria per krok do wykrycia zapętlenia przed `stepCountIs`.
Od 15.06.2026: programistyczne użycie Claude na subskrypcji zużywa osobny miesięczny kredyt **Agent SDK**, oddzielony od limitów interaktywnych. Dobór modelu do roli: Sonnet jako domyślny w review (Opus droższy, Haiku tańszy).

### Deep dive: Co można z tego zbudować

> **Deep dive** — schemat: zdarzenie z SDLC → agent robi żmudny kawałek → ustrukturyzowany wynik. Zastosowania: code review przed człowiekiem (M5L3), self-heal czerwonego builda/ticketa, triage backlogu, migracje/codemody, uzupełnianie testów/docstringów/changelogów. Antywzorzec agenta 24/7 = przepalanie budżetu. Reguły: odpalaj na _zdarzenie_ (PR, czerwony build, nowe zgłoszenie); najtańszy model wystarczająco dobry do zadania.

### Deep dive: Cheatsheet agentowych SDK

> **Deep dive** — decyzja w dwóch ruchach: kategoria, potem ekosystem.

| SDK                  | Kategoria   | Mocna strona                                                      | Na co uważać                          |
| -------------------- | ----------- | ----------------------------------------------------------------- | ------------------------------------- |
| Claude Agent SDK     | gotowy      | 14+ narzędzi, structured output, `total_cost_usd`, `maxBudgetUsd` | tylko modele Anthropic; TS V2 preview |
| Codex SDK            | gotowy      | CLI `codex`, sandbox per tura                                     | wymaga aktualnego CLI                 |
| Cursor SDK           | gotowy      | cloud/self-host/local, indeks repo                                | beta, brak structured output          |
| OpenRouter Agent SDK | do złożenia | 400+ modeli, fallback, `maxCost`                                  | narzędzia sam; to SDK OpenRoutera     |
| Vercel AI SDK 6      | do złożenia | standard TS/Next.js, streaming, structured output                 | brak `maxCost`; v7 w canary           |

Skróty: agent kodujący na FS → gotowy agent po ekosystemie (Anthropic = Claude, OpenAI = Codex, flota w chmurze = Cursor). Wybór modelu + twardy cap → OpenRouter Agent SDK lub `maxBudgetUsd`. TS/Next.js framework → Vercel AI SDK 6.
Język/runtime (połowa 2026): Claude Agent SDK — Python, TypeScript; Codex SDK — TS/JS (Node 18+), Python 3.10+; Cursor SDK — tylko TypeScript; OpenRouter Agent SDK — TypeScript, Python; Vercel AI SDK 6 — TS/JS, integracje UI uniwersalne (Node, Next.js, React, Svelte, Vue, Expo).

### Deep dive: Ten sam model, inne efekty?

> **Deep dive** — model nie jest przenośnym towarem; laby trenują z własnym harnessiem (konwencje narzędzi, schematów, protokołów). Ten sam Claude Opus: 77% domyślnie vs 92,1% z dopasowanym `CLAUDE.md` w Claude Code; 93% w Cursorze — ten sam model, inne opakowanie. Terminal-Bench 2.0: kolumna „Agent" = harness, „Model" = wagi; te same wagi kilka–kilkanaście miejsc od siebie. Część wyników = customowi agenci pod benchmarki (nieoficjalne przebiegi). Praktyczny odruch: model i harness jak para; po podmianie modelu **zmierz ponownie**.

### Deep dive: Agent jako samodzielna, wdrażalna jednostka

> **Deep dive** — Cloudflare Agents SDK (`npm install agents`): agent na Durable Objects z SQLite, harmonogramem, globalnym URL.
> **Durable Objects** vs bezstanowe Workers: unikalna tożsamość, pojedynczy wątek, SQLite kolokowana, uśpienie bez utraty stanu. Klasa `Agent<Env, State>` = Durable Object class; instancja (np. `pr-123`) = niezależny byt.

```typescript
export class ReviewAgent extends Agent<Env, { reviewCount: number }> {
  initialState = { reviewCount: 0 };
  async onRequest(request: Request) {
    const diff = await request.text();
    const result = await runReview(diff, this.env.AI);
    this.setState({ reviewCount: this.state.reviewCount + 1 });
    return Response.json(result);
  }
}
```

`this.state` persystowany w SQLite; `this.env` = bindingi (Workers AI bez klucza, KV, R2). Hibernacja po ~70-140 s: przeżywa `this.state` i `this.schedule`; przepadają zmienne in-memory i timery. Reguła: **co ma przeżyć → `this.state` lub SQL**.
`this.schedule(600, "runReview", { prId: "123" })` — harmonogram per-instancja, idempotentny, w SQLite. `wrangler.toml`: `[[durable_objects.bindings]]`, `new_sqlite_classes` (migracja SQLite), opcjonalnie `[ai]` binding. Sens gdy: webhook/HTTP, stan między wywołaniami, odpowiedź asynchroniczna, cron bez osobnej usługi, brak własnej infrastruktury.
