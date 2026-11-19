# Moduł 7: Wdrożenie i obserwowalność

> ⏱️ Ok. 4 godziny.
> 🔒 **Wejście:** MVP z zabezpieczeniami z Modułu 6.

## O co chodzi

Na Twoim komputerze agent ma jednego użytkownika, który wie, jak z nim rozmawiać. Po wdrożeniu dostaje ludzi, którzy piszą inaczej, niż zakładałeś, i sytuacje, których nie ma w ewaluacjach. Jedyny sposób, żeby się o tym dowiedzieć, to **widzieć, co agent robi** — każdy przebieg, krok po kroku.

Ten moduł to trzy rzeczy: wdrożenie, ślad każdego przebiegu oraz instrukcja na wypadek, gdy coś pójdzie źle.

## Cel

Po tym module:
- agent działa pod publicznym adresem, **za logowaniem**,
- każdy przebieg zostawia **ślad**, który da się odczytać po fakcie,
- masz **cztery liczby** o działaniu agenta i wiesz, gdzie je sprawdzić,
- masz `docs/RUNBOOK.md` — co robić przy awarii, wycieku klucza i skoku kosztów,
- agenta użyły **co najmniej 3 osoby** poza Tobą.

## Kroki

### 1. Przygotuj środowisko produkcyjne
Produkcja jest **oddzielona** od tego, na czym pracujesz lokalnie:

| Element | Lokalnie | Produkcja |
|---|---|---|
| baza danych | projekt deweloperski | **osobny projekt** |
| zmienne środowiskowe | `.env.local` | panel platformy wdrożeniowej |
| sekrety sesji i hasła | testowe | **nowe, wygenerowane** |
| model | najtańszy do pracy | ten, na którym przeszły ewaluacje |

Schemat bazy trzymaj w repo jako pliki SQL lub migracje, żeby drugą bazę dało się postawić jedną operacją — a nie z pamięci.

### 2. Wdróż
Platforma zależy od stacku: aplikację Next.js najprościej wdrożyć na Vercelu, backend w Pythonie — na dowolnym hostingu kontenerów. Niezależnie od wyboru sprawdź po wdrożeniu:

- [ ] wejście bez logowania **nie dociera** do żadnego endpointu wywołującego model (sprawdź `curl`-em bezpośrednio na adres API, nie przez interfejs),
- [ ] adresy powrotu logowania wskazują domenę produkcyjną,
- [ ] zadania dłuższe niż limit czasu funkcji na Twojej platformie kończą się czytelnym komunikatem, a nie urwanym połączeniem,
- [ ] wyłącznik awaryjny z Modułu 6 działa: ustaw zmienną, sprawdź, że agent odmawia, cofnij,
- [ ] w repo i w logach nie ma sekretów.

### 3. Zostaw ślad każdego przebiegu
Ślad (trace) to zapis jednego zadania jako drzewa kroków. Minimum, które musisz mieć:

```
przebieg 7f3a…   użytkownik u_12   start 14:02:11   czas 8,4 s   koszt 0,0031 USD   status: ok
├─ model      wejście 1 840 tok.   wyjście 62 tok.    → wywołanie search_issues
├─ narzędzie  search_issues {"query":"logowanie aktualizacja"}   0,4 s   → 3 wyniki
├─ model      wejście 2 310 tok.   wyjście 48 tok.    → wywołanie read_issue
├─ narzędzie  read_issue {"number":41}   0,3 s   → ok
└─ model      wejście 3 020 tok.   wyjście 210 tok.   → odpowiedź końcowa
```

Dla każdego przebiegu: identyfikator, użytkownik, czas, koszt, status, wersja instrukcji i modelu. Dla każdego kroku: typ, wejście, wyjście (przycięte), czas, tokeny.

Masz dwie drogi:
- **własna tabela w bazie** — dwie tabele (`runs`, `steps`) i prosta strona z listą. Wystarcza na ten projekt i uczy, co naprawdę jest w śladzie;
- **gotowe narzędzie do obserwowalności aplikacji LLM** — większość opiera się na standardzie OpenTelemetry, a popularne SDK potrafią emitować ślady bez ręcznego instrumentowania. Sprawdź w dokumentacji swojego SDK, jak to włączyć w wersji, której używasz.

> 🛑 Ślady zawierają to, co pisali użytkownicy. Zdecyduj, **co zapisujesz i jak długo**, i powiedz o tym użytkownikom. Nie zapisuj sekretów; rozważ maskowanie danych osobowych przed zapisem.

### 4. Patrz na cztery liczby
Z tabeli przebiegów wyprowadź i sprawdzaj regularnie:

| Liczba | Co mówi | Kiedy reagować |
|---|---|---|
| **odsetek przebiegów zakończonych błędem lub limitem kroków** | czy agent sobie radzi | rośnie po zmianie instrukcji lub modelu |
| **koszt na zadanie** (średni i najwyższy) | czy mieścisz się w budżecie | średni powyżej budżetu ze specyfikacji |
| **czas zadania** (mediana i 95. percentyl) | czy da się tego używać | percentyl powyżej limitu ze specyfikacji |
| **liczba kroków na zadanie** | czy agent błądzi | wyraźny wzrost bez zmiany typu zadań |

Zwykłe zapytanie SQL uruchamiane raz dziennie w zupełności wystarczy.

### 5. Zbierz sygnał od ludzi
Dodaj najprostszy mechanizm oceny — kciuk w górę i w dół z opcjonalnym komentarzem, zapisywany razem z identyfikatorem przebiegu. Przy akcjach zatwierdzanych przez człowieka masz ten sygnał za darmo: **odrzucona propozycja to ocena negatywna**.

Każdy negatywnie oceniony przebieg otwierasz, czytasz ślad i — zgodnie z zasadą z Modułu 4 — zamieniasz na przypadek ewaluacyjny.

### 6. Napisz runbook
`docs/RUNBOOK.md` — krótkie instrukcje dla Ciebie o trzeciej w nocy:

```markdown
## Agent nie odpowiada / zwraca błędy
1. Sprawdź status dostawcy modeli i platformy.
2. Otwórz ostatnie przebiegi ze statusem błędu: <gdzie>.
3. Jeśli zawinił ostatni deploy — cofnij do poprzedniego: <jak>.

## Koszt rośnie nienormalnie
1. Włącz wyłącznik awaryjny: <nazwa zmiennej, gdzie ustawić>.
2. Znajdź źródło: zapytanie o koszt wg użytkownika i godziny: <SQL>.
3. Zablokuj konto / obniż limit: <jak>.

## Podejrzenie wycieku klucza lub sekretu
1. Wyłącznik awaryjny.
2. Klucz kursowy: natychmiast zgłoś prowadzącemu. Własne klucze: unieważnij i wygeneruj nowe.
3. Zmień sekrety sesji, wygeneruj nowe klucze bazy, wdróż ponownie.
4. Sprawdź historię git i logi pod kątem sekretu.

## Agent zrobił coś, czego nie powinien
1. Zapisz identyfikator przebiegu.
2. Wyłącz narzędzie, którego dotyczy problem: <jak>.
3. Dodaj przypadek do ewaluacji, zanim naprawisz.
```

Każdą procedurę **wykonaj raz na sucho**. Runbook, którego nikt nie przeszedł, jest listą życzeń.

### 7. Daj agenta trzem osobom
Trzy osoby, najlepiej takie, dla których jest przeznaczony. Bez instrukcji obsługi poza jednym zdaniem. Potem przeczytaj ich ślady — nie pytaj tylko o wrażenia. Zapisz w `docs/DZIENNIK.md`, co zrobili inaczej, niż zakładały scenariusze.

## ✅ Dowód „done"

- Adres produkcyjny działa; żądanie bez sesji bezpośrednio na endpoint API dostaje odmowę.
- Dla dowolnego przebiegu potrafisz pokazać pełny ślad z kosztem i czasem.
- `docs/RUNBOOK.md` zawiera cztery procedury, każda przećwiczona.
- W bazie są przebiegi co najmniej 3 osób innych niż Ty; `docs/DZIENNIK.md` ma wpis z wnioskami.

## 🤖 Sprawdź się przez AI

> *„Oto opis wdrożenia mojego agenta, schemat tabel ze śladami, przykładowy ślad i runbook: [wklej]. Oceń jak osoba, która ma przejąć dyżur przy tym systemie — PASS/FAIL + jedno zdanie:*
> *1. Czy na podstawie śladu da się odtworzyć, dlaczego agent podjął każdą decyzję?*
> *2. Czy ślad zawiera wersję instrukcji i modelu, żeby dało się porównać zachowanie przed i po zmianie?*
> *3. Czy w śladach lub logach mogą znaleźć się sekrety albo dane, których nie powinno się przechowywać?*
> *4. Czy procedury w runbooku są wykonalne przez kogoś, kto nie pisał tego kodu?*
> *5. Jakiej sytuacji awaryjnej brakuje?*
> *Nie odczytuj i nie wypisuj plików .env."*

## 🚀 Idź dalej (nieobowiązkowe)

1. **Ewaluacje na danych z produkcji.** Raz w tygodniu wylosuj 10 przebiegów, oceń je ręcznie i dopisz najciekawsze do zestawu.
2. **Wersjonuj instrukcje.** Zapisuj w śladzie skrót pliku instrukcji i porównuj cztery liczby między wersjami.
3. **Wdrażaj zmiany stopniowo.** Nową wersję instrukcji włącz najpierw dla części przebiegów i porównaj wyniki, zanim przełączysz wszystkich.
