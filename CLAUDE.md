# CLAUDE.md — konstytucja projektu

Ten plik jest wczytywany do kontekstu agenta przy **każdej** sesji w tym repozytorium.
Zawiera to, czego agent nie wywnioskuje z kodu.

## Czym jest ten projekt

PoC „UoP vs B2B" — narzędzie i analiza wspierające decyzję, która forma współpracy jest
korzystniejsza dla konkretnej pary ofert. Projekt ma dwa równoległe cele:

1. **Merytoryczny** — kalkulator + analiza finansowa, niefinansowa i forecast wieloletni.
2. **Edukacyjny (nadrzędny)** — nauka agentic development i architecture-as-code. Właściciel repo
   przechodzi z line-promptingu na pracę agentową. Każdy etap ma uczyć konkretnego pojęcia.

Zakres podatkowy PoC: **UoP na skali** vs **B2B na ryczałcie**, Polska, rok bazowy 2026.

Stawka bazowa ryczałtu: **8,5%** (usługi wsparcia IT — stawka resztkowa dla działalności
usługowej). Kalkulator musi jednak umożliwiać **symulację dowolnej stawki** — co najmniej
8,5%, 12% i 15% — bo klasyfikacja PKWiU bywa sporna, a stawki zmieniają się w czasie.
Stawka jest **wejściem do symulacji**, nie stałą projektu.

## Konwencja językowa

| Co | Język |
|---|---|
| Dokumentacja (`docs/`), ADR, README, UI kalkulatora | **polski** |
| Kod, nazwy plików, katalogów, zmiennych, funkcji | **angielski** |
| Komunikaty commitów | **angielski** |
| Rozmowa z użytkownikiem | **polski** |

## Zasady architektoniczne (nienegocjowalne bez nowego ADR)

1. **`src/uop_b2b/` jest czysty.** Zero I/O, zero sieci, zero `print`, zero zależności od UI.
   Wejście: dataclasses. Wyjście: dataclasses. Dzięki temu jest testowalny i przeżyje zmianę UI.
2. **Parametry podatkowe to dane, nie kod.** Każda stawka, próg i limit żyje w
   `data/params/pl-2026.yaml` wraz z polami `source` i `verified_on`. Liczba podatkowa
   zahardkodowana w `.py` to błąd, nawet jeśli jest poprawna.
3. **Forma opodatkowania to strategia, nie `if`.** Nowa forma = nowy plik w `src/uop_b2b/taxation/`
   implementujący protokół z `base.py`. Żadnych rozgałęzień po nazwie formy w `compare.py`.
   **Stawka nie jest częścią tożsamości strategii.** Jest jedna strategia `b2b_lumpsum.py`
   („ryczałt"), która przyjmuje stawkę jako parametr — nie `b2b_lumpsum12.py` i `b2b_lumpsum85.py`.
   Stawka w nazwie pliku, klasy lub funkcji to ten sam błąd, co stawka zahardkodowana w kodzie.
4. **Każda decyzja architektoniczna ma ADR** w `docs/architecture/adr/`. Kod i decyzja lądują
   w tym samym commicie.
5. **UI jest cienkie.** `app/` i `cli.py` nie zawierają ani jednego wzoru podatkowego.

## Zasady pracy (ważne dla Ciebie, agencie)

- **Nigdy nie zmyślaj wartości podatkowych.** Nie masz dostępu do internetu w tym środowisku
  (WebSearch/WebFetch są zablokowane przez politykę sieci — `VPCSC`). Jeśli nie znasz stawki
  z pewnością, wpisz `TODO-VERIFY` i poproś użytkownika o potwierdzenie ze źródłem.
  Wiarygodnie brzmiąca zmyślona liczba jest tu najgorszym możliwym błędem.
- **TDD.** Dla logiki obliczeniowej najpierw test z ręcznie policzoną wartością oczekiwaną
  (podaje ją użytkownik), dopiero potem implementacja.
- **Pieniądze liczymy na `Decimal`**, nie na `float`. Zaokrąglenia zgodnie z regułami
  udokumentowanymi w `docs/analysis/02-financial-model.md`.
- **Krok po kroku.** Użytkownik uczy się. Po każdym etapie zatrzymaj się, wyjaśnij co powstało
  i czego to uczy, dopisz wpis do `docs/learning/log.md`. Nie wybiegaj do przodu.
- **Wyjaśniaj pojęcia w miejscu ich pierwszego użycia** i linkuj do `docs/learning/00-glossary.md`.
- **Git: commituj prosto na `main`**, jeden commit na etap, bez gałęzi roboczych. To repo
  jednoosobowe i edukacyjne — liniowa historia czyta się jak spis treści projektu.
  ADR i kod, który go realizuje, muszą trafić do **tego samego commita**.
- **Po commicie kończącym etap wykonaj `git push`.** Praca nie może zostać wyłącznie na kontenerze —
  workspace bywa resetowany, a wtedy niewypchnięte commity przepadają. `git push --force`
  jest zablokowany i pozostaje zablokowany: nadpisuje historię na GitHubie.

## Środowisko

- Runtime instalowany ręcznie (obraz bazowy jest pusty): Python 3.12 w `.venv/`.
- **Zawsze używaj `.venv/bin/python` i `.venv/bin/pytest`**, nigdy gołego `python3`.
- Brak dostępu do internetu z narzędzi agenta. `pip install` działa (przez proxy), WebSearch nie.

## Polecenia

```bash
.venv/bin/pytest -q                                   # testy
.venv/bin/python -m uop_b2b --help                    # CLI (od Etapu 4)
.venv/bin/streamlit run app/streamlit_app.py          # UI (od Etapu 7)
```

## Mapa repozytorium

- `docs/architecture/` — ADR + diagramy C4 (Mermaid). Architektura jako kod.
- `docs/analysis/` — produkt merytoryczny: model finansowy, analiza niefinansowa, forecast, decyzja.
- `docs/learning/` — produkt edukacyjny: słownik pojęć, jak działa harness, dziennik nauki.
- `data/params/` — parametry podatkowe z proweniencją. `data/scenarios/` — wejściowe pary ofert.
- `src/uop_b2b/` — silnik. `tests/` — testy. `app/` — UI. `Context/` — surowe materiały źródłowe.
- `out/` — wyniki generowane, poza gitem.

Plan całości: `docs/learning/log.md` (etapy 0–9).
