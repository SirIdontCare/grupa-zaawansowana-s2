# Moduł 7: Wdrożenie i obserwowalność

> ⏱️ Ok. 4 godziny. Potrzebujesz działającego agenta z zabezpieczeniami z [Modułu 6](M6_BEZPIECZENSTWO_I_KOSZTY.md): limity, budżet dzienny, wyłącznik awaryjny, logowanie.

## O co chodzi

Do tej pory agent działał „u Ciebie". Miał jednego użytkownika, który wie, jak z nim rozmawiać, i który sam napisał wszystkie przykłady testowe. „Działa dla ludzi" to coś innego: agent stoi pod adresem w internecie, a korzystają z niego osoby, które piszą inaczej, niż zakładałeś, i trafiają na sytuacje, których nie ma w Twoich ewaluacjach (czyli w zestawie przykładów, na których mierzysz agenta od Modułu 4).

Kłopot w tym, że tych ludzi nie widzisz. Nie siedzisz obok, gdy agent odpowiada im bzdurą albo kręci się w kółko. Dlatego potrzebujesz czegoś w rodzaju **czarnej skrzynki w samolocie**: urządzenia, które zapisuje każdy lot, krok po kroku. Kiedy lot przebiega spokojnie, nikt do niej nie zagląda. Kiedy coś pójdzie źle, to jedyne miejsce, z którego da się odczytać, co się naprawdę stało.

Ten moduł składa się z trzech rzeczy: wystawiasz agenta pod publicznym adresem, sprawiasz, że każde jego zadanie zostawia czytelny zapis, i spisujesz krótką instrukcję na wypadek awarii.

## Po tym module masz

- agenta pod publicznym adresem, dostępnego **tylko po zalogowaniu**,
- zapis każdego przebiegu: co agent zrobił, ile to trwało i ile kosztowało,
- **cztery liczby** o działaniu agenta i jedno zapytanie, które je liczy,
- plik `docs/RUNBOOK.md` z czterema przećwiczonymi procedurami awaryjnymi,
- przebiegi **co najmniej 3 osób** poza Tobą i wnioski z nich w `docs/DZIENNIK.md`.

## Słowa, które tu padną

| Słowo | Co to znaczy | Porównanie z życia |
|---|---|---|
| **Wdrożenie** (deploy) | umieszczenie aplikacji na serwerze, żeby działała pod adresem w internecie | przeniesienie dania z kuchni próbnej do restauracji, w której siedzą goście |
| **Środowisko produkcyjne** (produkcja) | ta kopia aplikacji, bazy i ustawień, z której korzystają prawdziwi ludzie | sala restauracji; Twój komputer to kuchnia próbna |
| **Zmienna środowiskowa** | ustawienie przekazywane aplikacji z zewnątrz, poza kodem: klucze, adresy, przełączniki | kartka z kodem do alarmu, której nie drukujesz w ulotce firmy |
| **Migracja** | plik z poleceniami SQL, który tworzy albo zmienia tabele w bazie | instrukcja montażu mebla: każdy, kto ją wykona, dostaje taki sam mebel |
| **Endpoint** | adres w aplikacji, pod który wysyła się żądania, np. ten, który uruchamia agenta | okienko w urzędzie: jedno do jednej sprawy |
| **Sesja** | dowód, że żądanie pochodzi od zalogowanej osoby | opaska na rękę na koncercie |
| **`curl`** | program w terminalu, który wysyła żądanie pod adres z pominięciem przeglądarki i Twojego interfejsu | zapukanie do tylnych drzwi, żeby sprawdzić, czy też są zamknięte |
| **Przebieg** | jedno zadanie agenta od początku do końca, np. jedno zapytanie zamienione w szkic oferty | jeden lot samolotu |
| **Ślad przebiegu** (trace) | zapis jednego przebiegu jako drzewa kroków | zapis czarnej skrzynki z jednego lotu |
| **Krok** (span) | jeden element śladu: jedno wywołanie modelu albo jedno użycie narzędzia | jeden wpis w dzienniku pokładowym |
| **Obserwowalność** | możliwość ustalenia po fakcie, co system zrobił i dlaczego, bez zgadywania | monitoring w sklepie: przewijasz nagranie zamiast pytać świadków |
| **Mediana** | wartość środkowa: połowa wyników jest mniejsza, połowa większa | „typowy" czas dojazdu do pracy, bez dnia, gdy stałeś w korku dwie godziny |
| **95. percentyl** | wartość, poniżej której mieści się 95% wyników | czas, w którym zdążysz do pracy w 19 dni na 20 |
| **Runbook** | krótka instrukcja „co robić, gdy…", pisana z wyprzedzeniem | instrukcja ewakuacyjna na drzwiach hotelu |
| **Wycofanie wdrożenia** (rollback) | powrót do poprzedniej, działającej wersji aplikacji | „cofnij" w edytorze, tylko dla całej aplikacji |
| **OpenTelemetry** | otwarty standard zapisu śladów, wspólny dla wielu narzędzi | wtyczka jednego typu: pasuje do gniazdek różnych producentów |

**Mediana i 95. percentyl na 20 liczbach.** Ofertownik przygotował 20 szkiców. Czasy w sekundach, ustawione od najkrótszego:

```text
8  9  10  10  11  11  12  12  13  13  |  13  14  14  15  16  17  19  22  41  58
```

- **Średnia** to 16,9 s. Zawyżają ją dwa ostatnie przebiegi; żaden typowy szkic nie trwał tyle.
- **Mediana** to środek listy: dziesiąta i jedenasta liczba to 13 i 13, więc mediana wynosi 13 s. Tyle trwa zwykły szkic.
- **95. percentyl** to dziewiętnasta liczba z dwudziestu: 41 s. Znaczy: 19 na 20 szkiców było gotowych w 41 s albo szybciej.

Mediana mówi, jak jest zwykle. 95. percentyl mówi, jak jest w złe dni, a ludzie pamiętają głównie złe dni. Dlatego patrzysz na obie liczby, a nie na średnią.

## Przykład: Ofertownik

Przypomnienie: Ofertownik to agent, który z maila z zapytaniem przygotowuje dla Ani szkic oferty szkoleniowej. Scenariusz S1 to zapytanie typowe: „Excel dla 12 osób, październik".

### Ślad jednego przebiegu S1

```text
przebieg 7f3a…  użytkownik: ania  start 14:02:11  czas 12,4 s  koszt 0,0180 USD  status: ok
instrukcja: v7   model: ten sam, na którym przeszły ewaluacje
├─ model      wejście 1 900 tok.  wyjście 40 tok.   1,8 s  0,0031 USD  → chce użyć search_price_list
├─ narzędzie  search_price_list {"query":"Excel"}                0,3 s  → 2 pozycje z cennika
├─ model      wejście 2 400 tok.  wyjście 45 tok.   1,9 s  0,0038 USD  → chce użyć check_calendar
├─ narzędzie  check_calendar {"from":"2026-10-01","to":"2026-10-31"}  0,4 s  → 3 wolne terminy
├─ model      wejście 2 900 tok.  wyjście 380 tok.  5,6 s  0,0062 USD  → chce użyć save_offer_draft
├─ narzędzie  save_offer_draft {…}                               0,2 s  → szkic nr 318
└─ model      wejście 3 400 tok.  wyjście 90 tok.   2,2 s  0,0049 USD  → odpowiedź końcowa
```

Co z tego widać bez zaglądania do kodu: agent zrobił 7 kroków, najdłużej trwało pisanie szkicu (5,6 s), a całość zmieściła się w limitach z briefu (60 s i 0,05 USD). Gdyby Ania zgłosiła, że w szkicu jest zła cena, otwierasz ten ślad i sprawdzasz drugi wiersz: co zwrócił cennik.

„Instrukcja" to tekst, który mówi modelowi, kim jest i jakich zasad ma przestrzegać (prompt systemowy). Jej wersję zapisujesz w śladzie, żeby po zmianie instrukcji wiedzieć, które przebiegi były „przed", a które „po".

### Tabele `runs` i `steps`

Ślad mieści się w dwóch tabelach: jedna trzyma przebiegi, druga ich kroki.

```sql
-- poglądowo: dopasuj typy i nazwy do swojego projektu
create table runs (
  id             uuid primary key,
  user_id        uuid,
  started_at     timestamptz,
  duration_ms    integer,
  cost_usd       numeric,
  status         text,      -- 'ok' | 'error' | 'step_limit'
  step_count     integer,
  prompt_version text,      -- wersja instrukcji
  model          text,
  rating         integer,   -- 1 = kciuk w górę, -1 = w dół, puste = brak oceny
  rating_comment text
);

create table steps (
  id            uuid primary key,
  run_id        uuid references runs (id),
  position      integer,    -- kolejność kroku w przebiegu
  type          text,       -- 'model' | 'tool'
  name          text,       -- np. 'search_price_list'
  input         jsonb,      -- przycięte
  output        jsonb,      -- przycięte
  duration_ms   integer,
  input_tokens  integer,
  output_tokens integer
);
```

### Cztery liczby Ofertownika

Stan po tygodniu używania przez Anię i dwie osoby z jej firmy:

| Liczba | Wynik | Limit ze specyfikacji | Ocena |
|---|---|---|---|
| przebiegi zakończone błędem lub limitem kroków | 3 z 74 (4,1%) | brak twardego limitu; ma nie rosnąć | 🟢 |
| koszt szkicu: średni / najwyższy | 0,019 / 0,047 USD | średni najwyżej 0,05 USD | 🟢 |
| czas szkicu: mediana / 95. percentyl | 13 s / 41 s | najwyżej 60 s | 🟢 |
| liczba kroków: średnia / najwyższa | 7,2 / 15 | brak; ma nie rosnąć | 🟡 sprawdzić przebieg z 15 krokami |

Wszystkie pochodzą z jednego zapytania:

```sql
select
  count(*)                                                          as przebiegi,
  round(100.0 * count(*) filter (where status <> 'ok') / count(*), 1) as procent_nieudanych,
  round(avg(cost_usd), 4)                                           as koszt_sredni,
  max(cost_usd)                                                     as koszt_najwyzszy,
  percentile_cont(0.5)  within group (order by duration_ms) / 1000  as czas_mediana_s,
  percentile_cont(0.95) within group (order by duration_ms) / 1000  as czas_p95_s,
  round(avg(step_count), 1)                                         as kroki_srednio,
  max(step_count)                                                   as kroki_najwiecej
from runs
where started_at > now() - interval '7 days';
```

### Runbook Ofertownika

Nazwy w ostrych nawiasach zastąp swoimi. `AGENT_WYLACZONY` to przykładowa nazwa zmiennej wyłącznika awaryjnego; u Ciebie jest to ta, którą ustaliłeś w Module 6.

```markdown
# Runbook: Ofertownik

Adres produkcyjny: <adres>. Lista przebiegów: <adres>/przebiegi (tylko dla właściciela).
Zmienne środowiskowe: panel platformy wdrożeniowej → ustawienia projektu.

## 1. Agent nie odpowiada albo zwraca błędy
1. Sprawdź strony statusu: dostawcy modeli (OpenRouter), platformy (Vercel), bazy (Supabase).
2. Otwórz listę przebiegów i odfiltruj status „error". Przeczytaj ostatni krok trzech najnowszych.
3. Jeśli błędy zaczęły się po ostatnim wdrożeniu: w panelu platformy przywróć
   poprzednie wdrożenie. Naprawiaj dopiero potem, lokalnie.
4. Napisz Ani, że szkice chwilowo nie działają i do kiedy.

## 2. Koszt rośnie nienormalnie
1. Włącz wyłącznik awaryjny: ustaw AGENT_WYLACZONY=1 i wdróż ponownie.
2. Znajdź źródło:
   select user_id, date_trunc('hour', started_at) as godzina,
          count(*) as przebiegi, sum(cost_usd) as koszt
   from runs where started_at > now() - interval '24 hours'
   group by 1, 2 order by koszt desc limit 10;
3. Jedno konto odpowiada za skok → zablokuj je w panelu logowania bazy.
   Wszyscy po trochu → obniż dzienny budżet z Modułu 6.
4. Wyłącz wyłącznik dopiero, gdy wiesz, skąd wziął się skok.

## 3. Podejrzenie wycieku klucza lub sekretu
1. Włącz wyłącznik awaryjny.
2. Klucz kursowy: natychmiast zgłoś prowadzącemu.
   Własne klucze: unieważnij u dostawcy i wygeneruj nowe.
3. Zmień sekrety sesji, wygeneruj nowe klucze bazy, wpisz je w panelu platformy, wdróż ponownie.
4. Przeszukaj historię git i logi pod kątem sekretu. Zapisz, którędy wyciekł.

## 4. Agent zrobił coś, czego nie powinien
(np. szkic z ceną spoza cennika albo rabat powyżej 10%)
1. Zapisz identyfikator przebiegu.
2. Wyłącz narzędzie, którego dotyczy problem: usuń je z listy narzędzi agenta
   w <plik> i wdróż. Przy błędnych szkicach: save_offer_draft.
3. Dodaj ten przypadek do ewaluacji w evals/, ZANIM cokolwiek naprawisz.
4. Napraw, uruchom ewaluacje, włącz narzędzie z powrotem.
```

## Kroki

### Krok 1 — Przygotuj środowisko produkcyjne

**Po co:** jeśli produkcja i Twój komputer dzielą jedną bazę i te same hasła, każdy eksperyment może zepsuć coś prawdziwym użytkownikom, a testowe dane mieszają się z prawdziwymi.

**Co robisz:** oddzielasz produkcję od tego, na czym pracujesz lokalnie.

| Element | Lokalnie | Produkcja |
|---|---|---|
| baza danych | projekt deweloperski | **osobny projekt** |
| zmienne środowiskowe | `.env.local` | panel platformy wdrożeniowej |
| sekrety sesji i hasła | testowe | **nowe, wygenerowane** |
| model | najtańszy do pracy | ten, na którym przeszły ewaluacje |

1. Załóż drugi projekt bazy danych, tylko na produkcję.
2. Zapisz schemat bazy (tabele, uprawnienia) w repozytorium jako pliki SQL lub migracje. Chodzi o to, żeby drugą bazę dało się postawić jedną operacją, a nie odtwarzać z pamięci, co kiedyś wyklikałeś w panelu.
3. Wykonaj te pliki na bazie produkcyjnej.
4. Wygeneruj nowe sekrety sesji i hasła. Nie przenoś testowych.
5. Przygotuj listę nazw zmiennych środowiskowych, które trzeba wpisać w panelu platformy. Same nazwy, bez wartości.

**Prompt do asystenta:**

```text
Przygotowujemy środowisko produkcyjne dla tego projektu. Przeczytaj
docs/ARCHITEKTURA.md oraz kod, który korzysta z bazy danych.

1. Sprawdź, czy cały schemat bazy (tabele, indeksy, uprawnienia) jest zapisany
   w repozytorium jako pliki SQL lub migracje. Jeśli czegoś brakuje, dopisz
   brakujące pliki tak, żeby pustą bazę dało się postawić, wykonując je po kolei.
2. Wypisz w docs/RUNBOOK.md (utwórz plik, jeśli go nie ma) sekcję
   „Zmienne środowiskowe": nazwa każdej zmiennej, do czego służy i czy na
   produkcji ma mieć inną wartość niż lokalnie. Same nazwy, bez wartości.
3. Wskaż miejsca w kodzie, w których adres, klucz albo hasło są wpisane na stałe
   zamiast pochodzić ze zmiennej środowiskowej.

Nie czytaj i nie wypisuj wartości z plików .env. Nie łącz się z żadną bazą
i niczego nie wdrażaj. Na końcu podaj kolejność, w jakiej mam wykonać pliki SQL.
```

**Gotowe, gdy:** na pustej bazie produkcyjnej wykonałeś pliki SQL z repozytorium i powstały wszystkie tabele, bez ręcznego klikania.

### Krok 2 — Wdróż i przejdź listę kontrolną

**Po co:** po wdrożeniu aplikacja zachowuje się inaczej niż lokalnie: ma inny adres, inne limity czasu i jest widoczna dla całego internetu, także dla automatów, które szukają niezabezpieczonych adresów.

**Co robisz:**

1. Wybierz platformę zgodną ze stackiem. Aplikację Next.js najprościej wdrożyć na Vercelu, backend w Pythonie na dowolnym hostingu kontenerów.
2. Podłącz repozytorium, wpisz w panelu platformy zmienne środowiskowe z kroku 1 i uruchom wdrożenie.
3. Przejdź listę kontrolną. Każdy punkt sprawdzasz na adresie produkcyjnym:

- [ ] wejście bez logowania **nie dociera** do żadnego endpointu wywołującego model (sprawdź `curl`-em bezpośrednio na adres API, nie przez interfejs),
- [ ] adresy powrotu logowania wskazują domenę produkcyjną,
- [ ] zadania dłuższe niż limit czasu funkcji na Twojej platformie kończą się czytelnym komunikatem, a nie urwanym połączeniem,
- [ ] wyłącznik awaryjny z Modułu 6 działa: ustaw zmienną, sprawdź, że agent odmawia, cofnij,
- [ ] w repozytorium i w logach nie ma sekretów.

Pierwszy punkt jest najważniejszy. To, że strona pokazuje ekran logowania, niczego nie dowodzi: endpoint pod spodem może nadal odpowiadać każdemu, kto zna adres, i każdy taki strzał kosztuje Twoje pieniądze. Test wygląda tak:

```bash
# wstaw adres swojego endpointu; żądanie nie zawiera sesji
curl -i -X POST https://<twoj-adres>/<sciezka-endpointu-agenta> \
  -H "Content-Type: application/json" \
  -d '{"message":"test"}'
```

- ✅ Dobrze: odpowiedź z kodem odmowy (np. 401) i żadnego nowego przebiegu w bazie.
- ❌ Źle: kod 200 i odpowiedź modelu albo przekierowanie, po którym odpowiedź jednak przychodzi.

**Prompt do asystenta:**

```text
Agent jest wdrożony pod adresem: [wklej adres produkcyjny].
Pomóż mi przejść listę kontrolną po wdrożeniu. Nic nie zmieniaj w kodzie.

1. Znajdź w kodzie wszystkie endpointy, które wywołują model albo narzędzia
   agenta. Wypisz ich ścieżki i metody.
2. Dla każdego przygotuj gotowe polecenie curl, które wysyła żądanie BEZ sesji
   na adres produkcyjny. Napisz, jaką odpowiedź uznajemy za poprawną odmowę.
3. Wskaż w kodzie, gdzie ustawiane są adresy powrotu logowania, i powiedz,
   co mam sprawdzić w panelu dostawcy logowania.
4. Sprawdź, co zobaczy użytkownik, gdy zadanie przekroczy limit czasu funkcji
   na platformie. Jeśli połączenie po prostu się urwie, zaproponuj zmianę
   (sam jej nie wprowadzaj).
5. Opisz krok po kroku test wyłącznika awaryjnego: jaką zmienną ustawić, co
   powinno się stać, jak to cofnąć.
6. Przeszukaj repozytorium pod kątem sekretów wpisanych w kod lub dokumenty.

Nie czytaj i nie wypisuj wartości z plików .env. Poleceń curl nie uruchamiaj
sam: wykonam je i wkleję Ci wyniki do oceny.
```

**Gotowe, gdy:** wszystkie pięć punktów listy jest odhaczonych, a polecenie `curl` bez sesji dostało odmowę na każdym endpoincie wywołującym model.

### Krok 3 — Zostaw ślad każdego przebiegu

**Po co:** gdy ktoś powie „agent dał mi dziwną odpowiedź", bez śladu możesz tylko zgadywać. Ze śladem otwierasz przebieg i czytasz, który krok poszedł źle.

**Co robisz:** zapisujesz każdy przebieg tak, jak w przykładzie Ofertownika wyżej. Minimum:

- **dla każdego przebiegu:** identyfikator, użytkownik, czas, koszt, status, wersja instrukcji i modelu,
- **dla każdego kroku:** typ (model albo narzędzie), wejście, wyjście (przycięte), czas, tokeny.

Masz dwie drogi:

- **własna tabela w bazie**: dwie tabele (`runs`, `steps`) i prosta strona z listą. Wystarcza na ten projekt i uczy, co naprawdę jest w śladzie. Ta droga jest opisana w krokach poniżej;
- **gotowe narzędzie do obserwowalności aplikacji LLM**: większość opiera się na standardzie OpenTelemetry, a popularne SDK potrafią emitować ślady bez ręcznego dopisywania kodu przy każdym kroku. Sprawdź w dokumentacji swojego SDK, jak to włączyć w wersji, której używasz.

> 🛑 **Ślady zawierają to, co pisali użytkownicy.** Zdecyduj, **co zapisujesz i jak długo**, i powiedz o tym użytkownikom. Nie zapisuj sekretów; rozważ maskowanie danych osobowych przed zapisem. W Ofertowniku w śladach lądują maile klientów Ani, więc adresy e-mail i numery telefonów są maskowane przed zapisem, ślady są kasowane po 30 dniach, a informacja o tym jest na ekranie logowania.

Trzy rzeczy, o które łatwo się potknąć:

- zapis śladu nie może zepsuć zadania. Jeśli baza śladów chwilowo nie działa, użytkownik i tak ma dostać odpowiedź;
- przebieg zakończony błędem albo limitem kroków też musi mieć ślad. To właśnie te chcesz czytać;
- duże wyniki narzędzi przycinaj. Wystarczy początek i informacja, ile było całości.

**Prompt do asystenta:**

```text
Dodajemy ślady przebiegów agenta (własne tabele w bazie). Przeczytaj
docs/ARCHITEKTURA.md, kod pętli agenta i definicje narzędzi.

1. Przygotuj plik SQL (albo migrację, zgodnie z tym, jak projekt trzyma schemat)
   tworzący dwie tabele:
   - runs: identyfikator, użytkownik, czas startu, czas trwania, koszt, status
     (ok / błąd / limit kroków), liczba kroków, wersja instrukcji, model;
   - steps: przebieg, kolejność, typ (model / narzędzie), nazwa, wejście,
     wyjście, czas trwania, tokeny wejścia i wyjścia.
   Dostęp do obu tabel tylko po stronie serwera.
2. Dopisz zapis: jeden wiersz w runs na każde zadanie i jeden wiersz w steps
   na każde wywołanie modelu i każde użycie narzędzia. Skorzystaj z danych
   o tokenach i krokach, które zwraca używane w projekcie SDK; sprawdź w jego
   dokumentacji dla zainstalowanej wersji, jak je odczytać. Niczego nie zgaduj.
3. Zasady zapisu:
   - wejście i wyjście kroku przycinaj do rozsądnej długości i zaznacz przycięcie,
   - nie zapisuj nagłówków, tokenów sesji ani zmiennych środowiskowych,
   - przebieg zakończony błędem lub limitem kroków też ma mieć komplet kroków
     i właściwy status,
   - błąd zapisu śladu nie może przerwać zadania użytkownika.
4. Zaproponuj, skąd brać wersję instrukcji, i pokaż mi propozycję, zanim ją wdrożysz.

Nie czytaj i nie wypisuj wartości z plików .env. Nie zmieniaj zachowania agenta
ani treści instrukcji. Na końcu pokaż, jak sprawdzić jednym zapytaniem SQL,
że ostatni przebieg ma komplet kroków.
```

**Gotowe, gdy:** po jednym zadaniu na produkcji widzisz w bazie jeden wiersz w `runs` i tyle wierszy w `steps`, ile agent wykonał kroków, z czasem i kosztem.

<details>
<summary>🔬 Głębiej: co zapisać, żeby odtworzyć decyzję modelu, i co maskować</summary>

Ślad odpowiada na pytanie „dlaczego model zrobił X" tylko wtedy, gdy zawiera wszystko, co model **widział** w chwili decyzji. Samo „wejście: 2 400 tokenów" tego nie daje. Do odtworzenia decyzji potrzebujesz: wersji instrukcji (najlepiej skrótu jej treści, nie ręcznie podbijanego numeru), listy narzędzi z opisami w wersji z tamtego dnia, pełnej historii wiadomości przekazanej do tego wywołania, parametrów wywołania i identyfikatora modelu. Zmiana opisu narzędzia zmienia zachowanie tak samo jak zmiana instrukcji, a w śladach prawie nikt jej nie wersjonuje.

Napięcie jest oczywiste: im wierniejszy ślad, tym więcej cudzych danych przechowujesz. Kilka sposobów, żadnego idealnego:

- **Maskowanie przed zapisem** (wyrażenia regularne na e-maile, telefony, numery) jest tanie i dziurawe: nazwiska, adresy i treść sprawy przechodzą. Do tego maskowanie niszczy odtwarzalność: nie sprawdzisz już, czy model pomylił się przez literówkę w adresie.
- **Dwa poziomy retencji**: metadane i liczby (czas, koszt, status, nazwy narzędzi) trzymasz długo, treść wejść i wyjść krótko. Trendy zostają, dane osobowe znikają.
- **Treść tylko dla przebiegów ocenionych negatywnie albo zakończonych błędem.** Mniej danych, ale tracisz próbę kontrolną: nie porównasz złego przebiegu z dobrym.

Osobny problem: wyniki narzędzi bywają większym wyciekiem niż wiadomości użytkownika. `find_similar_offers` zwraca fragmenty dawnych ofert innych klientów i one też lądują w śladzie.

Pytanie do sporu: czy przycinać wyjście narzędzia do pierwszych N znaków, czy zapisywać skrót całości i próbkę? Przycięcie do początku gubi dokładnie te przypadki, w których model oparł się na czymś z końca długiego wyniku.

</details>

### Krok 4 — Zbuduj stronę z listą przebiegów

**Po co:** jeśli obejrzenie śladu wymaga pisania zapytań SQL, będziesz to robić rzadko. Strona, którą otwierasz jednym kliknięciem, sprawia, że czytasz ślady codziennie.

**Co robisz:** prosta strona w aplikacji, dostępna **tylko dla Ciebie**, z dwoma widokami:

1. **Lista przebiegów**: data, użytkownik, status, czas, koszt, liczba kroków, ocena. Najnowsze na górze, z filtrem po statusie.
2. **Szczegóły przebiegu**: kroki po kolei jak w drzewie z przykładu, z wejściem i wyjściem każdego kroku.

Nie musi być ładna. Ma odpowiadać na dwa pytania: „co poszło źle wczoraj?" i „co dokładnie zrobił agent w tym jednym przebiegu?".

**Prompt do asystenta:**

```text
Zbuduj prostą stronę do przeglądania śladów agenta z tabel runs i steps.

1. Widok listy: ostatnie przebiegi od najnowszego. Kolumny: data i godzina,
   użytkownik, status, czas, koszt, liczba kroków, ocena. Filtr po statusie.
2. Widok szczegółów po kliknięciu w przebieg: nagłówek (identyfikator, wersja
   instrukcji, model, czas, koszt) i kroki w kolejności, każdy z typem, nazwą,
   czasem, tokenami oraz rozwijanym wejściem i wyjściem.
3. Dostęp: tylko zalogowany właściciel. Zaproponuj, jak go rozpoznać w tym
   projekcie, i pokaż mi propozycję przed wdrożeniem. Każdy inny użytkownik
   i każde żądanie bez sesji ma dostać odmowę, także przy wejściu bezpośrednio
   na adres ze szczegółami.
4. Dane czytaj po stronie serwera. Nie wysyłaj do przeglądarki niczego ponad
   to, co widać na ekranie.

Użyj komponentów i stylu, które już są w projekcie; nie dodawaj nowych
bibliotek. Nie czytaj i nie wypisuj wartości z plików .env.
Na końcu podaj polecenie curl, którym sprawdzę, że strona odmawia bez sesji.
```

**Gotowe, gdy:** otwierasz dowolny przebieg z listy i widzisz jego pełny ślad z kosztem i czasem, a wejście na ten sam adres bez sesji kończy się odmową.

### Krok 5 — Patrz na cztery liczby

**Po co:** pojedynczy ślad mówi, co się stało raz. Liczby mówią, czy agent jako całość ma się lepiej, czy gorzej niż tydzień temu, zwłaszcza po tym, jak coś zmieniłeś.

**Co robisz:** z tabeli przebiegów wyprowadzasz i sprawdzasz regularnie:

| Liczba | Co mówi | Kiedy reagować |
|---|---|---|
| **odsetek przebiegów zakończonych błędem lub limitem kroków** | czy agent sobie radzi | rośnie po zmianie instrukcji lub modelu |
| **koszt na zadanie** (średni i najwyższy) | czy mieścisz się w budżecie | średni powyżej budżetu ze specyfikacji |
| **czas zadania** (mediana i 95. percentyl) | czy da się tego używać | percentyl powyżej limitu ze specyfikacji |
| **liczba kroków na zadanie** | czy agent błądzi | wyraźny wzrost bez zmiany typu zadań |

Zwykłe zapytanie SQL uruchamiane raz dziennie w zupełności wystarczy. Wzór masz w przykładzie Ofertownika. Budżet i limit czasu bierzesz z `docs/SPEC.md`.

Jak czytać wynik: w Ofertowniku trzy liczby są zielone, ale jeden przebieg miał 15 kroków przy średniej 7,2. To nie alarm, tylko zaproszenie: otwórz ten jeden ślad i sprawdź, czy agent szukał w kółko tego samego.

**Prompt do asystenta:**

```text
Przeczytaj docs/SPEC.md i schemat tabel runs i steps.

1. Napisz jedno zapytanie SQL liczące dla ostatnich 7 dni:
   - odsetek przebiegów zakończonych błędem lub limitem kroków,
   - koszt na zadanie: średni i najwyższy,
   - czas zadania: medianę i 95. percentyl,
   - liczbę kroków na zadanie: średnią i najwyższą.
2. Zapisz je w pliku sql/cztery_liczby.sql z komentarzem przy każdej kolumnie.
3. Dopisz w docs/RUNBOOK.md sekcję „Cztery liczby": gdzie uruchomić zapytanie
   oraz, dla każdej liczby, próg wzięty z docs/SPEC.md, po którym reaguję.
   Jeśli w specyfikacji nie ma progu, wpisz „DO USTALENIA" zamiast go wymyślać.

Nie łącz się z bazą i nie uruchamiaj zapytania: zrobię to w edytorze SQL
i wkleję Ci wynik. Nie czytaj i nie wypisuj wartości z plików .env.
```

**Gotowe, gdy:** jedno zapytanie zwraca cztery liczby dla Twojego agenta, a w runbooku jest zapisane, przy jakiej wartości każdej z nich reagujesz.

<details>
<summary>🔬 Głębiej: wersjonowanie instrukcji i porównywanie liczb między wersjami</summary>

Cztery liczby zyskują sens dopiero po pogrupowaniu po wersji instrukcji i modelu. „Odsetek błędów wzrósł z 4% do 9%" bez tej kolumny to zagadka; z nią to jedno `group by`.

Co uznać za wersję? Numer podbijany ręcznie zawsze kiedyś zostanie zapomniany. Pewniejszy jest skrót liczony z treści wszystkiego, co kształtuje zachowanie: instrukcji, opisów narzędzi, parametrów wywołania. Wtedy każda zmiana, także „drobna poprawka literówki w opisie narzędzia", automatycznie tworzy nową wersję.

Pułapki przy porównywaniu:

- **Inny ruch, nie inna wersja.** Jeśli wersja B działała w tygodniu, w którym przyszło więcej zapytań typu S4, wyjdzie drożej i wolniej bez żadnej winy instrukcji. Porównuj w obrębie tego samego typu zadania albo uruchamiaj obie wersje równolegle.
- **Mała próba.** Przy 30 przebiegach na wersję różnica 4% kontra 9% to jeden albo dwa przebiegi. Zanim wyciągniesz wniosek, policz, ilu przebiegów dotyczy.
- **Percentyl z małej próby.** 95. percentyl z 20 przebiegów to w praktyce drugi najgorszy wynik. Podawaj go razem z liczbą przebiegów.
- **Cicha zmiana po stronie dostawcy.** Ten sam identyfikator modelu może po pewnym czasie zachowywać się inaczej. Jeśli liczby zmieniły się bez Twojej zmiany, uruchom ewaluacje z Modułu 4 na niezmienionym kodzie i porównaj z ostatnim zapisanym wynikiem.

Rzecz do przemyślenia: ewaluacje mierzą na stałym zestawie, produkcja na płynącym ruchu. Kiedy obie miary się rozjeżdżają, zwykle rację ma produkcja, a zestaw ewaluacyjny przestał przypominać prawdziwy ruch.

</details>

### Krok 6 — Zbierz sygnał od ludzi

**Po co:** cztery liczby pokażą, że agent działa szybko i tanio. Nie pokażą, czy odpowiedź była dobra. To wie tylko człowiek, który ją dostał.

**Co robisz:**

1. Dodaj najprostszy mechanizm oceny: kciuk w górę i w dół z opcjonalnym komentarzem, zapisywany razem z identyfikatorem przebiegu.
2. Przy akcjach zatwierdzanych przez człowieka masz ten sygnał za darmo: **odrzucona propozycja to ocena negatywna**. W Ofertowniku Ania każdy szkic zatwierdza albo odrzuca; odrzucenie zapisuje się jako kciuk w dół, bez żadnego dodatkowego klikania.
3. Każdy negatywnie oceniony przebieg otwierasz, czytasz ślad i, zgodnie z zasadą z [Modułu 4](M4_EWALUACJE.md), zamieniasz na przypadek ewaluacyjny: nowy przykład w `evals/`, na którym agent ma odtąd przechodzić.

**Prompt do asystenta:**

```text
Dodaj ocenę odpowiedzi agenta przez użytkownika.

1. Pod każdą odpowiedzią końcową agenta pokaż dwa przyciski: kciuk w górę
   i kciuk w dół. Po kliknięciu kciuka w dół pokaż opcjonalne pole komentarza.
2. Ocenę i komentarz zapisz w tabeli runs przy właściwym przebiegu. Przygotuj
   plik SQL (albo migrację) dodający potrzebne kolumny.
3. Użytkownik może ocenić tylko własny przebieg; sprawdzaj to po stronie serwera.
   Żądanie bez sesji ma dostać odmowę.
4. Znajdź w kodzie miejsca, w których człowiek zatwierdza albo odrzuca propozycję
   agenta. Odrzucenie zapisuj automatycznie jako ocenę negatywną tego przebiegu.
5. Na stronie z listą przebiegów dodaj kolumnę z oceną i filtr „tylko negatywne".

Nie dodawaj nowych bibliotek. Nie czytaj i nie wypisuj wartości z plików .env.
Na końcu opisz, jak ręcznie sprawdzić, że ocena trafia do właściwego przebiegu.
```

**Gotowe, gdy:** kliknięcie kciuka w dół pojawia się na liście przebiegów przy właściwym przebiegu, a odrzucenie propozycji zapisuje ocenę negatywną samo.

<details>
<summary>🔬 Głębiej: próbkowanie przebiegów z produkcji do ewaluacji</summary>

Kciuki mają dwie wady: klika mały odsetek ludzi i klikają głównie niezadowoleni. Zestaw ewaluacyjny budowany wyłącznie z kciuków w dół opisuje porażki, których ludzie byli świadomi. Błędów, których nikt nie zauważył (dobrze brzmiąca oferta z nie tym terminem), tam nie ma.

Dlatego warto dołożyć **próbkę losową**: raz w tygodniu wylosuj 10 przebiegów, oceń je ręcznie tą samą rubryką co w Module 4 i dopisz najciekawsze do zestawu. Losowo nie znaczy „ostatnie dziesięć". Rozsądniej losować warstwami: kilka typowych, kilka z najwyższym kosztem lub liczbą kroków, kilka zakończonych odmową albo dopytaniem. Ogon rozkładu jest ciekawszy niż środek, ale tylko środek powie Ci, jak jest zwykle.

Trzy rzeczy, o które warto się pospierać:

- **Anonimizacja przed dopisaniem.** Przypadek z produkcji trafia do repozytorium i zostaje tam na zawsze, także w historii git. Dane osobowe usuń przed zapisem, nie po.
- **Oczekiwany wynik pisze człowiek.** Jeśli za „poprawną odpowiedź" uznasz to, co agent wtedy odpowiedział, zamrażasz obecne zachowanie razem z jego błędami.
- **Rozrost zestawu.** Każdy dopisany przypadek wydłuża i podraża każde uruchomienie ewaluacji. Po kilku miesiącach potrzebujesz podziału na szybki zestaw uruchamiany przy każdej zmianie i pełny uruchamiany przed wdrożeniem, a także odwagi, żeby usuwać przypadki, które nigdy niczego nie złapały.

</details>

### Krok 7 — Napisz runbook

**Po co:** awaria przychodzi wtedy, gdy jesteś zmęczony, zestresowany albo na telefonie. To najgorszy moment na przypominanie sobie, gdzie jest wyłącznik. Runbook piszesz spokojnie dziś, dla siebie o trzeciej w nocy.

**Co robisz:** tworzysz `docs/RUNBOOK.md` z czterema procedurami. Układ jak w runbooku Ofertownika wyżej:

1. **Agent nie odpowiada albo zwraca błędy**: sprawdź status dostawcy modeli i platformy → otwórz ostatnie przebiegi ze statusem błędu → jeśli zawiniło ostatnie wdrożenie, wycofaj je do poprzedniego.
2. **Koszt rośnie nienormalnie**: włącz wyłącznik awaryjny → znajdź źródło zapytaniem o koszt według użytkownika i godziny → zablokuj konto albo obniż limit.
3. **Podejrzenie wycieku klucza lub sekretu**: wyłącznik awaryjny → klucz kursowy: natychmiast zgłoś prowadzącemu; własne klucze: unieważnij i wygeneruj nowe → zmień sekrety sesji, wygeneruj nowe klucze bazy, wdróż ponownie → sprawdź historię git i logi pod kątem sekretu.
4. **Agent zrobił coś, czego nie powinien**: zapisz identyfikator przebiegu → wyłącz narzędzie, którego dotyczy problem → dodaj przypadek do ewaluacji, zanim naprawisz.

Zasada pisania: każdy punkt ma mówić **gdzie kliknąć albo co wpisać**, a nie co osiągnąć.

- ❌ „Wyłącz agenta."
- ✅ „W panelu platformy → ustawienia projektu → zmienne środowiskowe ustaw `<nazwa zmiennej wyłącznika>` na `1` i uruchom ponowne wdrożenie."

**Prompt do asystenta:**

```text
Napisz docs/RUNBOOK.md dla tego projektu (jeśli plik istnieje, uzupełnij go,
niczego nie kasując). Przeczytaj najpierw docs/BEZPIECZENSTWO.md,
docs/ARCHITEKTURA.md, kod wyłącznika awaryjnego i limitów oraz schemat tabel
runs i steps.

Runbook ma zawierać cztery procedury, każda jako numerowane kroki:
1. Agent nie odpowiada albo zwraca błędy.
2. Koszt rośnie nienormalnie (z gotowym zapytaniem SQL: koszt według
   użytkownika i godziny z ostatnich 24 godzin).
3. Podejrzenie wycieku klucza lub sekretu. Zapisz wprost: wyciek klucza
   kursowego zgłaszam natychmiast prowadzącemu; własne klucze unieważniam
   i generuję nowe.
4. Agent zrobił coś, czego nie powinien (z krokiem „dodaj przypadek do
   ewaluacji, zanim naprawisz").

Zasady:
- Każdy krok mówi konkretnie, gdzie kliknąć albo co wpisać: nazwa zmiennej,
  plik, adres strony. Nazwy bierz z kodu, nie wymyślaj ich.
- Jeśli czegoś nie da się ustalić z repozytorium (np. gdzie w panelu platformy
  wycofuje się wdrożenie), wpisz „DO UZUPEŁNIENIA: …" i wypisz mi te miejsca
  na końcu jako pytania.
- Ma to umieć wykonać osoba, która nie pisała tego kodu.

Nie czytaj i nie wypisuj wartości z plików .env. Nie zmieniaj kodu.
```

**Gotowe, gdy:** `docs/RUNBOOK.md` zawiera cztery procedury i żadnego „DO UZUPEŁNIENIA".

### Krok 8 — Przećwicz każdą procedurę na sucho

**Po co:** runbook, którego nikt nie przeszedł, jest listą życzeń. Dopiero próba pokazuje, że zmienna nazywa się inaczej, zapytanie ma literówkę, a przycisku do wycofania wdrożenia szukasz dziesięć minut.

**Co robisz:** każdą z czterech procedur wykonujesz raz, na prawdziwym środowisku, w kontrolowany sposób:

| Procedura | Jak ją przećwiczyć bez szkody |
|---|---|
| błędy | wdróż drobną, nieszkodliwą zmianę (np. w tekście strony), a potem wycofaj wdrożenie do poprzedniego |
| koszt | włącz wyłącznik, sprawdź odmowę, uruchom zapytanie o koszt, wyłącz wyłącznik |
| wyciek | wymień jeden **własny** sekret (np. sekret sesji) na nowy i wdróż ponownie; klucza kursowego nie ruszaj |
| niepożądane działanie | wyłącz jedno narzędzie, sprawdź, że agent sobie bez niego radzi albo czytelnie odmawia, włącz z powrotem |

Przy każdej próbie zmierz czas i zapisz, co w instrukcji było niejasne. Popraw runbook od razu.

**Prompt do asystenta:**

```text
Przećwiczę na sucho procedury z docs/RUNBOOK.md. Poprowadź mnie przez nie
po jednej. Ty niczego nie wykonujesz i niczego nie zmieniasz na produkcji.

Dla każdej z czterech procedur:
1. Zaproponuj bezpieczny sposób próby, który nie zaszkodzi użytkownikom i nie
   wymaga prawdziwej awarii.
2. Podawaj mi po JEDNYM kroku z runbooka i czekaj, aż napiszę, co się stało.
3. Gdy krok okaże się niejasny, nieaktualny albo niewykonalny, zanotuj to.

Po każdej procedurze popraw jej treść w docs/RUNBOOK.md według notatek
i dopisz pod nią linijkę: „Przećwiczone: [data], czas: [ile minut]".

Nie czytaj i nie wypisuj wartości z plików .env. Nie proś mnie o wklejanie
kluczy ani haseł: jeśli potrzebujesz potwierdzenia, zapytaj „czy zadziałało".
```

**Gotowe, gdy:** pod każdą z czterech procedur jest data próby i czas jej wykonania.

<details>
<summary>🔬 Głębiej: stopniowe wdrażanie zmian w instrukcjach</summary>

Zmiana jednego zdania w instrukcji jest wdrożeniem, choć nie wygląda jak zmiana kodu. Potrafi zepsuć zachowanie, którego nie pokrywa żaden przypadek w ewaluacjach. Stąd pomysł: nową wersję instrukcji włącz najpierw dla części przebiegów i porównaj wyniki, zanim przełączysz wszystkich.

Praktyczne pytania, które pojawiają się od razu:

- **Jak dzielić ruch.** Losowanie przy każdym przebiegu daje czyste porównanie, ale ten sam użytkownik dostaje raz jedno zachowanie, raz drugie. Przydział po użytkowniku jest spójny dla ludzi, ale przy kilkunastu użytkownikach grupy są nieporównywalne.
- **Co porównywać.** Cztery liczby plus odsetek ocen negatywnych, pogrupowane po wersji. Ustal z góry, co uznasz za powód do wycofania; inaczej każdą różnicę da się potem wytłumaczyć.
- **Jak cofać.** Wycofanie wersji instrukcji powinno być zmianą jednej wartości, nie nowym wdrożeniem kodu. To argument za trzymaniem „która wersja jest aktywna" w zmiennej środowiskowej albo w bazie. Kosztem jest to, że instrukcja przestaje być w pełni opisana przez commit.
- **Tryb cienia.** Zamiast pokazywać nową wersję ludziom, uruchamiasz ją równolegle na tych samych wejściach i zapisujesz wynik tylko do śladu. Zero ryzyka dla użytkownika, podwójny koszt. I uwaga na narzędzia zapisujące: w cieniu nie mogą się wykonać naprawdę, bo Ofertownik zapisałby dwa szkice do jednego zapytania.

Przy ruchu rzędu kilkudziesięciu przebiegów tygodniowo stopniowe wdrażanie nie da wyniku, któremu można ufać. Uczciwszą bramką jest wtedy pełny zestaw ewaluacji przed wdrożeniem i uważne czytanie pierwszych dwudziestu śladów po nim.

</details>

### Krok 9 — Daj agenta trzem osobom i przeczytaj ich ślady

**Po co:** Ty wiesz, jak z agentem rozmawiać, więc nieświadomie omijasz jego słabe miejsca. Obca osoba trafi w nie w pierwszej minucie.

**Co robisz:**

1. Wybierz trzy osoby, najlepiej takie, dla których agent jest przeznaczony. Dla Ofertownika: Ania i dwie osoby z jej firmy, które też odpowiadają na zapytania.
2. Daj im adres i **jedno zdanie** instrukcji, np. „Wklej treść zapytania od klienta, dostaniesz szkic oferty do sprawdzenia". Nic więcej. Nie siedź obok i nie podpowiadaj.
3. Uprzedź, że rozmowy są zapisywane, i poproś o przykłady bez prawdziwych danych osobowych.
4. Potem przeczytaj ich ślady. Nie pytaj tylko o wrażenia: ludzie mówią „fajne", a ślad pokazuje, że trzy razy przeformułowali pytanie.
5. Zapisz w `docs/DZIENNIK.md`, co zrobili inaczej, niż zakładały scenariusze ze specyfikacji.

Przykład wpisu z Ofertownika: „Handlowiec wkleił trzy zapytania naraz w jednej wiadomości. Agent przygotował szkic tylko dla pierwszego i nie wspomniał o pozostałych. Żaden scenariusz S1–S8 tego nie przewiduje → nowy przypadek w `evals/`, decyzja do `docs/POTEM.md`".

**Prompt do asystenta:**

```text
Trzy osoby użyły agenta na produkcji. Poniżej wklejam ich przebiegi
wyeksportowane z tabel runs i steps (bez danych osobowych): [wklej].

Przeczytaj docs/SPEC.md i porównaj to, co zrobili użytkownicy, ze scenariuszami
ze specyfikacji:
1. Które przebiegi pasują do znanych scenariuszy, a które do żadnego?
2. Co użytkownicy zrobili inaczej, niż zakładała specyfikacja (sposób pisania,
   kolejność, kilka spraw naraz, nieoczekiwane prośby)?
3. W których przebiegach agent błądził: powtarzał to samo narzędzie, robił
   wyraźnie więcej kroków niż zwykle, kończył limitem albo błędem?
4. Zaproponuj najwyżej 5 nowych przypadków do evals/. Dla każdego: wejście
   i oczekiwane zachowanie do mojej akceptacji. Nie dopisuj ich sam.

Dopisz do docs/DZIENNIK.md wpis z dzisiejszą datą: „Pierwsi użytkownicy:
co zrobili inaczej, niż zakładałem" (najwyżej 10 punktów).
Nie zmieniaj kodu ani instrukcji agenta.
```

**Gotowe, gdy:** w bazie są przebiegi co najmniej 3 osób innych niż Ty, a `docs/DZIENNIK.md` ma wpis z wnioskami.

## Najczęstsze potknięcia

- **Ekran logowania jest, endpoint stoi otworem.** Objaw: strona wymaga zalogowania, ale `curl` na adres API dostaje odpowiedź modelu. → Sprawdzaj sesję w samym endpoincie, po stronie serwera, i powtórz test z kroku 2.
- **Logowanie na produkcji wraca na `localhost`.** Objaw: po zalogowaniu przeglądarka pokazuje błąd połączenia. → Popraw adresy powrotu logowania w panelu dostawcy logowania i w zmiennych środowiskowych.
- **Ślad urywa się na błędzie.** Objaw: przebiegi ze statusem błędu mają zero kroków. → Zapisuj kroki na bieżąco, a nie dopiero po zakończeniu zadania.
- **W śladach są rzeczy, których nie powinno tam być.** Objaw: w kolumnie wejścia widzisz nagłówki żądań, tokeny albo pełne dane klientów. → Zapisuj tylko treść wiadomości i argumenty narzędzi, dodaj maskowanie, skasuj dotychczasowe wiersze.
- **Runbook z nazwami „z głowy".** Objaw: w czasie próby zmienna nazywa się inaczej niż w instrukcji. → Nazwy przepisuj z kodu i z panelu, a po każdej zmianie przechodź procedurę jeszcze raz.

## ✅ Sprawdź, czy masz wszystko

- [ ] Adres produkcyjny działa; żądanie bez sesji wysłane bezpośrednio na endpoint API dostaje odmowę.
- [ ] Produkcja ma osobną bazę i nowe sekrety; schemat bazy jest w repozytorium jako pliki SQL lub migracje.
- [ ] Lista kontrolna z kroku 2 jest odhaczona w całości.
- [ ] Dla dowolnego przebiegu potrafisz pokazać pełny ślad z kosztem i czasem.
- [ ] Ślad zawiera wersję instrukcji i modelu.
- [ ] Wiesz, co zapisujesz w śladach i jak długo, a użytkownicy są o tym poinformowani.
- [ ] Jedno zapytanie zwraca cztery liczby.
- [ ] Jest kciuk w górę i w dół; odrzucona propozycja zapisuje ocenę negatywną.
- [ ] `docs/RUNBOOK.md` zawiera cztery procedury, każda przećwiczona.
- [ ] W bazie są przebiegi co najmniej 3 osób innych niż Ty; `docs/DZIENNIK.md` ma wpis z wnioskami.

## 🤖 Poproś asystenta o recenzję

```text
Przeczytaj docs/RUNBOOK.md, pliki SQL ze schematem tabel runs i steps, kod
zapisujący ślady oraz przykładowy ślad, który wklejam poniżej: [wklej jeden
przebieg bez danych osobowych].

Oceń jak osoba, która ma przejąć dyżur przy tym systemie. Dla każdego punktu
napisz PASS albo FAIL i jedno zdanie uzasadnienia:

1. Czy na podstawie śladu da się odtworzyć, dlaczego agent podjął każdą decyzję?
2. Czy ślad zawiera wersję instrukcji i modelu, żeby dało się porównać
   zachowanie przed zmianą i po niej?
3. Czy w śladach lub logach mogą znaleźć się sekrety albo dane, których nie
   powinno się przechowywać?
4. Czy procedury w runbooku wykona ktoś, kto nie pisał tego kodu?
5. Jakiej sytuacji awaryjnej brakuje?

Nie czytaj i nie wypisuj wartości z plików .env. Niczego nie zmieniaj w plikach.
Na końcu podaj jedną rzecz, którą poprawiłbyś w pierwszej kolejności.
```

## 🚀 Dla chętnych

- 🔵 **Cotygodniowy przegląd.** Raz w tygodniu wylosuj 10 przebiegów, oceń je ręcznie i dopisz najciekawsze do zestawu w `evals/`. Zapisz w dzienniku, ile z nich oceniłeś inaczej, niż sugerowały kciuki.
- 🔵 **Ostrzeżenie zamiast sprawdzania.** Spraw, żeby zapytanie o cztery liczby uruchamiało się samo raz dziennie i dawało Ci znać, gdy któraś przekroczy próg. O zadaniach uruchamianych według harmonogramu mówi [T2: Praca w tle](T2_PRACA_W_TLE.md).
- 🟣 **Wersje instrukcji w liczbach.** Zapisuj w śladzie skrót treści instrukcji i opisów narzędzi. Zmień jedno zdanie instrukcji, zbierz przebiegi na obu wersjach i porównaj cztery liczby. Zanim spojrzysz na wynik, zapisz, jaką różnicę uznasz za prawdziwą, a jaką za szum, i ile przebiegów potrzebujesz, żeby to rozstrzygnąć.
- 🟣 **Stopniowe wdrożenie.** Nową wersję instrukcji włącz dla części przebiegów. Zaprojektuj podział ruchu, kryterium wycofania i sposób cofnięcia bez wdrażania kodu. Opisz w `docs/DZIENNIK.md`, przy jakim ruchu ta metoda w Twoim projekcie zaczyna dawać wynik, któremu ufasz.

## Co dalej

Agent działa dla ludzi i widzisz, co robi. Zostało pokazać go innym i wyciągnąć wnioski: [Moduł 8: Demo i retrospektywa](M8_DEMO.md).
