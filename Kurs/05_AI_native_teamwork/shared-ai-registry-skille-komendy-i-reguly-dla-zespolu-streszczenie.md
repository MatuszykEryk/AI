### Wprowadzenie

Problem zespołowej pracy z AI to nie duplikacja, lecz **utrzymanie i synchronizacja** artefaktów — w multi-repo po miesiącu nikt nie wie, która wersja skilla obowiązuje. Odpowiedź: jedno źródło prawdy + mechanizm dystrybucji dopasowany do środowiska (trzy modele w lekcji).

### Artefakty AI to kod

Skille, reguły i komendy to wykonywane instrukcje, nie notatki na wiki. Nieaktualna wersja to cichy błąd w systemie — potrzebują wersjonowania, kontroli zmian i kontrolowanej dystrybucji, jak biblioteki przed erą npm/PyPI/Maven.

Rejestr pakietów rozwiązuje kopiuj-wklej: wersjonowana paczka, deklaracja zależności, menedżer robi resztę. Dwie kluczowe własności: kontrola dostępu (publiczny/prywatny) i jedno źródło prawdy z historią wersji. Rejestrowi obojętne, co w paczce — `index.js` czy `SKILL.md`. Mechanizm pokazany na npm, ale działa też PyPI, Maven, Cargo; `SKILL.md` to otwarty, przenośny standard.

### Czego potrzebuje dystrybucja artefaktów AI

Miara każdego rozwiązania — pięć wymagań:

- **Jedno źródło prawdy** — jedno miejsce pochodzenia artefaktu.
- **Wersjonowanie** — semantyczna etykieta każdej zmiany.
- **Uwierzytelnianie** — kontrola kto czyta i kto publikuje.
- **Instalacja, aktualizacja i deinstalacja** — kontrolowane wciąganie, podbijanie i czyste usuwanie.
- **Dostarczanie multi-tool** — ten sam artefakt do Claude Code, Cursora, Codexa.

### Repozytorium ze źródłem prawdy

Fundament: **repo źródła prawdy** (jedno miejsce zmian i publikacji) + **repa konsumentów** (projekty zespołu) + **instalator** (wciąga artefakty, układa pod konwencje narzędzia, aktualizuje, usuwa). W najprostszym modelu instalator = skrypt `postinstall`/`preinstall` po pobraniu paczki.

```
ai-toolkit/
├── package.json
├── skills/
│   ├── 10x-plan/
│   └── 10x-research/
├── rules/
│   └── CLAUDE.md
├── prompts/
└── config-templates/
```

Nowy artefakt = plik + podbicie wersji; reszta zespołu dostaje przy `install`.

### Trzy modele dystrybucji

1. **Rejestr, który już masz** — GitHub Packages; domyślna rekomendacja.
2. **Rejestr chmurowy** — np. AWS CodeArtifact + Terraform; dla org z AWS lub własnym Nexusem.
3. **API i CLI** — gdy brak wspólnego rejestru lub wielu stacków; `10x-cli` z kursu.

Matt Pocock (czerwiec 2026) niezależnie opisał ten sam wzorzec: paczka npm ze skillami + `postinstall` symlinkujący do narzędzi.

### A co z marketplace'ami?

Marketplace Claude Code / Cursor to katalog gitowy — sensowny skrót dla zespołu na **jednym** narzędziu. Antywzorzec: **vendor lock-in dystrybucji** — `SKILL.md` jest przenośny, ale marketplace przywiązany do ekosystemu; przy rotacji narzędzi kanał się rozjeżdża.

### Model 1: GitHub Packages

Zespół na GitHubie ma już rejestr. Producent — jedno pole w `package.json`:

```json
{
  "name": "@twoj-zespol/ai-toolkit",
  "publishConfig": {
    "registry": "https://npm.pkg.github.com"
  }
}
```

Konsument — `.npmrc` w repo (bez sekretów):

```
@twoj-zespol:registry=https://npm.pkg.github.com
```

Token (`//npm.pkg.github.com/:_authToken=${GH_PKG_TOKEN}`) tylko ze zmiennej w CI, nigdy w historii gita. Pipeline: merge → CI publikuje → konsumenci `install`. Ten sam toolkit może iść rejestrem (zespół) lub CLI (uczestnik kursu) — rejestr nie gorszy, zależy od odbiorcy.

### Gdzie chowa się jedyna realna trudność

Asymetria uwierzytelniania: **zapis darmowy** — CI używa efemerycznego `GITHUB_TOKEN` + `permissions: packages: write`. **Odczyt problematyczny** — trwały token wszędzie, gdzie instaluje się paczka: dev, CI konsumentów, zewnętrzne buildy (Cloudflare). W tej samej org GitHub `GITHUB_TOKEN` wystarczy; poza org — PAT.

Instalator dokłada `preinstall` do `package.json`:

```bash
[ -n "$GH_PKG_TOKEN" ] && echo '//npm.pkg.github.com/:_authToken=${GH_PKG_TOKEN}' >> .npmrc || true
```

Lokalnie deweloper loguje się `npm login`; w CI sekret wstrzyknięty. Na CF Pages/Workers instalator synchronizuje token przez API platformy (prod + preview). Właściwości instalatora: **idempotencja**, **brak sekretów w repo**, **łagodne łączenie z istniejącym `package.json`**. Łata też workflowy z `actions/setup-node` (`registry-url`, `scope`, `NODE_AUTH_TOKEN`).

### Wersjonowanie bez ręcznego podbijania

Conventional commits (`fix:`, `feat:`, `feat!:`) → patch/minor/major, ale typy bywają błędne (AI też się myli). Druga kontrola w CI: `git diff` — czy faktycznie zmieniły się pliki paczki między tagiem a HEAD. Commit kłamie o intencji, diff pokazuje realność. Narzędzia: [semantic-release](https://semantic-release.org) (auto-publikacja), [release-please](https://github.com/googleapis/release-please) (PR z release'em), lub własna logika CI.

### ⚠️ Model 1 — na co uważać

- **Typ tokena do odczytu to ruchomy cel** — fine-grained PAT dla npm na GitHub Packages bywa niespójny; klasyczne PAT wygaszane; sprawdź aktualną dokumentację przed wdrożeniem.
- **Zewnętrzne platformy buildów** nie widzą sekretów GitHuba — CF wymaga synchronizacji tokena do ich systemu sekretów.
- **GitHub Packages odrzuca duplikat wersji** (409) — ręczne podbijanie prędzej czy później wygeneruje konflikt.

### Model 2: Rejestr jako gotowa infrastruktura

AWS CodeArtifact postawiony Terraformem, auth CI przez OIDC (krótkie tokeny zamiast długiego klucza AWS). Więcej kontroli nad rejestrem niż GitHub Packages; mniej roboty niż własny Nexus/Verdaccio na klastrze. Koszt ~zero w darmowym limicie AWS CA vs ~kilkadziesiąt USD/mies. za Verdaccio na ECS.

Rekomendacja GitHub Packages = najniższy próg wejścia dla zespołu na GitHubie. Model 2 gdy org już na AWS lub potrzebuje zarządzania na poziomie rejestru.

### Jak ten rejestr wygląda od środka

W CodeArtifact „repozytorium" = magazyn paczek, nie repo git. Hierarchia: **domena** (`devs10x`) → repozytoria (prywatne + proxy do publicznego npm). **Scope** (`@10xdevs`) ≠ **domena** (`devs10x`) — dwie różne rzeczy przy logowaniu. ~70 linii Terraform na zasoby, 250–400 z politykami i remote state. OIDC: `permissions: id-token: write`; rola IAM i bucket stanu = jednorazowy bootstrap.

### ⚠️ Model 2 — na co uważać

- **Nazwa domeny od małej litery** — `10xdevs` odrzucone → `devs10x`.
- **Provider AWS ~700 MB** — pierwszy `terraform plan` timeout; ustaw `TF_PLUGIN_TIMEOUT=300`.
- **Pipeline wymaga `permissions: id-token: write`** — bez tego OIDC pada po cichu.

### Wzorce, które przeżyją każdy model

**Sentinel markery dla reguł** — instalator wstrzykuje blok między znacznikami, nie nadpisuje całego pliku:

```
<!-- BEGIN @twoj-zespol/ai-toolkit -->
... reguły zespołowe ...
<!-- END @twoj-zespol/ai-toolkit -->
```

Idempotencja: przy aktualizacji podmienia tylko blok między markerami. Uszkodzony blok (jeden marker zniknął) wymaga osobnej obsługi — inaczej duplikacja reguł.

**Manifest instalacji** — `.claude/.10x-toolkit-manifest.json` zapisuje paczkę, wersję, narzędzie i listę plików; deinstalacja usuwa dokładnie to, co dodała, niezależnie od `node_modules` i hooków menedżera pakietów.

```json
{
  "package": "@przeprogramowani/10x-cli",
  "version": "1.2.0",
  "lessonId": "m1l5",
  "tool": "claude-code",
  "files": {
    "skills": {
      "10x-init": { "files": ["SKILL.md"] },
      "10x-shape": { "files": ["references/prd-schema.md", "SKILL.md"] }
    },
    "configs": ["settings.json.template", "mcp.json.template"]
  }
}
```

**`SKILL.md` jako przenośny standard** — neutralny format; instalator (lub profile CLI) układa ten sam plik w katalog docelowy narzędzia. Zmienia się kanał dostarczania, nie treść.

### Model 3: Pełny produkt z API i CLI

Gdy odbiorcy na różnych stackach lub potrzebujesz dawkowania treści. Dlaczego kurs nie leży na npm: (1) różnorodność stacków łatwiej ogarnąć profilami CLI niż mnożeniem rejestrów, (2) precyzyjne rozsyłanie artefaktów w czasie — w rejestrze paczka albo jest, albo nie.

Trzy warstwy: **źródło prawdy** (bundle artefaktów) → **API** (auth + dystrybucja REST) → **CLI** (zapis pod konwencje narzędzia).

```ts
export const PROFILES = {
  "claude-code": {
    skillPath: (name) => `.claude/skills/${name}/SKILL.md`,
    rulesFile: "CLAUDE.md",
    manifestDir: ".claude",
  },
  cursor: {
    skillPath: (name) => `.cursor/skills/${name}/SKILL.md`,
    rulesFile: ".cursor/rules/10x-course.mdc",
    manifestDir: ".cursor",
  },
  codex: {
    skillPath: (name) => `.agents/skills/${name}/SKILL.md`,
    rulesFile: "AGENTS.md",
    manifestDir: ".agents",
  },
};
```

API nie musi znać konwencji każdego narzędzia — CLI jest „applicatorem".

### API: dostęp, którego rejestr nie udźwignie

Magic link → JWT ~1h, odświeżany w tle; uprawnienia sprawdzane przy każdym odświeżeniu, nie tylko przy logowaniu — odbieranie dostępu w jednym miejscu, token wygasa ≤1h. Wzorzec: wyciągnij uprawnienia z systemu biznesowego do lokalnego magazynu (KV), sprawdzaj lokalnie zamiast odpytywać źródło przy każdym żądaniu. API umożliwia **dawkowanie treści w czasie** — moduł odblokowuje się, gdy ma być dostępny.

### CLI jako granica bezpieczeństwa

**Podpisywanie paczek (Ed25519)** — API podpisuje bundle, CLI weryfikuje przed zapisem (`REQUIRE_SIGNATURES = true`); klucz publiczny wkompilowany w CLI. Najpierw hash treści, potem podpis nad kanonicznym stringiem — ochrona przed MITM na łańcuchu dostaw.

**Allowlista hosta API** — tylko produkcyjny HTTPS lub localhost; bez dowolnej podmiany endpointu (przechwycenie tokenu).

**Guard na sentinel-injection** — reguła zawierająca markery sentinel w treści jest odrzucana; inaczej przy następnym apply CLI mógłby potraktować podrzucony marker jako prawdziwy i skasować treści spoza bloku.

### Tabela decyzyjna i alternatywy

| Wymiar              | Model 1: GitHub Packages | Model 2: CodeArtifact | Model 3: API + CLI  |
| ------------------- | ------------------------ | --------------------- | ------------------- |
| Odbiorca            | zespół na GitHubie       | zespół na AWS         | dowolny             |
| Uwierzytelnianie    | PAT / GITHUB_TOKEN       | IAM / OIDC            | magic link + JWT    |
| Uprawnienia         | członkostwo org/repo     | polityka IAM          | system biznesowy    |
| Odbieranie dostępu  | usunięcie z org          | odłączenie polityki   | usunięcie z systemu |
| Dawkowanie w czasie | nie                      | nie                   | tak                 |
| Multi-tool          | instalator               | instalator            | profile CLI         |
| Koszt uruchomienia  | ~zero                    | ~70 linii TF + AWS    | pełny produkt       |
| Koszt utrzymania    | tokeny w obcym CI        | rotacja tokenów, IAM  | API, auth, klient   |

**Pierwszy wiersz (odbiorca) rozstrzyga wybór** — reszta doprecyzowuje. Starter paczki: `npx @przeprogramowani/10x-cli@latest get m5l4`. Ścieżka budowy: `/10x-new` → `/10x-research` → `/10x-plan` → `/10x-implement`; przy niepewności wcześniej `/10x-shape` → `/10x-prd` → `/10x-roadmap`.

### Najczęstszy błąd

Antywzorzec: **dystrybucja pod CV** — API+CLI lub Terraform, gdy odbiorcą jest zespół wyłącznie na GitHubie. Wybieraj model dopasowany do odbiorcy, nie najbardziej imponujący.

### Deep dive: Anatomia instalatora

> **Deep dive** — Dwa tryby: **symlink** (`npm install`) — artefakty wędrują z repo; **copy** (`npx ... install`) — fizyczna kopia, działa bez `package.json` (Python, Go, Rust); copy obowiązkowy przy `npx` (ulotny cache). Manifest z trybem i listą plików → pewna deinstalacja bez polegania na `node_modules`. Skille z `requires` we frontmatterze — instalator dociąga zależności; preset nie może po cichu pominąć wymaganego skilla. Migracja: sprząta stare wersje; przy uszkodzonym manifeście woli zostawić pliki niż ryzykować utratę.

### Deep dive: Kontrakt między repo prywatnym a publicznym CLI

> **Deep dive** — Prywatne repo treści + publiczny CLI spotykają się na OpenAPI 3.1; CLI generuje typy na buildzie. Zmiana API łamiąca klienta wychodzi przy kompilacji CLI, nie u użytkownika. Wzorzec na dwa niezależnie ewoluujące repozytoria, które nie mogą się rozjechać.
