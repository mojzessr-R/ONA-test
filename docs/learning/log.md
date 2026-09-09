# Dziennik nauki

Jeden wpis na etap: co powstało, czego uczy, co zapamiętać.

---

## ▶ Gdzie jesteśmy (notatka przekazania)

**Aktualizowano:** 2026-09-09, po odtworzeniu środowiska.
Ta sekcja jest po to, żeby nowa sesja agenta — bez historii czatu — wiedziała, co robić dalej.
Aktualizuj ją na koniec każdego etapu.

**Stan:** Etap 0 ukończony. Etap 1 **w połowie** — istnieje C4 poziom 1
(`docs/architecture/c4-01-context.md`, commit `a385880`). Brak katalogu `adr/` i poziomu 2.

**Następny krok:** dokończyć Etap 1 — ADR-y `0001`–`0007` + `c4-02-container.md`
(poziom 1 już do niego linkuje). Zero kodu produkcyjnego. Spisujemy decyzje, które już zapadły,
razem z odrzuconymi alternatywami. Planowany zakres ADR-ów:

| ADR | Decyzja |
|---|---|
| 0001 | Stos: Python 3.12 + `.venv` + pytest (odrzucone: HTML+JS, Excel) |
| 0002 | Parametry podatkowe jako dane z proweniencją; `TODO-VERIFY` blokuje raport |
| 0003 | Forma opodatkowania jako strategia; stawka jako parametr, nie tożsamość |
| 0004 | `Decimal` zamiast `float` + reguły zaokrągleń |
| 0005 | Czysty rdzeń, cienkie UI; CLI jako pierwsze UI |
| 0006 | Kształt `Offer` — **wymaga decyzji właściciela**, patrz niżej |
| 0007 | C4 na `flowchart`, nie na eksperymentalnym `C4Context` |

**Rozstrzygnięte 2026-09-09 (nie pytaj o to ponownie):** Mermaid renderuje się poprawnie
**i** na GitHubie, **i** w podglądzie edytora w przeglądarce. Awaria z poprzedniej sesji była
chwilowa — rozszerzenie nie wstało w tamtym kontenerze i naprawił to dopiero reset.
Diagram w `c4-01-context.md` jest składniowo poprawny.

**Nadal czeka na odpowiedź właściciela repo:**
1. Czy `docs/learning/00-glossary.md` jest zrozumiały — co doprecyzować?
2. Czy reguły w `CLAUDE.md` są zaakceptowane, zwłaszcza „`src/` jest czysty"
   i „parametry podatkowe to dane, nie kod"?

**Zaparkowane do rozstrzygnięcia w Etapie 1 (kandydat na ADR 0006):**
czy `Offer` ma trzymać **listę strumieni przychodu** (różne stawki ryczałtu dla różnych usług),
czy wystarczy jedna stawka. Rekomendacja: lista od razu, typowy przypadek jednoelementowy —
dorabianie tego później oznacza przeróbkę centralnej dataclassy i wszystkich testów.

**GitHub:** `https://github.com/mojzessr-R/ONA-test`, gałąź `main`. Od 2026-09-09 obowiązuje
zasada: commit kończący etap jest natychmiast pushowany. Jeśli `git status` pokazuje
„ahead of origin/main", coś poszło nie tak — wypchnij.

**Jak wznowić rozmowę:** `cd /workspaces/ONA-test && claude --continue`.
Historia czatu żyje w `~/.claude/projects/-workspaces-ONA-test/` i **nie jest w repozytorium** —
jeśli workspace zostanie zresetowany, przetrwa tylko to, co jest w commitach. Dlatego wnioski
lądują tutaj, a nie w czacie. **To już się zdarzyło** (2026-09-09) — patrz wpis na końcu pliku.
Nie licz na `--continue`. Ta sekcja jest jedynym niezawodnym punktem wejścia.

---

## Plan etapów

| Etap | Temat | Pojęcia | Status |
|---|---|---|---|
| 0 | Fundament środowiska i pojęć | harness, kontekst, `CLAUDE.md`, uprawnienia, plan mode | ✅ |
| 1 | Szkielet architecture-as-code | ADR/MADR, C4, Mermaid | ⬜ |
| 2 | Parametry podatkowe jako dane | separacja polityki od logiki, fail-fast, proweniencja | ⬜ |
| 3 | Silnik finansowy w TDD | TDD z agentem, test jako specyfikacja, `/code-review` | ⬜ |
| 4 | Porównanie, break-even, CLI | CLI jako pierwsze UI | ⬜ |
| 5 | Analiza niefinansowa | scoring ważony, analiza wrażliwości | ⬜ |
| 6 | Forecast wieloletni | scenariusze, założenia jako dane, NPV | ⬜ |
| 7 | UI kalkulatora | cienkie UI nad czystym silnikiem | ⬜ |
| 8 | Automatyzacja harnessu | skill, slash command, hook, subagent, MCP | ⬜ |
| 9 | Decyzja i retrospektywa | domknięcie, transfer do projektów SAP | ⬜ |

---

## Etap 0 — Fundament środowiska i pojęć

**Data:** 2026-09-08

### Co powstało

| Plik | Rola |
|---|---|
| `.venv/` + `requirements.txt` | Python 3.12.3, pyyaml, pytest, streamlit |
| `CLAUDE.md` | konstytucja projektu — kontekst wczytywany przy każdej sesji agenta |
| `.claude/settings.json` | uprawnienia: co agent może zrobić bez pytania |
| `docs/learning/00-glossary.md` | słownik pojęć agentic development |
| `README.md`, `.gitignore` | wejście do projektu, higiena repo |
| `Context/README.md` | granica: co jest faktem z zewnątrz, a co naszym wnioskiem |

### Czego uczy

**1. Harness ≠ model.** Model to funkcja tekst→tekst: nie widzi plików, nie pamięta rozmowy,
nie wykonuje niczego. Wszystko, co „potrafi" agent, robi harness. Konfigurujesz harness,
nie model — i tu leży większość Twojego wpływu na jakość pracy.

**2. Plan mode zapłacił za siebie w pierwszej sesji.** Zanim cokolwiek powstało, rozpoznanie
środowiska wykryło dwie rzeczy, które unieważniłyby domyślne podejście:

- w kontenerze **nie było żadnego runtime'u** — ani Pythona, ani Node (tylko `git`, `curl`, `jq`);
- **WebSearch i WebFetch są zablokowane** przez politykę sieci (`VPCSC`).

Bez tego rozpoznania powstałby kalkulator w HTML+JS (bo nie było Pythona) wypełniony zmyślonymi
stawkami na 2026 (bo nie dało się ich sprawdzić) — działający i wewnętrznie spójny, a bezużyteczny.
Nawyk: **przy nietrywialnym zadaniu zaczynaj od plan mode.**

**3. Ograniczenie środowiska stało się decyzją architektoniczną.** Brak dostępu do internetu
oznacza, że agent nie zweryfikuje stawek podatkowych. Zamiast to obchodzić, wpisaliśmy to
w architekturę: każdy parametr niesie `source` i `verified_on`, a niepotwierdzone wartości
noszą `TODO-VERIFY` i blokują tryb raportu (Etap 2).

To jest wzorzec przenośny wprost do projektów SAP: **dane oparte o zmienne prawo lub politykę
biznesową muszą nieść swoją proweniencję.** Zahardkodowana stawka VAT w transformacji jest
dokładnie tym samym błędem, co zahardkodowany ryczałt 12% w `.py`.

**4. `CLAUDE.md` to nie README dla ludzi.** To pamięć trwała agenta. Wszystko, co powiesz
w czacie, zniknie za trzy sesje; co zapiszesz w `CLAUDE.md`, wróci w każdej. Reguła praktyczna:
jeśli poprawiasz agenta drugi raz w tej samej sprawie, to nie jest wina agenta — to brakujący
wpis w `CLAUDE.md`.

**5. Uprawnienia to projekt piaskownicy, nie irytujące pop-upy.** Pozwalamy hurtowo na rzeczy
odwracalne i tylko-odczytujące (`pytest`, `git diff`, `git log`), blokujemy `git push` i `rm -rf`.
Cel: agent pracuje płynnie w granicach, które Ty wyznaczyłeś świadomie i raz.

### Do zapamiętania

> Konfigurowanie harnessu jest tą samą czynnością, co projektowanie architektury:
> ustalasz granice, w których praca może toczyć się bezpiecznie i szybko.

### Dopisane po etapie

`docs/learning/01-git-for-sap-architects.md` — powstało na pytanie „o co chodzi z branchami
i commitowaniem, w SAPie nie ma takich rzeczy". Okazuje się, że są: transport request to commit,
release TR to push, drugi landscape to branch, retrofit to merge. Git nie wprowadza nowych pojęć,
tylko tanie wersje tych, które już znasz. Notatka wyjaśnia też, dlaczego commit per etap jest
fundamentem architecture-as-code: ADR i kod w jednym commicie nie mogą się rozjechać.

### Otwarte / przeniesione dalej

- `docs/learning/02-how-claude-code-works.md` — świadomie odłożone do Etapu 8, gdzie skille,
  slash commands i hooki powstaną naprawdę. Opisywanie ich wcześniej byłoby teorią bez artefaktu.
- Parametry podatkowe na 2026 wymagają Twojej weryfikacji ze źródłem — temat Etapu 2.

---

## Korekta zakresu — stawka ryczałtu 8,5%, nie 12%

**Data:** 2026-09-08, po Etapie 0, przed Etapem 1.

Właściciel repo, czytając `CLAUDE.md`, wychwycił błąd: interesuje nas **ryczałt 8,5%**
(usługi wsparcia IT), nie 12%. Dodatkowo zgłosił wymaganie: możliwość **symulacji innych stawek**,
bo mogą się zmienić.

### Czego ta korekta uczy — trzy rzeczy

**1. Architektura zarobiła na siebie, zanim powstała pierwsza linia kodu.** Gdyby stawka siedziała
w kodzie, ta uwaga oznaczałaby refaktor. Ponieważ reguła 2 (`CLAUDE.md`) mówi „parametry to dane",
zmiana sprowadza się do jednej wartości w YAML-u w Etapie 2. **Wartość dobrej decyzji
architektonicznej ujawnia się przy pierwszej zmianie wymagań, nie przy pierwszym uruchomieniu.**

**2. Abstrakcja przeciekła o jeden poziom wyżej — do nazwy pliku.** W planie był
`taxation/b2b_lumpsum12.py`. Stawka w nazwie pliku to dokładnie ten sam błąd, co stawka
zahardkodowana w ciele funkcji, tylko trudniejszy do zauważenia w review. Poprawione na
`b2b_lumpsum.py` — **jedna** strategia „ryczałt", stawka wstrzykiwana z parametrów.
Reguła ogólna: *stawka nie należy do tożsamości strategii*.

Wniosek przenośny do SAP: jeśli nazywasz obiekt `Z_CALC_VAT_23`, właśnie zahardkodowałeś stawkę
w nazwie i za dwa lata będziesz miał `Z_CALC_VAT_23_NEW`.

**3. Nowe wymaganie zmieniło kształt produktu, nie tylko liczbę.** „Symulacja innych stawek"
oznacza, że stawka przestaje być parametrem konfiguracji, a staje się **wymiarem analizy** —
kalkulator ma liczyć porównanie dla zestawu stawek naraz i pokazywać wrażliwość wyniku.
To wchodzi do zakresu Etapów 4 i 6.

### Ryzyko dopisane do modelu

Granica między „wsparciem IT" (8,5% — stawka resztkowa dla działalności usługowej) a usługami
związanymi z oprogramowaniem (PKWiU 62.01.1 / 62.02 / 62.03.1 → 12%) bywa rozstrzygana
interpretacją indywidualną. Kalkulator ma pokazywać koszt scenariusza, w którym US kwestionuje
8,5% i przeklasyfikowuje przychód na 12% wstecz.

Mapowania PKWiU: **`TODO-VERIFY`** — brak dostępu do internetu, nie zgadujemy.

---

## Reset workspace'u — środowisko jako kod

**Data:** 2026-09-09, w trakcie Etapu 1.

Sesja zaczęła się od „wracamy do tego, co robiliśmy". Rozpoznanie wykazało, że **workspace został
zresetowany**: brak `.venv/`, brak `python3`, katalog z historią czatu utworzony tego samego dnia
o 13:46. Poprzednia rozmowa nie istnieje.

### Bilans: co przetrwało, a co nie

| Przetrwało | Zniknęło |
|---|---|
| Wszystkie commity, w tym `c4-01-context.md` | Historia czatu z poprzedniej sesji |
| `CLAUDE.md`, `log.md`, słownik | `.venv/` i cały runtime Pythona |
| Treści commitów — z nich odtworzono kontekst | Wiedza „co dokładnie wpisano w terminal" |

Zasada „po commicie kończącym etap zrób `git push`" (`4fca634`, dodana dzień wcześniej) uratowała
projekt. Nie w sensie retorycznym — po prostu wszystko, czego nie było w commicie, przepadło.

### Czego to uczy — trzy rzeczy

**1. W projekcie agentowym wiedza mieszka w artefaktach, nie w kontekście modelu.** Kontekst jest
najbardziej ulotnym nośnikiem w całym stosie — ulotniejszym niż katalog roboczy. `log.md` okazał
się jedynym mostem między sesjami. Notatka przekazania na górze tego pliku nie jest ozdobnikiem,
tylko interfejsem: *tak nowa sesja dowiaduje się, co robić.* Warto ją pisać dla obcego.

**2. Treść commita to dokumentacja, nie formalność.** Stan Mermaida odtworzono wyłącznie
z opisu commita `a385880` („the VS Code preview extension refused to activate in the browser
build"). Gdyby ten commit nosił opis „update docs", ta informacja byłaby nie do odzyskania.
**Commit opisuje decyzję i jej powód, nie listę zmienionych plików** — tę git pokazuje sam.

**3. Environment as code — siostra architecture as code.** Etap 0 zbudował środowisko komendami
w czacie. Zadziałało i nie przetrwało, bo `.venv/` jest w `.gitignore`, a instrukcja nie była
nigdzie zapisana. Lekarstwo to nie backup, tylko przeniesienie instrukcji do repo:

```jsonc
// .devcontainer/devcontainer.json
"postCreateCommand": "sudo apt-get update -qq && sudo apt-get install -y -qq python3 python3-venv && ..."
```

> Jeśli odtworzenie środowiska wymaga pamiętania, co się wpisało w terminal, to środowisko nie
> jest opisane — tylko zapamiętane. Deklaratywnie opisane środowisko zamienia reset z katastrofy
> w kilka minut oczekiwania.

Odpowiednik w SAP: różnica między systemem postawionym według dokumentu instalacyjnego a takim,
który „skonfigurował kiedyś Marek". Drugi działa dokładnie do momentu, w którym trzeba go odtworzyć.

### Przy okazji: różnica między awarią a brakującym narzędziem

Poprzednia sesja uznała podgląd Mermaida za trwale zepsuty. W nowym kontenerze działa —
rozszerzenie po prostu wstało. Awaria była chwilowa, ale została zapisana jako właściwość
środowiska i zablokowała pracę.

Warto pamiętać też, dlaczego GitHub renderował diagram od początku, a edytor nie: GitHub ma
Mermaida **wbudowanego**, VS Code potrzebuje **rozszerzenia**. To nie były dwie próby tego samego,
tylko dwa różne silniki. Nawyk: **zanim uznasz artefakt za wadliwy, sprawdź, czy nie patrzysz
na niego przez zepsutą szybę** — i czy szyba nie jest po prostu nieszybą.
