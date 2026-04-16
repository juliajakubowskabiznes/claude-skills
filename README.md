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

Wirtualna rada doradcza **4-12 person które wybierasz sam** debatująca Twoją decyzję przez 3 rundy. Każdy agent czyta swój profil, daje nieocenzurowaną opinię, ściera się z innymi, potem formułuje rekomendację. Output: jeden plik debaty z syntezą, która zachowuje realne niezgody.

📂 [`advisor-board/SKILL.md`](./advisor-board/SKILL.md)

**Pierwsze uruchomienie** przeprowadza one-time onboarding: pyta o Twoje imię, ile osób w radzie (rekomendacja: 4), KOGO konkretnie chcesz na pokładzie (możesz wpisać dowolnych ludzi — przedsiębiorcy, myśliciele, sportowcy, postacie fikcyjne, mentorzy — Claude sam zbuduje ich profile z wiedzy publicznej), opcjonalnie ścieżki do plików kontekstowych. Potem skill podmienia placeholdery `{{USER_NAME}}` → Twoje imię i zapisuje config.

> **Wskazówka przy wyborze rady:** dobierz osoby z **różnymi światopoglądami**, nie 4 wariantów tego samego. Rada 4 founderów mówiących "ship faster" jest bezużyteczna — rada ekonomista + długoterm + BS-caller + values-questioner daje realne tarcie.

**Koszt per run (Sonnet):**
| Advisors | Tokeny | Koszt |
|---|---|---|
| 4 (default) | ~60K | ~$0.30 |
| 6 | ~90K | ~$0.45 |
| 8 | ~120K | ~$0.60 |
| 12 (max) | ~175K | ~$1.00 |

> Przy 12 advisorach debata zbliża się do limitu 200K okna kontekstu Claude Pro — możesz uderzyć w limit w środku debaty. Bezpiecznie trzymać 4-8.

**Instalacja:**
1. Skopiuj cały folder `advisor-board/` (SKILL.md + references/)
2. Wklej do `~/.claude/skills/advisor-board/` w swoim projekcie
3. Zrestartuj Claude Code
4. Wywołaj `/advisor-board [pytanie]` lub powiedz *„convene the board"*

**Użycie:**
```
/advisor-board Czy zamknąć stary produkt i postawić wszystko na nowy?
```
Claude frame'uje pytanie, dispatcha N subagentów równolegle × 3 rundy, zwraca plik debaty z syntezą.

---

## Licencja

MIT — używaj, modyfikuj, dziel się.
