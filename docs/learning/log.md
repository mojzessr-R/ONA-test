# Dziennik nauki

Jeden wpis na etap: co powstało, czego uczy, co zapamiętać.

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
