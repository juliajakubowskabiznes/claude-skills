# Claude Skills — Julia Jakubowska

Kolekcja skilli dla [Claude Code](https://docs.claude.com/en/docs/claude-code/overview) używanych w mentoringu i codziennej pracy z AI.

## Dostępne skille

### 🌐 dev-browser

Sterowanie prawdziwą przeglądarką z persystentnym stanem sesji (cookies, logowanie). Do scrapingu, research klientów, monitoringu konkurencji, LinkedIn analytics, ekstrakcji danych z narzędzi bez API.

📂 [`dev-browser/SKILL.md`](./dev-browser/SKILL.md)

**Wymagania:**
- [dev-browser](https://github.com/SawyerHood/dev-browser) (by Sawyer Hood) — zainstalowany globalnie:
  ```bash
  npm install -g dev-browser
  dev-browser install
  ```

**Instalacja skilla:**
1. Skopiuj zawartość [`dev-browser/SKILL.md`](./dev-browser/SKILL.md)
2. Wklej do `~/.claude/skills/dev-browser/SKILL.md` w swoim projekcie
3. Zrestartuj Claude Code
4. Mów do Claude naturalnymi zdaniami: *„wejdź na stronę X", „zrób research firmy Y", „sprawdź mój LinkedIn"*

---

### 🎭 advisor-board

Wirtualna rada doradcza 12 person (Hormozi, Goggins, Naval, Taleb, Taylor Swift, Bryan Johnson, Mark Manson, James Clear, Daniel Priestley, Nick Saraev, Leon Hendrix, Rian Doris) debatująca Twoją decyzję przez 3 rundy. Każdy agent czyta swój profil, daje nieocenzurowaną opinię, ściera się z innymi, potem formułuje rekomendację. Output: jeden plik debaty z syntezą, która zachowuje realne niezgody.

📂 [`advisor-board/SKILL.md`](./advisor-board/SKILL.md)

**Pierwsze uruchomienie** przeprowadza one-time onboarding: pyta o imię, pozwala zostawić domyślną radę lub podać własnych 12 advisorów, opcjonalnie podpinasz ścieżki do swoich plików kontekstowych. Potem skill sam podmienia placeholdery `{{USER_NAME}}` → Twoje imię i zapisuje config.

**Instalacja:**
1. Skopiuj cały folder `advisor-board/` (SKILL.md + references/)
2. Wklej do `~/.claude/skills/advisor-board/` w swoim projekcie
3. Zrestartuj Claude Code
4. Wywołaj `/advisor-board [pytanie]` lub powiedz *„convene the board"*

**Użycie:**
```
/advisor-board Czy zamknąć stary produkt i postawić wszystko na nowy?
```
Claude frame'uje pytanie, dispatcha 12 subagentów równolegle × 3 rundy, zwraca plik debaty z syntezą.

---

## Licencja

MIT — używaj, modyfikuj, dziel się.
