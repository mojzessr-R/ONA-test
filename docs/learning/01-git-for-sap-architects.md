# Git dla architekta SAP

Notatka powstała, bo padło pytanie: „o co chodzi z branchami i commitowaniem, w SAPie nie ma
takich rzeczy". Teza tej notatki: **są, tylko nazywają się inaczej i kosztują dziesięć razy więcej.**

---

## 1. Mapowanie pojęć

| Git | Odpowiednik w SAP | Komentarz |
|---|---|---|
| working tree (pliki na dysku) | obiekt otwarty w edytorze, jeszcze niezaktywowany | Twoje bieżące, niezapisane trwale zmiany |
| staging area (`git add`) | przypisanie obiektu do **zadania (task)** w transporcie | wybierasz, co wchodzi do najbliższej porcji |
| **commit** | **zwolnienie zadania (release task)** | nazwana, podpisana, atomowa porcja zmian |
| `git push` | zwolnienie transportu (release TR) → import do QAS | dopiero teraz zmiana opuszcza Twoją maszynę |
| **branch** | osobny tor rozwoju — drugi landscape (N / N+1), projekt vs maintenance | równoległa linia zmian |
| **merge** | **retrofit** (SCWB / retrofit tool) | scalenie dwóch torów |
| `git log` | historia wersji obiektu (SE38 → Utilities → Versions) + lista TR | kto, kiedy, co, po co |
| tag | milestone / release / wersja produkcyjna | nazwany punkt w historii |
| repozytorium | landscape + baza wersji razem | całość, z pełną historią |

**Wniosek:** znasz to wszystko. Znasz nawet najtrudniejszą część — retrofit między torami — czyli
merge, tylko robiony ręcznie. Git nie wprowadza nowych pojęć; wprowadza *tanie* pojęcia.

> Uwaga na przyszłość: SAP idzie w tę stronę. **abapGit** przynosi prawdziwego gita do ABAP-a,
> **gCTS** (git-enabled CTS) to gitowe transporty w S/4HANA, a BTP/CAP/Fiori używają gita
> natywnie od początku. To, czego się teraz uczysz, nie jest odskocznią od Twojej ścieżki.

---

## 2. Cztery różnice, które naprawdę mają znaczenie

### Różnica 1: commit jest lokalny i darmowy

Zwolnienie transportu w SAPie to **zdarzenie w landscapie** — coś wyjeżdża, ktoś to zaimportuje,
cofnięcie jest bolesne. Dlatego pakujesz w jeden TR możliwie dużo i wahasz się przed zwolnieniem.

`git commit` **nie wysyła niczego nigdzie**. Zapisuje punkt kontrolny na Twoim dysku.
Dopiero `git push` odpowiada zwolnieniu transportu.

Konsekwencja dla nawyku: **commituj często i drobno.** Dwadzieścia commitów dziennie to norma,
nie patologia. Jeden commit = jedna sensowna, kompletna zmiana.

### Różnica 2: git zapisuje migawkę całego repo, nie listę obiektów

TR to **lista obiektów** (`ZCL_FOO`, `ZTAB_BAR`). Jeśli ktoś ruszył ten sam obiekt w innym TR,
masz konflikt na poziomie obiektu i objects lock.

Commit to **migawka całego drzewa plików** w danym momencie. Możesz odtworzyć cały stan projektu
z dowolnego punktu historii — nie tylko listę tego, co się zmieniło, ale kompletny, działający stan.

To jest właśnie fundament, na którym stoi architecture-as-code: skoro commit zawiera *wszystko*,
to ADR i kod, który ten ADR realizuje, mogą być w **jednym commicie** — i nigdy się nie rozjadą.

### Różnica 3: branch to jedna linijka tekstu

Drugi tor rozwoju w SAPie to serwery, licencje, synchronizacja, retrofit i etat na utrzymanie.

Branch w gicie to **plik z jednym identyfikatorem commita**. Stworzenie: 0,01 s. Skasowanie: tyle samo.

Dlatego strategie, które w SAPie są nie do udźwignięcia, w gicie są rutyną: osobny branch
na eksperyment, na review, na każde zadanie. Jeśli eksperyment się nie uda — kasujesz branch
i po sprawie. Nic nie zostaje.

### Różnica 4: merge jest automatyczny (3-way)

Retrofit robisz w dużej mierze ręcznie, obiekt po obiekcie.

Git zna **wspólnego przodka** obu gałęzi, więc potrafi automatycznie połączyć zmiany, które
dotyczą różnych fragmentów plików. Pyta Cię tylko o realne konflikty — te same linie zmienione
po obu stronach. W praktyce 90% merge'ów przechodzi bez Twojego udziału.

---

## 3. Jak to wygląda mechanicznie

```
   pliki na dysku          poczekalnia            historia
  (working tree)       (staging area)          (repozytorium)
        │                     │                       │
        │─── git add ────────>│                       │
        │                     │─── git commit ───────>│
        │                     │                       │
        │<────────────── git checkout ────────────────│
```

Trzy poziomy zamiast dwóch. Poczekalnia (staging) istnieje po to, żeby z dziesięciu zmienionych
plików wybrać pięć, które tworzą jedną spójną zmianę, i zamknąć je w jednym commicie.
To dokładnie ta sama funkcja, co przypisywanie obiektów do konkretnego zadania w TR.

Branch i merge:

```
main      A───B───────────────────M         M = merge, scala obie linie
               \                 /
poc/…           C───D───E───F───┘           C…F = kolejne etapy PoC
```

`main` przez cały czas pozostaje w stanie B — sprawnym i czystym. Praca toczy się obok.
Gdy jest gotowa, jeden merge przenosi ją całą.

---

## 4. Co realnie robimy w tym projekcie

Ustaliliśmy 10 etapów. Na koniec każdego jest **jeden commit**. To daje trzy rzeczy:

1. **Punkty kontrolne.** Etap 5 coś popsuł? `git checkout` na commit z Etapu 4 i masz z powrotem
   stan działający. Bez kopiowania folderów „projekt_v2_final_OK".
2. **Diff jako materiał do nauki.** `git show` na commicie etapu pokazuje dokładnie, co ten etap
   wniósł — nic więcej, nic mniej. To najlepszy możliwy przegląd „co się właściwie stało".
3. **Decyzja i realizacja w jednym miejscu.** ADR i kod, który go realizuje, jadą w tym samym
   commicie. Za rok `git log` odpowie nie tylko „co", ale i „dlaczego".

Punkt 3 to nie kosmetyka. To **cała istota architecture-as-code** i powód, dla którego
architektura w repo bije architekturę w Confluence: slajd i system rozjeżdżają się po miesiącu,
commit nie rozjedzie się nigdy.

---

## 5. Minimalny zestaw poleceń

Tyle wystarczy na cały ten projekt:

```bash
git status              # co się zmieniło — używaj bez opamiętania, nic nie psuje
git diff                # co dokładnie się zmieniło, linia po linii
git add -A              # wrzuć wszystkie zmiany do poczekalni
git commit -m "opis"    # zamknij porcję zmian, lokalnie
git log --oneline       # historia
git show <hash>         # co wniósł konkretny commit
```

Rzadziej:

```bash
git checkout -b nazwa   # nowa gałąź i przełączenie się na nią
git switch main         # przejście na inną gałąź
git merge nazwa         # scal gałąź do bieżącej
git push                # dopiero to wysyła cokolwiek na zewnątrz
```

Zasada bezpieczeństwa: **wszystko, co zostało zacommitowane, da się odzyskać.**
Ryzykowne są wyłącznie zmiany *niezacommitowane*. To kolejny argument, żeby commitować często.

---

## 6. Trzy najczęstsze nieporozumienia

| Mit | Jak jest |
|---|---|
| „Commit wysyła zmiany do repozytorium zdalnego" | Nie. Commit jest lokalny. Wysyła `git push`. |
| „Branch to kopia projektu" | Nie. To wskaźnik na commit. Nic się nie kopiuje. |
| „Trzeba commitować tylko rzeczy skończone" | Odwrotnie. Commituj każdy sensowny krok. Historię można potem uporządkować. |
