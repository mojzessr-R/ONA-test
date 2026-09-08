# UoP vs B2B — kalkulator i analiza decyzyjna

PoC porównujący dwie formy współpracy: **umowę o pracę** i **B2B na ryczałcie 12%**
(Polska, rok bazowy 2026). Odpowiada na pytanie „która oferta jest lepsza" w trzech wymiarach:
finansowym, niefinansowym i w perspektywie wieloletniej.

Projekt jest jednocześnie **ćwiczeniem z agentic development i architecture-as-code** —
patrz [`docs/learning/`](docs/learning/).

## Szybki start

```bash
# Środowisko (obraz bazowy jest pusty — Python instalujemy sami)
sudo apt-get install -y python3 python3-venv python3-pip
python3 -m venv .venv
.venv/bin/pip install -r requirements.txt

# Weryfikacja
.venv/bin/python -c "import yaml, pytest, streamlit; print('env ok')"
```

Od kolejnych etapów dojdą:

```bash
.venv/bin/pytest -q                                    # testy silnika        (Etap 3)
.venv/bin/python -m uop_b2b compare --scenario ...     # porównanie z CLI     (Etap 4)
.venv/bin/python -m uop_b2b forecast --years 5         # projekcja wieloletnia(Etap 6)
.venv/bin/streamlit run app/streamlit_app.py           # kalkulator w UI      (Etap 7)
```

## Mapa repozytorium

| Katalog | Co zawiera |
|---|---|
| `docs/architecture/` | ADR-y i diagramy C4 w Mermaid — **architektura jako kod** |
| `docs/analysis/` | produkt merytoryczny: model finansowy, analiza jakościowa, forecast, decyzja |
| `docs/learning/` | produkt edukacyjny: [słownik pojęć](docs/learning/00-glossary.md), dziennik nauki |
| `data/params/` | parametry podatkowe z polami `source` i `verified_on` |
| `data/scenarios/` | wejściowe pary ofert do porównania |
| `src/uop_b2b/` | silnik obliczeniowy — czysty, bez I/O i bez UI |
| `tests/` | testy jednostkowe i golden files |
| `app/` | cienkie UI nad silnikiem |
| `Context/` | surowe materiały źródłowe (oferty, wyciągi ze stron urzędowych) |
| `.claude/` | konfiguracja harnessu: uprawnienia, slash commands, skille, hooki |

## Zasady projektu

1. Silnik w `src/` jest czysty — zero I/O, zero UI, wejście i wyjście to dataclasses.
2. Parametry podatkowe to **dane z podanym źródłem**, nigdy liczby w kodzie.
3. Forma opodatkowania to strategia — nowa forma = nowy plik, nie nowy `if`.
4. Każda decyzja architektoniczna ma ADR, w tym samym commicie co jej realizacja.
5. Kwoty liczone na `Decimal`. Żadna wartość podatkowa nie trafia do repo bez weryfikacji.

Pełny zestaw reguł dla agenta: [`CLAUDE.md`](CLAUDE.md).

## Status

Realizacja etapami 0–9, dziennik postępu w [`docs/learning/log.md`](docs/learning/log.md).

**Aktualny stan: Etap 0 ukończony** — środowisko, konstytucja projektu, słownik pojęć.

## Zastrzeżenie

To narzędzie edukacyjne i pomocnicze. **Nie jest poradą podatkową ani prawną.** Parametry
podatkowe wymagają samodzielnej weryfikacji ze źródłem urzędowym przed użyciem wyników
do rzeczywistej decyzji — środowisko, w którym powstaje projekt, nie ma dostępu do internetu,
więc żadna stawka nie została zweryfikowana automatycznie.
