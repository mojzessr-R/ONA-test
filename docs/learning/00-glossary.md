# Słownik pojęć agentic development

Wszystkie przykłady pochodzą z tego repozytorium — nie z abstrakcyjnej dokumentacji.
Kolejność jest dydaktyczna, nie alfabetyczna: od fundamentów do rzeczy zaawansowanych.

---

## 1. Fundamenty

### Model (LLM)
Sama sieć neuronowa — funkcja, która dostaje tekst i zwraca tekst. Nic więcej. Model **nie ma**
dostępu do plików, terminala ani internetu. Nie pamięta poprzedniej rozmowy. Nie potrafi
„wykonać" niczego.

W tej sesji model to `claude-opus-5[1m]`.

### Harness
Program, który **opakowuje** model i daje mu ręce. To harness czyta pliki, uruchamia komendy,
pilnuje uprawnień, składa prompt, pętli się po odpowiedziach modelu i wykonuje to, o co model
poprosi. Claude Code, Cursor, Copilot Agent — to wszystko harnessy nad tym samym typem modelu.

> **To jest rozróżnienie, od którego wszystko zależy.** Kiedy narzekasz, że „AI nie umie czegoś
> zrobić", w 80% przypadków problem leży w harnessie i jego konfiguracji, nie w modelu.
> Ty konfigurujesz harness. Modelu nie zmienisz.

Konkretnie w tym repo harnessem sterują pliki: `CLAUDE.md` i `.claude/`.

### Kontekst / okno kontekstu (context window)
Cały tekst, który model widzi w jednym wywołaniu: instrukcje systemowe, `CLAUDE.md`, historia
rozmowy, treść wczytanych plików, wyniki komend. Ma skończony rozmiar, liczony w **tokenach**
(token ≈ 3–4 znaki polskiego tekstu). Ta sesja ma okno 1 mln tokenów.

Konsekwencje praktyczne:
- Wczytanie dużego pliku „zjada" okno. Dlatego agent czyta fragmenty, nie wszystko.
- Gdy okno się zapełnia, starsza część rozmowy jest **streszczana** — stąd wrażenie, że agent
  „zapomniał". Nie zapomniał; został podsumowany.
- `CLAUDE.md` jest wstrzykiwany na nowo w każdej sesji — dlatego trwałe zasady piszemy tam,
  a nie w czacie.

### Prompt systemowy
Instrukcje wstrzykiwane przed Twoją wiadomością, których nie piszesz i zwykle nie widzisz.
Definiują zachowanie agenta. `CLAUDE.md` doklejasz do niego Ty — to Twój sposób na kształtowanie
zachowania agenta bez dotykania modelu.

---

## 2. Jak agent działa

### Tool (narzędzie)
Konkretna zdolność wystawiona modelowi przez harness, opisana schematem. W Claude Code m.in.:
`Read`, `Write`, `Edit`, `Bash`, `Grep`, `Glob`, `WebSearch`, `Task`.

Model nie „uruchamia" narzędzia — model **prosi** o jego uruchomienie, generując ustrukturyzowane
wywołanie. Wykonuje je harness i wkleja wynik z powrotem do kontekstu.

Przykład z tej sesji: sprawdzenie, że w kontenerze nie ma Pythona, to było `Bash(command: "...")`
→ harness wykonał → wynik wrócił do modelu → model wyciągnął wniosek i zmienił plan.

### Agentic loop (pętla agentowa)
Rdzeń całej koncepcji:

```
Twój cel
   ↓
[model myśli] → prosi o narzędzie → [harness wykonuje] → wynik wraca do kontekstu
   ↑                                                                  ↓
   └──────────────────── powtarzaj, aż cel osiągnięty ────────────────┘
```

### Line prompting vs agentic development
To jest różnica, dla której robisz ten projekt:

| | Line prompting | Agentic development |
|---|---|---|
| Jednostka pracy | jedno pytanie → jedna odpowiedź | cel → wieloetapowa realizacja |
| Kto sprawdza wynik | Ty, ręcznie, po fakcie | agent, testami, w pętli |
| Stan | w Twojej głowie i schowku | w repozytorium |
| Kontekst | wklejasz za każdym razem | `CLAUDE.md` + pliki repo |
| Weryfikowalność | „wygląda dobrze" | `pytest` przechodzi albo nie |
| Skala | jeden plik | cały projekt |

Kluczowa zmiana nawyku: **przestajesz opisywać rozwiązanie, zaczynasz opisywać cel i kryterium
akceptacji.** Test jest lepszą specyfikacją niż akapit promptu, bo jest wykonywalny.

### Agent
Model + harness + narzędzia + cel, działające w pętli. Tym rozmawiasz teraz.

### Subagent
Osobny agent uruchomiony przez agenta głównego, z **własnym, czystym kontekstem**. Dostaje zadanie,
wykonuje je i zwraca sam wniosek — nie zaśmiecając Twojego okna kontekstu setkami linii wyników.

Do czego: „przeszukaj 200 plików i powiedz gdzie jest X" — chcesz odpowiedź, nie 200 plików
w kontekście. W Claude Code są gotowe typy: `Explore` (wyszukiwanie), `Plan` (projektowanie).

> Cena: subagent nie widzi Twojej rozmowy. Musisz mu przekazać kontekst w zadaniu.

---

## 3. Konfiguracja harnessu

Cztery mechanizmy, które łatwo pomylić. Rozróżnienie: **kto to uruchamia i kiedy.**

| Mechanizm | Kto uruchamia | Kiedy | Plik |
|---|---|---|---|
| `CLAUDE.md` | harness | zawsze, każda sesja | `CLAUDE.md` |
| **Skill** | agent, sam z siebie | gdy uzna, że pasuje do zadania | `.claude/skills/*/SKILL.md` |
| **Slash command** | Ty, ręcznie | gdy wpiszesz `/nazwa` | `.claude/commands/*.md` |
| **Hook** | harness, automatycznie | na zdarzenie (np. po edycji pliku) | `.claude/settings.json` |

### Skill
Spakowana **wiedza proceduralna** — „jak się u nas robi X". Ma opis; agent czyta opisy wszystkich
skilli i **sam decyduje**, że dany skill pasuje do zadania, po czym wczytuje jego treść.

Dlaczego nie wrzucić wszystkiego do `CLAUDE.md`: bo `CLAUDE.md` jest w kontekście **zawsze** i
kosztuje tokeny przy każdej sesji. Skill kosztuje tylko wtedy, gdy jest potrzebny.

W tym projekcie (Etap 8): skill `tax-params` — procedura dodania nowego parametru podatkowego
wraz z wymogiem podania źródła.

### Slash command
Zapisany prompt, który uruchamiasz **Ty**, wpisując `/nazwa`. Deterministyczny skrót do
powtarzalnej czynności.

W tym projekcie (Etap 8): `/adr "tytuł"` → tworzy kolejny ADR z szablonu i poprawnym numerem.

> Skill vs command w jednym zdaniu: **skill to wiedza, którą agent bierze sam; command to przycisk,
> który naciskasz Ty.**

### Hook
Skrypt shellowy uruchamiany przez harness **automatycznie** na zdarzenie — np. `PostToolUse`
po każdej edycji pliku. Wykonuje go harness, nie model, więc jest w 100% deterministyczny.

To odpowiedź na życzenia typu „od teraz zawsze po zmianie kodu uruchom testy". Zapisanie tego
w `CLAUDE.md` daje ~90% skuteczności (model może pominąć). Hook daje 100%.

W tym projekcie (Etap 8): hook uruchamiający `pytest` po każdej edycji w `src/`.

### settings.json i uprawnienia (permissions)
`.claude/settings.json` mówi harnessowi, czego agent może użyć bez pytania Cię o zgodę.
Bez tego przy każdym `pytest` dostajesz pytanie „czy pozwolić?". Z tym — agent pracuje płynnie
w bezpiecznej piaskownicy, którą Ty wyznaczyłeś.

Zasada: pozwalaj hurtowo na rzeczy **odwracalne i tylko-odczytujące** (`pytest`, `git status`,
`git diff`), nigdy hurtowo na `git push`, `rm -rf` czy `sudo`.

### MCP (Model Context Protocol)
Otwarty standard podłączania agenta do **zewnętrznych systemów**. Serwer MCP wystawia narzędzia
i dane; harness je odkrywa i udostępnia modelowi jak własne.

Analogia z Twojego świata: **MCP jest dla narzędzi AI tym, czym ODBC/OData jest dla systemów
raportowych.** Jeden protokół zamiast N integracji per para (narzędzie, system).

Dlaczego to dla Ciebie ważne w perspektywie SAP: serwer MCP do SAP, Jiry, Confluence czy
katalogu danych oznacza, że agent czyta realny stan systemu, zamiast zgadywać z Twojego opisu.
Architecture-as-code przestaje wtedy być dokumentacją, a staje się kodem sprzężonym z rzeczywistością.

W tym PoC MCP **nie jest potrzebny** — wszystko jest w plikach. Wyjaśnimy go na przykładzie
w Etapie 8, żeby pojęcie nie zostało puste.

---

## 4. Tryby pracy

### Plan mode
Tryb, w którym agent może tylko czytać — nie zmieni ani jednego pliku. Służy do rozpoznania
i zaprojektowania rozwiązania, zakończonego planem do Twojej akceptacji.

Użyliśmy go przed chwilą: agent zbadał repo, wykrył brak Pythona i blokadę WebSearch, zapytał
Cię o cztery decyzje i dopiero potem napisał plan. Gdyby ruszył od razu, zbudowałby kalkulator
w HTML-u (bo nie było Pythona) i wpisałby zmyślone stawki 2026 (bo nie mógł ich sprawdzić).

> Nawyk do wyrobienia: **przy każdym nietrywialnym zadaniu zaczynaj od plan mode.** Koszt to jedna
> runda pytań. Zysk to uniknięcie pracy zrobionej dobrze, ale nie na temat.

### Tryby uprawnień
Od „pytaj o wszystko" po „nie pytaj o nic" (`--dangerously-skip-permissions`). Ten ostatni
wyłącznie w izolowanym kontenerze — czyli w takim, w jakim właśnie jesteśmy.

---

## 5. Architecture-as-code

### ADR (Architecture Decision Record)
Krótki dokument w Markdown: **kontekst → rozważane opcje → decyzja → konsekwencje**. Numerowany,
niemodyfikowalny po zatwierdzeniu (zmiana = nowy ADR ze statusem „supersedes 0004").

Wartość jest w rzeczy, której nie ma w kodzie: **dlaczego** i **czego świadomie nie wybraliśmy**.
Kod pokazuje, że używamy Pythona. Tylko ADR powie, że rozważaliśmy HTML+JS i dlaczego odpadł.

Używamy formatu **MADR** (Markdown ADR).

### C4
Model opisu architektury w czterech poziomach zoomu: **Context** (system i jego otoczenie),
**Container** (uruchamialne jednostki), **Component**, **Code**. W praktyce robi się poziom 1 i 2.

Sens: jeden diagram nie może służyć jednocześnie zarządowi i deweloperowi. C4 wymusza świadomy
wybór poziomu szczegółowości dla odbiorcy.

### Mermaid
Język tekstowego opisu diagramów, renderowany natywnie przez VS Code i GitHub. Diagram jest
**tekstem w gicie**: widać go w `git diff`, można go zrewiewować w PR, nie da się go zgubić
w załączniku do maila.

### Architecture-as-code
Zasada nadrzędna: **architektura mieszka w repozytorium, obok kodu, i podlega tym samym rygorom** —
wersjonowaniu, review, CI. Decyzja (ADR), model (C4) i realizacja (kod) trafiają do jednego commita,
więc nie mogą się rozjechać.

To jest dokładnie to, czego szukasz do projektów SAP: koniec z rozjazdem między slajdem
architektury a tym, co faktycznie stoi w systemie.

---

## Rzeczy, których pojęcia tu nie ma, a mogą Cię zmylić

- **RAG** — doklejanie wyszukanych fragmentów dokumentów do promptu. Nie używamy: repo jest małe,
  agent czyta pliki bezpośrednio.
- **Fine-tuning** — dotrenowanie modelu. Prawie nigdy nie jest odpowiedzią; kontekst i narzędzia są.
- **Temperature / top_p** — parametry losowości generowania. W pracy agentowej praktycznie ich
  nie dotykasz.
