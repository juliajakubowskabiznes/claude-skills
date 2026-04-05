---
name: dev-browser
description: Sterowanie prawdziwą przeglądarką z persystentnym stanem sesji (cookies, localStorage, logowanie). Używaj do nawigacji po stronach, wypełniania formularzy, robienia screenshotów, scrapingu, testowania web appek, automatyzacji powtarzalnych akcji w przeglądarce, ekstrakcji danych z narzędzi bez API, research potencjalnych klientów, monitoringu konkurencji, LinkedIn analytics. Trigger phrases — PL: "wejdź na [url]", "otwórz stronę", "kliknij", "wypełnij formularz", "zrób screenshot", "scrapuj", "zautomatyzuj to w przeglądarce", "przetestuj stronę", "zaloguj się na [serwis]", "pobierz dane ze strony", "sprawdź stronę", "research klienta [firma]", "zobacz co pisze konkurencja"; EN: "open [url]", "click", "fill form", "screenshot", "scrape", "test the site", "log in", "fetch data from site".
allowed-tools: Bash(dev-browser *)
---

# Dev Browser 

CLI do sterowania prawdziwą przeglądarką (Chromium pod Playwright) za pomocą sandboxowanych skryptów JavaScript. Daemon w tle utrzymuje persystentny stan: cookies, localStorage, sessionStorage i otwarte tab'y przeżywają między wywołaniami skryptów.

**Kluczowa cecha:** **named pages** (`getPage("nazwa")`) trzymają stan między skryptami — zaloguj się raz, kolejne skrypty widzą tę samą sesję. Anonimowe strony (`newPage()`) zamykają się po zakończeniu skryptu.

## Instalacja

```bash
npm install -g dev-browser
dev-browser install      # pobiera Chromium (~150MB, raz na instalację)
```

## Środowisko sandboxa

Skrypty działają w **QuickJS**, nie w Node.js. Oznacza to że standardowe API Node są **niedostępne**:

**❌ NIEDOSTĘPNE:** `require()`, `import()`, `process`, `fs`, `fetch`, `WebSocket`, `__dirname`, `Buffer`, `global`

**✅ DOSTĘPNE globale:**
| Global | Opis |
|---|---|
| `browser` | preconnected handle do daemona przeglądarki |
| `console` | `log`, `warn`, `error`, `info` |
| `setTimeout` / `clearTimeout` | timery |
| `saveScreenshot(buf, name)` | zapis screenshota (async) — zwraca ścieżkę |
| `writeFile(name, data)` | zapis pliku do `~/.dev-browser/tmp/` (async) |
| `readFile(name)` | odczyt pliku z `~/.dev-browser/tmp/` (async) |

Pliki I/O są ograniczone do `~/.dev-browser/tmp/` — to jest jedyne miejsce gdzie skrypt może czytać/pisać.

## Sposoby wywołania

**Preferowany — inline heredoc:**
```bash
dev-browser <<'EOF'
const page = await browser.getPage("main");
await page.goto("https://example.com");
console.log(await page.title());
EOF
```

**Z pliku (gdy skrypt długi lub będzie używany wielokrotnie):**
```bash
dev-browser run script.js
```

**Z pipe (jednolinijkowce):**
```bash
echo 'console.log(await browser.listPages())' | dev-browser
```

**Headless (bez widocznego okna — dla automatyzacji w tle):**
```bash
dev-browser --headless <<'EOF'
const page = await browser.getPage("bg");
await page.goto("https://example.com");
console.log(await page.textContent("h1"));
EOF
```

**Connect — podłączenie do istniejącego Chrome użytkownika (KLUCZOWE dla autoryzacji):**
```bash
# Użytkownik uruchamia Chrome z: chrome --remote-debugging-port=9222
dev-browser --connect <<'EOF'
const page = await browser.getPage("main");
await page.goto("https://jakis-serwis-gdzie-jestem-zalogowany.com");
console.log(await page.title());
EOF
```

Tryb `--connect` dziedziczy sesję zalogowanego użytkownika — nie trzeba ponownie wpisywać haseł, działa ze wszystkimi serwisami gdzie użytkownik jest już zalogowany w swojej Chrome.

## Flagi CLI

| Flaga | Znaczenie |
|---|---|
| `dev-browser run <plik>` | Uruchom skrypt z pliku |
| `dev-browser install` | Pobierz/zainstaluj Chromium dla Playwright |
| `--browser <nazwa>` | Osobna instancja (izolowany profil, własne cookies) |
| `--connect [url]` | Podłącz do istniejącej Chrome przez CDP (domyślnie `http://localhost:9222`) |
| `--headless` | Bez widocznego okna — szybsze, do workflow w tle |
| `--timeout <ms>` | Timeout na cały skrypt (domyślnie brak — **zawsze ustawiaj**) |

**Tip produkcyjny:** `--timeout 15000` chroni przed skryptami wiszącymi na brakujących elementach. Bez timeoutu jeden uszkodzony selektor może zawiesić skrypt na nieskończoność.

## API przeglądarki

### Zarządzanie stronami

```javascript
// Named page — persystentna między skryptami. Stan, cookies, localStorage zachowane.
const page = await browser.getPage("main");

// Anonimowa strona — zamyka się automatycznie po zakończeniu skryptu.
const temp = await browser.newPage();

// Lista otwartych tabów (wszystkie named + aktualne anon)
const tabs = await browser.listPages();
// zwraca: [{id, url, title, name}, ...]

// Jawne zamknięcie named page
await browser.closePage("main");
```

**Zasada:** named page używaj gdy potrzebujesz zachować stan (logowanie, wieloetapowy workflow, research w sesji). Anonimową gdy robisz jednorazową operację i nie chcesz zaśmiecać listy otwartych stron.

### Nawigacja i podstawowa interakcja

```javascript
await page.goto("https://example.com");
await page.goto("https://example.com", { waitUntil: "networkidle" });  // czeka aż sieć się uspokoi

await page.click("button.submit");
await page.fill("input[name=email]", "test@test.com");   // natychmiastowe wypełnienie
await page.type("#search", "szukana fraza");              // wpisywanie znak po znaku (symuluje pisanie)
await page.press("#search", "Enter");                     // naciśnięcie pojedynczego klawisza

await page.waitForSelector(".result");                    // czeka aż element się pojawi
await page.waitForURL("**/dashboard");                    // czeka na zmianę URL
await page.waitForLoadState("networkidle");               // czeka aż sieć ucichnie

const text = await page.textContent("h1");
const html = await page.innerHTML(".content");
const value = await page.inputValue("#email");
```

### Targeting elementów — kolejność preferencji

**1. getByRole (accessibility) — najbardziej odporne na zmiany w DOM:**
```javascript
await page.getByRole("button", { name: "Zaloguj" }).click();
await page.getByRole("textbox", { name: "Email" }).fill("test@test.com");
await page.getByRole("link", { name: "Dashboard" }).click();
```

**2. Playwright locator z hasText — dobry kompromis:**
```javascript
const btn = page.locator("button", { hasText: "Zapisz" });
await btn.click();
```

**3. CSS selektory — ostateczność (łamią się przy zmianach CSS):**
```javascript
await page.click(".btn-primary");
await page.fill("#email", "user@test.com");
```

**Zasada:** zawsze preferuj `getByRole` → `locator({hasText})` → CSS. Im wyżej w hierarchii, tym mniej łamie się przy zmianach layoutu strony.

### snapshotForAI — analiza struktury strony

`snapshotForAI()` zwraca uproszczoną reprezentację DOM z informacją o interaktywnych elementach. **Używaj zawsze gdy nie znasz struktury strony** — zamiast zgadywać selektory.

```javascript
const page = await browser.getPage("analyze");
await page.goto("https://example.com");

// Pełny snapshot
const { full } = await page.snapshotForAI();
console.log(full);

// Z opcjami (gdy strona jest bardzo złożona)
const snapshot = await page.snapshotForAI({
  depth: 3,        // głębokość drzewa DOM — mniej = szybciej
  timeout: 5000    // timeout w ms
});
console.log(snapshot.full);
```

**Kiedy używać snapshotForAI vs screenshot:**
- `snapshotForAI()` → gdy potrzebujesz **struktury** (selektory, tekst, rola elementów, state). Szybkie, tanie w tokenach.
- `page.screenshot()` → gdy potrzebujesz **wyglądu** (layout, kolory, CSS, proporcje). Wolniejsze, drogie w tokenach, ale niezbędne do weryfikacji wizualnej.

Debugowanie skryptów który zwraca „element not found" → **zacznij od `snapshotForAI()`**, żeby zobaczyć co faktycznie jest na stronie.

### Screenshoty

```javascript
// Viewport (widoczna część)
const buf = await page.screenshot();
const path = await saveScreenshot(buf, "wynik.png");

// Cała strona (z przewijaniem)
const fullBuf = await page.screenshot({ fullPage: true });
await saveScreenshot(fullBuf, "cala-strona.png");

// Pojedynczy element
const elementBuf = await page.locator(".card").screenshot();
await saveScreenshot(elementBuf, "karta.png");
```

### JavaScript w kontekście strony

```javascript
// Ewaluacja w kontekście przeglądarki — dostęp do window, document, DOM
const title = await page.evaluate(() => document.title);
const scrollY = await page.evaluate(() => window.scrollY);

// $eval / $$eval — selektor + funkcja, zwięzłe
const price = await page.$eval(".price", el => el.textContent.trim());
const links = await page.$$eval("a", els => els.map(el => ({
  text: el.textContent.trim(),
  href: el.href
})));
```

### Pliki (sandbox `~/.dev-browser/tmp/`)

```javascript
// Zapis
await writeFile("wyniki.json", JSON.stringify(data, null, 2));
await writeFile("raport.txt", "Hello world");

// Odczyt
const content = await readFile("wyniki.json");
const data = JSON.parse(content);
```

## Typowe workflow

### 1. Scraping listy danych

```bash
dev-browser --timeout 30000 <<'EOF'
const page = await browser.getPage("scraper");
await page.goto("https://example.com/products");
await page.waitForLoadState("networkidle");

const items = await page.$$eval(".product", els =>
  els.map(el => ({
    name: el.querySelector("h2")?.textContent?.trim(),
    price: el.querySelector(".price")?.textContent?.trim(),
    url: el.querySelector("a")?.href
  }))
);

await writeFile("produkty.json", JSON.stringify(items, null, 2));
console.log(`Zebrano ${items.length} produktów`);
EOF
```

### 2. Screenshot całej strony

```bash
dev-browser <<'EOF'
const page = await browser.getPage("shot");
await page.goto("https://example.com");
await page.waitForLoadState("networkidle");
const buf = await page.screenshot({ fullPage: true });
const path = await saveScreenshot(buf, "strona.png");
console.log("Screenshot zapisany:", path);
EOF
```

### 3. Wypełnianie formularza + logowanie

```bash
dev-browser --timeout 20000 <<'EOF'
const page = await browser.getPage("form");
await page.goto("https://example.com/login");

await page.getByRole("textbox", { name: "Email" }).fill("user@example.com");
await page.getByRole("textbox", { name: "Hasło" }).fill("haslo123");
await page.getByRole("button", { name: "Zaloguj" }).click();

await page.waitForURL("**/dashboard");
console.log("Zalogowano, URL:", page.url());
EOF
```

### 4. Praca z już zalogowaną Chrome użytkownika (`--connect`)

Dla serwisów wymagających logowania (LinkedIn, GitHub, CRM, banki) najlepiej podłączyć się do istniejącej sesji użytkownika — oszczędza hasła i działa z 2FA.

```bash
# Najpierw użytkownik uruchamia swoją Chrome z flagą debugowania:
#   chrome --remote-debugging-port=9222
# Potem:
dev-browser --connect <<'EOF'
const page = await browser.getPage("main");
await page.goto("https://www.linkedin.com/feed/");
// Jesteś już zalogowany — dziedziczysz sesję użytkownika
const posts = await page.$$eval(".feed-shared-update-v2", els =>
  els.slice(0, 5).map(el => ({
    text: el.textContent?.trim()?.substring(0, 200)
  }))
);
console.log(JSON.stringify(posts, null, 2));
EOF
```

### 5. Research potencjalnego klienta

```bash
dev-browser --timeout 30000 <<'EOF'
const page = await browser.getPage("research");
await page.goto("https://firma-klienta.pl");
await page.waitForLoadState("networkidle");

const { full } = await page.snapshotForAI({ depth: 2 });

// Wyciąganie kluczowych sekcji
const aboutText = await page.textContent("body");
const heroText = await page.textContent("h1, h2").catch(() => "");

await writeFile("research.txt", `URL: ${page.url()}\n\nHero: ${heroText}\n\nSnapshot:\n${full}`);
console.log("Research zapisany do research.txt");
EOF
```

### 6. Analiza struktury nieznanej strony

```bash
dev-browser <<'EOF'
const page = await browser.getPage("analyze");
await page.goto("https://strona-ktorej-nie-znam.com");
await page.waitForLoadState("networkidle");

const { full } = await page.snapshotForAI();
console.log(full);
// Teraz widzisz wszystkie interaktywne elementy — możesz napisać kolejny skrypt
// używając właściwych selektorów
EOF
```

## Error handling i debugging

### Wzorzec ochronny dla produkcyjnych skryptów

```bash
dev-browser --timeout 20000 <<'EOF'
const page = await browser.getPage("safe");

try {
  await page.goto("https://example.com");
  await page.waitForSelector(".main-content", { timeout: 10000 });

  const data = await page.$eval(".main-content", el => el.textContent.trim());
  console.log(JSON.stringify({ ok: true, data }));
} catch (e) {
  // Gdy coś pada — zrób screenshot stanu + dump informacji
  const buf = await page.screenshot({ fullPage: true });
  const path = await saveScreenshot(buf, `error-${Date.now()}.png`);
  console.log(JSON.stringify({
    ok: false,
    error: e.message,
    url: page.url(),
    screenshot: path
  }));
}
EOF
```

### Typowe problemy i ich rozwiązania

| Problem | Przyczyna | Rozwiązanie |
|---|---|---|
| "Element not found" | Zła struktura DOM lub timing | Uruchom `snapshotForAI()` i sprawdź co faktycznie jest na stronie |
| Skrypt wisi w nieskończoność | Brak timeoutu, oczekiwanie na coś czego nie ma | Dodaj `--timeout 10000` |
| "Navigation timeout" | Strona ładuje się za wolno lub zwraca błąd | `page.goto(url, { waitUntil: "domcontentloaded" })` zamiast `networkidle` |
| Logowanie nie działa | Serwis wymaga CAPTCHA lub 2FA | Użyj `--connect` — podłącz się do swojej już zalogowanej Chrome |
| Cookies znikają między skryptami | Używasz `newPage()` zamiast `getPage("nazwa")` | Zmień na named page |

### Inspekcja stanu named page po błędzie

Gdy named page zostaje w złym stanie — możesz ją zdebugować bez utraty kontekstu:

```bash
dev-browser <<'EOF'
const page = await browser.getPage("checkout");  // ta sama nazwa co przedtem
const buf = await page.screenshot({ fullPage: true });
const path = await saveScreenshot(buf, "debug.png");
console.log(JSON.stringify({
  url: page.url(),
  title: await page.title(),
  screenshot: path
}, null, 2));
EOF
```

## Zasady (priorytet malejący)

1. **Named pages dla persystencji, anon dla jednorazówki** — stan sesji = `getPage("nazwa")`, jednorazowa operacja = `newPage()`.
2. **Zawsze dawaj `--timeout`** — domyślny brak limitu = potencjalne zawieszenie na nieskończoność.
3. **`snapshotForAI()` przed zgadywaniem selektorów** — taniej w tokenach, szybciej, bardziej deterministycznie.
4. **`getByRole` > `locator({hasText})` > CSS** — preferuj accessibility selectors, są odporne na zmiany.
5. **`--connect` dla zalogowanych serwisów** — nie wklejaj haseł do skryptów, podłączaj się do istniejącej sesji użytkownika.
6. **`console.log(JSON.stringify(...))` dla wyników** — structured output = łatwe parsowanie w kolejnych krokach.
7. **Try/catch + screenshot w produkcji** — gdy skrypt pada, chcesz wiedzieć co widziała przeglądarka w momencie błędu.
8. **Pliki I/O tylko w `~/.dev-browser/tmp/`** — sandbox ograniczony, nie próbuj czytać spoza.

---

**TL;DR:** CLI do sterowania prawdziwą przeglądarką z persystentnym stanem sesji. Nawigacja, klikanie, formularze, scraping, screenshoty, snapshoty DOM dla AI. Named pages trzymają cookies i login między skryptami. `--connect` do istniejącej Chrome = zero problemów z hasłami i 2FA. Użyj `snapshotForAI()` gdy nie wiesz co jest na stronie, `screenshot()` gdy liczy się wygląd. Zawsze dawaj `--timeout`. 🌐