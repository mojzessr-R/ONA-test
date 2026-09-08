# Context — materiały źródłowe

Tu wrzucasz **surowe wejście** do analizy. Agent czyta te pliki, ale ich nie generuje.

## Co tu wrzucać

- `offers/` — konkretne oferty do porównania: PDF-y, zrzuty maili, notatki z rozmów.
  Nazewnictwo: `offers/2026-09-firma-uop.md`, `offers/2026-09-firma-b2b.md`.
- Wyciągi ze stron ZUS / gov.pl / Ministerstwa Finansów potwierdzające stawki na dany rok
  (agent nie ma dostępu do internetu — jeśli chcesz, żeby coś zweryfikował, musisz to tu położyć).
- Cokolwiek, co jest faktem z zewnątrz, a nie wytworem tego projektu.

## Czego tu nie wrzucać

- Wyników analizy → `docs/analysis/`
- Parametrów podatkowych w formie ustrukturyzowanej → `data/params/`
- **Danych wrażliwych, których nie chcesz w gicie** (imiona, nazwy firm, kwoty realnych ofert),
  jeśli repo miałoby kiedykolwiek trafić poza Twoją maszynę. Do testów użyj wartości zmyślonych
  i trzymaj je w `data/scenarios/`.

## Granica odpowiedzialności

`Context/` to fakty z zewnątrz. `docs/` to nasze wnioski. `data/` to nasze dane wejściowe
w formacie maszynowym. Ta separacja pozwala w każdej chwili odpowiedzieć na pytanie
„skąd to wiemy" — kluczowe, gdy analiza dotyczy zmiennego prawa.
