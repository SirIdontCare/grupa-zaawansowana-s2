# Moduł techniczny 2: Praca w tle — harmonogram, kolejka, długie zadania

> ⏱️ Ok. 5 godzin. Potrzebujesz działającej najmniejszej wersji agenta z [Modułu 5](M5_MVP.md) — z limitem kroków, limitem czasu narzędzi i zatwierdzaniem przez człowieka.
>
> 🧩 **Moduł do wyboru.** Weź go, jeśli Twój agent ma działać **bez otwartej karty przeglądarki**: o stałej porze, w reakcji na zdarzenie albo nad zadaniem dłuższym niż jedno żądanie HTTP. Najlepszy moment: po [Module 5](M5_MVP.md), przed [Modułem 7](M7_WDROZENIE.md). Jeśli Twój agent odpowiada tylko wtedy, gdy ktoś do niego pisze — pomiń ten moduł.

## O co chodzi

Agent, którego zbudowałeś, żyje dokładnie tyle, ile trwa jedno żądanie: użytkownik pisze, agent odpowiada, połączenie się zamyka. To jak rozmowa przy okienku w urzędzie — stoisz, czekasz, dostajesz odpowiedź, odchodzisz.

To wystarcza do czatu. Nie wystarcza, gdy agent ma co rano przejrzeć nowe maile, przetworzyć dwieście dokumentów albo poczekać dobę na czyjąś akceptację. Wtedy potrzebujesz innego modelu: **zostawiasz zlecenie i odbierasz wynik później**. Jak w punkcie ksero albo w pralni.

A żeby to działało, potrzebne są trzy rzeczy, których przy okienku nie było: **numerek** (żeby wiedzieć, które zlecenie jest Twoje), **status** (przyjęte, w trakcie, gotowe, nie udało się) i **ktoś, kto pilnuje, żeby tego samego zlecenia nie wykonać dwa razy** — bo pracownik mógł zachorować w połowie, a kolega nie wie, co już zrobiono.

Agent przestaje więc być funkcją wywoływaną z przeglądarki, a staje się **zadaniem**: czymś, co ma stan, może się przerwać w połowie, może zostać uruchomione dwa razy i czego nikt nie ogląda na żywo. Jest przy tym jedna różnica względem zwykłych programów: każde powtórzenie kroku agenta **kosztuje pieniądze i może dać inny wynik**.

## Po tym module masz

- wybrany **jeden z czterech wzorców** pracy w tle, pasujący do Twojego zadania,
- tabelę `jobs` w bazie i jedno zadanie, które uruchamia się bez przeglądarki,
- zadanie odporne na podwójne uruchomienie i na przerwanie w połowie — sprawdzone dwoma testami,
- widok zadań dla użytkownika ze statusem, kosztem i przyciskiem Anuluj,
- limit kosztu na każde zadanie,
- plik `docs/ZADANIA_W_TLE.md` opisujący zadanie.

## Słowa, które tu padną

| Słowo | Co to znaczy | Porównanie z życia |
|---|---|---|
| **Zadanie w tle** (job) | praca, którą program wykonuje bez czekającego użytkownika; ma swój wiersz w bazie | zlecenie w pralni z numerkiem |
| **Limit czasu funkcji** | najdłuższy czas, przez jaki platforma pozwala działać jednemu wywołaniu Twojego kodu; potem je ucina | parkometr: po opłaconym czasie auto odholowują, choćbyś był w połowie zakupów |
| **Harmonogram** (cron) | mechanizm, który uruchamia coś o ustalonej porze | budzik nastawiony na 7:00 |
| **Kolejka** | lista wiadomości „do zrobienia"; jedne programy je wkładają, inne wyjmują i wykonują | tacka z zamówieniami w kuchni restauracji |
| **Pracownik** (worker) | program, który wyjmuje wiadomości z kolejki i wykonuje pracę | kucharz biorący kolejne zamówienie z tacki |
| **Okno niewidoczności** | czas, na który wiadomość wyjęta przez jednego pracownika znika dla pozostałych | kucharz zabiera karteczkę z tacki; jeśli nie wyda dania w kwadrans, karteczka wraca |
| **Ponowienie** (retry) | kolejna próba wykonania tego samego po błędzie | drugi telefon, gdy pierwszy nie doszedł |
| **Idempotencja** | cecha operacji: wykonana dwa razy daje ten sam skutek, co wykonana raz | przycisk windy: naciśnięty dwa razy nie zamawia dwóch wind |
| **Klucz idempotencji** | unikalna nazwa zlecenia, po której system poznaje, że już je ma | numer zamówienia: drugi raz z tym samym numerem sklep nie przyjmie |
| **Trwały przepływ** (durable workflow) | sposób pisania wieloetapowych zadań, w którym wynik każdego kroku jest zapisywany, więc po awarii praca rusza od miejsca przerwania | gra z zapisem stanu po każdym poziomie |
| **Webhook** | zapytanie, które obca usługa sama wysyła do Twojej aplikacji, gdy coś się u niej wydarzy | SMS od kuriera „paczka czeka" — nie musisz dzwonić i pytać |

## Przykład: Ofertownik

Przypomnienie: Ofertownik przygotowuje szkice ofert dla firmy szkoleniowej Ani. Po [T1](T1_INTEGRACJE.md) umie czytać jej skrzynkę Gmail. Ale Ania nadal musi usiąść, otworzyć aplikację i kliknąć. Chcemy inaczej: **co rano o 7:00 Ofertownik sam przegląda nowe maile i przygotowuje szkice**, a Ania o 8:00 siada do gotowej listy.

**Wzorzec:** harmonogram + kolejka. Harmonogram mówi „już czas", kolejka rozdaje pracę po jednym mailu.

**Jak wygląda jeden poranek:**

1. Około 7:00 platforma wywołuje adres `/api/cron/morning-review`. Kod sprawdza sekret.
2. Kod wstawia do tabeli `jobs` zadanie „poranny przegląd" z kluczem `morning_review:<id Ani>:2026-11-19`. Jeśli harmonogram odpali się drugi raz tego samego dnia, baza odrzuci drugi wiersz — klucz już istnieje.
3. Pracownik bierze to zadanie, pobiera listę nowych maili i dla **każdego maila** wstawia osobne zadanie z kluczem `draft_offer:<id Ani>:<identyfikator maila>` oraz wiadomość do kolejki. Dziś jest ich 14.
4. Pracownik, uruchamiany co minutę, wyjmuje z kolejki po kilka wiadomości. Dla każdego maila: ocenia, czy to zapytanie ofertowe → **zapisuje wynik** → uruchamia agenta → **zapisuje szkic** przez `save_offer_draft` → oznacza zadanie jako `done` i usuwa wiadomość z kolejki.
5. Niczego nie wysyła. Wysyłka nadal wymaga kliknięcia Ani.

**Co widzi Ania o 8:00:**

| Zapytanie | Status | Koszt | |
|---|---|---|---|
| Excel dla 12 osób, październik | ✅ szkic gotowy | 0,03 USD | Otwórz |
| Program dla trzech działów | ✅ szkic gotowy | 0,08 USD | Otwórz |
| Szkolenie bez terminu i liczby osób | ❓ pytania do klienta gotowe | 0,02 USD | Otwórz |
| Reklamacja faktury | ↪️ to nie zapytanie — do przekazania | 0,01 USD | Otwórz |
| Zapytanie z dużym załącznikiem | ❌ nie udało się (3 próby) | 0,04 USD | Spróbuj ponownie |
| … | ⏳ w trakcie | — | Anuluj |

**Budżet:** jedno zadanie „szkic dla jednego maila" może kosztować najwyżej 0,10 USD (dwa razy więcej niż cel 0,05 USD z briefu — zapas na trudne przypadki). Po przekroczeniu zadanie kończy się statusem `failed`, a nie mieli dalej.

**Wypełniony `docs/ZADANIA_W_TLE.md`:**

```markdown
## Poranny przegląd skrzynki
**Wyzwalacz:** harmonogram codziennie rano (cel: 7:00 czasu Ani; na planie
  darmowym godzina jest przybliżona)
**Wzorzec:** harmonogram + kolejka
**Klucz idempotencji:** morning_review:<owner_id>:<data>
  oraz dla każdego maila: draft_offer:<owner_id>:<identyfikator maila>
**Kroki i co jest zapisywane po każdym:**
  1. pobranie listy nowych maili → identyfikatory maili w jobs.result przeglądu
  2. utworzenie zadań draft_offer i wiadomości w kolejce → wiersze w jobs
  3. (dla maila) ocena, czy to zapytanie ofertowe → jobs.result.classification
  4. (dla maila) przebieg agenta → szkic w tabeli ofert (save_offer_draft),
     jego identyfikator w jobs.result.draft_id, koszt w jobs.cost_usd
**Limity:** budżet 0,10 USD na zadanie draft_offer, 3 próby,
  1 przegląd na użytkownika na dobę, najwyżej 30 zadań draft_offer na dobę
**Co widzi użytkownik:** lista zadań ze statusem, wynikiem i kosztem;
  powiadomienie w aplikacji „przegląd gotowy: 11 szkiców, 2 do decyzji, 1 błąd";
  przycisk Anuluj przy zadaniach w toku
**Co się dzieje przy awarii w połowie:** wiadomość wraca do kolejki po upływie
  okna niewidoczności (300 s). Pracownik sprawdza jobs.result: jeśli jest już
  classification, nie ocenia ponownie; jeśli jest draft_id, oznacza zadanie
  jako done bez wywołania modelu. Po 3 próbach: wiadomość do archiwum,
  status failed, Ania widzi „Spróbuj ponownie".
**Czego zadanie NIE robi:** nie wysyła ofert. Wysyłka to osobne zadanie,
  tworzone dopiero po zatwierdzeniu szkicu przez Anię.
```

## Kroki

### Krok 1 — Rozpoznaj swój przypadek

**Po co:** są cztery wzorce pracy w tle, od prostego do złożonego. Wybranie za prostego kończy się zadaniem uciętym w połowie. Wybranie za złożonego — tygodniem konfiguracji dla czegoś, co mógł zrobić budzik.

**Co robisz:** przeczytaj cztery wzorce po kolei i wybierz **najprostszy**, który wystarcza.

**A. Odpowiedz od razu, dokończ po odpowiedzi.**
Endpoint tworzy wiersz w `jobs`, odsyła `job_id` i kończy pracę już po wysłaniu odpowiedzi. W Next.js służy do tego `after()` z `next/server`; na innych platformach szukaj odpowiednika (`waitUntil`). Interfejs odpytuje o status zadania.
Ograniczenie: praca po odpowiedzi **nadal mieści się w limicie czasu funkcji**. To sposób na to, żeby użytkownik nie czekał z otwartym połączeniem — nie na zadania wielogodzinne.
*Dla Ofertownika wyglądałoby to tak:* Ania wkleja bardzo długie zapytanie dla trzech działów (S4), od razu widzi „przyjęte, pracuję", a szkic pojawia się po dwóch minutach — bez trzymania otwartego połączenia.

**B. Harmonogram.**
Platforma sama wywołuje Twój endpoint o ustalonej porze. Nikt niczego nie klika.
*Dla Ofertownika wyglądałoby to tak:* codziennie około 7:00 platforma woła `/api/cron/morning-review` — to jest nasz przypadek.

**C. Kolejka.**
Jeden kawałek kodu (producent) wkłada wiadomości „do zrobienia", inny (pracownik) wyjmuje je i przetwarza po jednej. Dobre, gdy elementów jest wiele i są od siebie niezależne, albo gdy praca przychodzi z zewnątrz w nieprzewidywalnych momentach.
*Dla Ofertownika wyglądałoby to tak:* każdy z 14 porannych maili to osobna wiadomość w kolejce; awaria przy mailu nr 5 nie zatrzymuje pozostałych trzynastu.

**D. Trwały przepływ (durable workflow).**
Biblioteki tego typu zapisują wynik każdego kroku, więc po awarii albo po wielogodzinnej przerwie wykonanie wznawia się od ostatniego ukończonego kroku, a nie od początku. Dają też wbudowane czekanie („uśpij na 7 dni") i wznawianie po sygnale z zewnątrz — czyli dokładnie to, czego potrzebuje zatwierdzanie przez człowieka rozłożone w czasie. W ekosystemie Vercela jest to Workflow SDK (dyrektywy `"use workflow"` i `"use step"`); istnieją też rozwiązania niezależne od platformy.
Dwie rzeczy do zapamiętania: **pojedynczy krok nadal podlega limitowi czasu funkcji**, a kroki są automatycznie ponawiane — więc każdy krok wywołujący model lub narzędzie zapisujące musi być idempotentny (krok 3).
*Dla Ofertownika wyglądałoby to tak:* jeden przepływ „przygotuj szkic → czekaj na decyzję Ani, choćby trzy dni → wyślij → po tygodniu sprawdź, czy klient odpisał".

Teraz tabela wyboru:

| Przypadek | Przykład | Wzorzec |
|---|---|---|
| zadanie trwa dłużej, niż użytkownik chce patrzeć, ale mieści się w limicie funkcji | analiza długiego dokumentu, 2–4 minuty | **odpowiedz od razu, dokończ po odpowiedzi** |
| ma się wydarzyć o stałej porze | poranny przegląd, cotygodniowy raport | **harmonogram** |
| wiele niezależnych elementów albo reakcja na zdarzenie z zewnątrz | 200 dokumentów do przetworzenia, webhook o nowym zgłoszeniu | **kolejka** |
| wiele kroków, przerwy liczone w godzinach, oczekiwanie na człowieka | proces z akceptacją następnego dnia | **trwały przepływ (durable workflow)** |

Wybierz **najprostszy**, który wystarcza. Trwały przepływ dla zadania raz dziennie to nadmiar. Wzorce można łączyć: Ofertownik używa B i C razem.

Zanim pójdziesz dalej, sprawdź limity platformy, na której wdrażasz — to one wyznaczają granice między wzorcami. Dla polecanego stacku (stan na listopad 2026, sprawdź aktualne wartości w dokumentacji):

| Platforma | Limit |
|---|---|
| funkcja na Vercelu, plan Hobby | do 300 s na wywołanie; przekroczenie kończy się błędem 504 |
| zadania cykliczne na Vercelu, plan Hobby | najwyżej **raz na dobę**, z dokładnością do godziny |
| Supabase Edge Function, plan darmowy | do 150 s czasu zegarowego, 2 s czasu procesora na żądanie |
| Supabase Cron | zalecane: zadanie do 10 minut, nie więcej niż 8 jednocześnie |

Jak to czytać na przykładzie: 14 maili po około 60 sekund to 14 minut pracy. Jedno wywołanie funkcji ma 300 sekund. Poranny przegląd **nie zmieści się** w jednym wywołaniu — dlatego Ofertownik potrzebuje kolejki, a nie samego harmonogramu.

**Prompt do asystenta:**

```text
Pomóż mi wybrać wzorzec pracy w tle dla mojego agenta.
Przeczytaj docs/SPEC.md i docs/ARCHITEKTURA.md. Niczego nie twórz w kodzie.

Zadawaj mi po JEDNYM pytaniu naraz:
1. Co ma się dziać bez udziału użytkownika i co to uruchamia: pora dnia,
   zdarzenie z zewnątrz czy kliknięcie użytkownika?
2. Ile elementów przetwarzamy w jednym uruchomieniu i ile trwa jeden?
3. Czy elementy są od siebie niezależne?
4. Czy gdzieś czekamy na człowieka albo na coś, co trwa godziny?
5. Na jakiej platformie i jakim planie wdrażam?

Potem:
- sprawdź w aktualnej dokumentacji tej platformy limit czasu funkcji
  i ograniczenia zadań cyklicznych dla mojego planu; nie podawaj limitów z pamięci,
- policz, czy cała praca mieści się w jednym wywołaniu funkcji,
- zaproponuj NAJPROSTSZY z czterech wzorców (odpowiedz od razu i dokończ
  po odpowiedzi / harmonogram / kolejka / trwały przepływ) albo ich połączenie
  i uzasadnij jednym zdaniem, dlaczego prostszy nie wystarczy.

Zapisz decyzję w nowym pliku docs/ZADANIA_W_TLE.md w polach „Wyzwalacz" i „Wzorzec".
```

**Gotowe, gdy:** w `docs/ZADANIA_W_TLE.md` jest wybrany wzorzec i jedno zdanie, dlaczego prostszy nie wystarczy.

<details>
<summary>🔬 Głębiej: kiedy trwały przepływ jest wart swojej złożoności</summary>

Tabela `jobs` z kolejką to w gruncie rzeczy ręcznie napisany, ubogi silnik przepływów. Pytanie brzmi, w którym momencie przestaje się opłacać go dopisywać.

Sygnały, że czas na trwały przepływ:

- **stan zadania przestaje mieścić się w kolumnie `status`.** Gdy pojawia się `waiting_approval_2`, `after_reminder`, `retrying_step_4`, piszesz własną maszynę stanów i będziesz ją debugować o trzeciej w nocy,
- **czekanie jest częścią logiki**, a nie wyjątkiem: „czekaj na decyzję, ale po 48 godzinach przypomnij, a po 7 dniach anuluj",
- **kroki mają zależności**: wynik kroku 2 decyduje, czy wykonać krok 3a, czy 3b, a każdy z nich może się nie udać niezależnie,
- **kompensacje**: jeśli krok 4 się nie uda, trzeba cofnąć skutki kroków 2 i 3.

Co dostajesz w zamian za złożoność: zapis wyniku każdego kroku bez własnego kodu, wznawianie, czekanie i sygnały z zewnątrz jako elementy języka, a zwykle także podgląd wykonania krok po kroku.

Czym płacisz:

- **determinizm kodu przepływu.** Silnik odtwarza przepływ od początku, podstawiając zapisane wyniki kroków. Wszystko, co niedeterministyczne (czas, losowość, wywołanie modelu, sieć), musi siedzieć **wewnątrz kroku**, nie między krokami. To najczęstsze źródło dziwnych błędów u początkujących.
- **wersjonowanie.** Przepływ uruchomiony w poniedziałek może się wznowić w czwartek, po wdrożeniu nowego kodu. Co wtedy — stara czy nowa definicja? Każdy silnik odpowiada na to inaczej i trzeba tę odpowiedź znać przed pierwszą zmianą na produkcji.
- **automatyczne ponowienia.** Są wygodne i jednocześnie groźne dla agentów: krok, który wywołał model za 0,08 USD i padł przy zapisie, zostanie ponowiony z kolejnym wywołaniem. Idempotencja kroków nie jest tu opcją.
- **związanie z platformą** albo utrzymanie własnej infrastruktury silnika.

Dla Ofertownika rachunek wygląda tak: poranny przegląd to płaska lista niezależnych elementów — kolejka wystarcza i jest prostsza. Gdyby jednak doszło „po zatwierdzeniu wyślij, po trzech dniach bez odpowiedzi przygotuj przypomnienie, po zatwierdzeniu przypomnienia wyślij", przepływ zaczyna wygrywać. Granica nie leży w liczbie kroków, tylko w tym, **ile różnych rodzajów czekania** masz między nimi.

</details>

### Krok 2 — Zaprojektuj stan zadania

**Po co:** to jest numerek z pralni. Niezależnie od wzorca, zadanie w tle ma wiersz w bazie i ten wiersz jest jego **jedynym źródłem prawdy**. Nie pamięć procesu, nie log, nie kolejka — wiersz.

**Co robisz:** tworzysz tabelę `jobs`.

```sql
create table jobs (
  id uuid primary key default gen_random_uuid(),
  owner_id uuid not null,
  kind text not null,                 -- 'morning_review', 'ingest_document', ...
  idempotency_key text unique,        -- np. 'morning_review:<owner>:2026-11-19'
  status text not null default 'queued',   -- queued | running | waiting_approval | done | failed | cancelled
  input jsonb not null,
  result jsonb,
  error text,
  attempts int not null default 0,
  cost_usd numeric not null default 0,
  created_at timestamptz not null default now(),
  started_at timestamptz,
  finished_at timestamptz
);
```

Co znaczą kolumny, które nie tłumaczą się same:

| Kolumna | Do czego służy |
|---|---|
| `kind` | rodzaj zadania; w Ofertowniku `morning_review` i `draft_offer` |
| `idempotency_key` | unikalna nazwa zlecenia; słowo `unique` sprawia, że baza nie przyjmie drugiego wiersza z tym samym kluczem |
| `status` | `queued` (czeka), `running` (ktoś nad nim pracuje), `waiting_approval` (czeka na człowieka), `done`, `failed`, `cancelled` |
| `input` / `result` | z czym zadanie startuje i co po sobie zostawia — także wyniki pośrednie |
| `attempts` | ile razy już próbowano |
| `cost_usd` | suma kosztu wszystkich wywołań modelu w tym zadaniu |

Powiąż tabelę ze śladami z [Modułu 7](M7_WDROZENIE.md) (ślad to zapis przebiegu agenta krok po kroku): każdy przebieg agenta w ramach zadania zapisuje `job_id`. Jeśli Modułu 7 jeszcze nie robiłeś — dodaj `job_id` do tego, co już logujesz.

**Prompt do asystenta:**

```text
Dodaj do projektu tabelę zadań w tle.
Najpierw przeczytaj istniejące migracje w folderze supabase/ i sprawdź,
jak w projekcie zapisywane są ślady lub logi przebiegów agenta.

1. Utwórz migrację z tabelą jobs o DOKŁADNIE tych kolumnach:

[wklej tu blok SQL z tego kroku]

2. Włącz RLS: użytkownik może ODCZYTAĆ wyłącznie swoje zadania
   (owner_id = jego identyfikator). Wstawianie i zmiany — tylko po stronie
   serwera, kluczem serwerowym. Wyjątek zrobimy później dla anulowania.
3. Dodaj indeks pod listę zadań użytkownika (owner_id, created_at)
   i pod wyszukiwanie zadań po statusie.
4. Do zapisu śladów/logów przebiegu agenta dodaj pole job_id, tak żeby
   dało się znaleźć wszystkie przebiegi należące do jednego zadania.

Nie zmieniaj nazw kolumn ani listy statusów. Nie pisz jeszcze żadnej logiki
zadań. Nie czytaj i nie wypisuj wartości z plików .env.
```

**Gotowe, gdy:** tabela `jobs` istnieje, a zalogowany użytkownik widzi z przeglądarki tylko swoje wiersze i nie może ich zmienić.

### Krok 3 — Zrób zadanie idempotentnym

**Po co:** załóż, że **każde zadanie w tle kiedyś uruchomi się dwa razy**. Platformy wprost to zastrzegają (dostarczenie „w miarę możliwości", możliwe pominięcia i duplikaty), a przy awarii w połowie i tak ponawiasz sam. Dwa razy uruchomiony agent to podwójny koszt, a przy narzędziach zapisujących — podwójny e-mail albo komentarz.

Chcesz, żeby zadanie zachowywało się jak przycisk windy: naciśnięty dwa razy nie zamawia dwóch wind.

**Co robisz:** stosujesz trzy mechanizmy **razem**. Każdy łapie inny rodzaj powtórki.

| Mechanizm | Jak |
|---|---|
| **klucz idempotencji** | unikalny klucz na zadanie (`kind` + właściciel + okres lub identyfikator zdarzenia); druga próba wstawienia kończy się konfliktem i wychodzi bez pracy |
| **zajęcie zadania** | `update jobs set status='running' where id=$1 and status='queued' returning id` — pracuje tylko ten, kto dostał wiersz |
| **akcje zewnętrzne z własnym kluczem** | przed wysłaniem czegokolwiek na zewnątrz zapisz zamiar w bazie; po wysłaniu oznacz jako wykonany; przy wznowieniu pomiń wykonane |

Jak to działa po kolei:

- **Klucz** chroni przed dwukrotnym **utworzeniem** zlecenia. Harmonogram odpalił dwa razy? Drugie wstawienie `morning_review:<Ania>:2026-11-19` baza odrzuca. Dobry klucz zawiera wszystko, co odróżnia jedno zlecenie od drugiego: rodzaj, właściciela i okres albo identyfikator zdarzenia. Zły klucz: losowy identyfikator generowany przy każdej próbie — zawsze jest „nowy", więc niczego nie chroni.
- **Zajęcie** chroni przed dwukrotnym **wykonaniem** tego samego zlecenia. Dwóch pracowników naraz próbuje zmienić status z `queued` na `running`. Baza pozwoli tylko jednemu; drugi dostanie zero wierszy i odchodzi.
- **Zapis zamiaru** chroni przed dwukrotną **akcją na zewnątrz** — dla Ofertownika to wysyłka oferty po zatwierdzeniu.

Jedna pułapka: zadanie, którego pracownik padł w połowie, zostaje w statusie `running` na zawsze, i nikt go już nie zajmie. Dlatego zajęcie musi też umieć przejąć zadanie **porzucone**: takie, które jest `running`, ale wystartowało dawniej, niż mogło trwać jakiekolwiek uczciwe wykonanie.

**Prompt do asystenta:**

```text
Dodaj mechanizmy idempotencji dla zadań w tle. Przeczytaj migrację tabeli jobs.
Utwórz jeden moduł serwerowy z trzema funkcjami:

1. utwórz zadanie (kind, owner_id, idempotency_key, input):
   wstawia wiersz do jobs. Jeśli klucz już istnieje — NIE zgłasza błędu,
   tylko zwraca informację „już istnieje" i nic więcej nie robi.

2. zajmij zadanie (id): atomowo, jednym zapytaniem:
   update jobs set status='running' ... where id=$1 and status='queued' returning id
   Przy zajęciu zwiększ attempts i ustaw started_at.
   Zwraca prawdę tylko temu wywołaniu, które dostało wiersz.
   Dodatkowo pozwól zająć zadanie PORZUCONE: status 'running', a started_at
   starsze niż [X] sekund (X = okno niewidoczności kolejki, ustalę w kroku 5;
   na razie stała w jednym miejscu).

3. zakończ zadanie (id, status, result lub error): ustawia status
   done / failed / waiting_approval, zapisuje wynik i finished_at.

Zasady:
- żadnego „najpierw sprawdź, potem zmień" w dwóch zapytaniach — to musi być
  jedno zapytanie, inaczej dwóch pracowników zajmie to samo zadanie,
- dodaj test: dwa równoległe wywołania „zajmij zadanie" dla tego samego id —
  prawdę dostaje dokładnie jedno.

Nie czytaj i nie wypisuj wartości z plików .env.
```

**Gotowe, gdy:** test z dwoma równoległymi zajęciami przechodzi, a drugie wstawienie zadania z tym samym kluczem nie tworzy wiersza i nie kończy się błędem.

<details>
<summary>🔬 Głębiej: „dokładnie raz" jako złudzenie — co naprawdę gwarantują kolejki</summary>

W systemie, w którym cokolwiek może paść między dwoma krokami, są tylko dwie uczciwe gwarancje dostarczenia:

- **najwyżej raz** — wiadomość może zginąć, ale nigdy się nie powtórzy (usuwasz przed pracą),
- **co najmniej raz** — wiadomość nie zginie, ale może przyjść kilka razy (usuwasz po pracy).

„Dokładnie raz" wymagałoby, żeby „wykonałem pracę" i „potwierdziłem wykonanie" były jedną niepodzielną operacją. Między systemami nie są: zawsze istnieje chwila, w której praca jest zrobiona, a potwierdzenie jeszcze nie dotarło. Pracownik pada dokładnie wtedy — i z punktu widzenia kolejki nic się nie wydarzyło.

Okno niewidoczności to implementacja „co najmniej raz": wiadomość wraca, jeśli nikt jej nie usunął. Stąd cały ciężar przenosi się na odbiorcę: skoro dostarczenie się powtarza, **przetwarzanie musi być idempotentne**. To, co w materiałach marketingowych nazywa się „dokładnie raz", to prawie zawsze „co najmniej raz + deduplikacja po stronie odbiorcy". Uczciwsza nazwa: skutek dokładnie raz.

Kolejka w tej samej bazie Postgres, co dane, ma tu jedną realną przewagę: zapis wyniku i usunięcie wiadomości możesz objąć **jedną transakcją**. Wtedy dla skutków w bazie naprawdę dostajesz „dokładnie raz". Nie rozciąga się to jednak na nic poza bazą — wywołanie modelu i wysłanie maila nie biorą udziału w transakcji.

Dla agentów powtórzenie ma dodatkowy wymiar, którego nie ma w klasycznych systemach: **nie jest powtórzeniem tego samego**. Drugie wywołanie modelu da inny tekst. Jeśli pierwszy szkic został zapisany, a drugi przebieg zapisze swój, masz dwa różne szkice tej samej oferty — i brak dobrej odpowiedzi, który jest „właściwy". Dlatego w zadaniach agentowych wynik wywołania modelu traktuje się jak fakt do zapisania natychmiast, a nie jak coś, co można bezpiecznie policzyć ponownie.

Pułapka z oknem: jeśli przetwarzanie trwa dłużej niż okno, wiadomość wraca **w trakcie pracy** i bierze ją drugi pracownik. Zajęcie zadania w tabeli `jobs` to łapie — ale tylko jeśli warunek „porzucone" nie jest krótszy niż realny czas pracy. Te dwie liczby (okno i próg porzucenia) muszą być ustawione razem i obie powyżej limitu czasu funkcji.

</details>

<details>
<summary>🔬 Głębiej: idempotencja akcji zewnętrznych — zapis zamiaru przed wysyłką</summary>

Klucz idempotencji i zajęcie zadania chronią to, co dzieje się w Twojej bazie. Mail wysłany do klienta jest poza nią. Wzorzec „zapisz zamiar → wykonaj → oznacz" wygląda tak:

1. w jednej transakcji z decyzją o wysyłce wstaw wiersz „zamiar wysyłki" z własnym kluczem (dla Ofertownika `send_offer:<draft_id>`) i statusem `pending`,
2. wyślij,
3. oznacz wiersz jako `sent`, zapisując identyfikator zwrócony przez usługę.

Przy wznowieniu: `sent` → pomijasz. Brak wiersza → nic nie wysłano. Zostaje stan trudny: **`pending`**. Proces padł między krokiem 2 a 3 i nie wiesz, czy mail wyszedł. Masz trzy wyjścia, każde z ceną:

- **zapytaj usługę.** Jeśli API pozwala wyszukać wysłaną wiadomość po własnym znaczniku, który do niej dołączyłeś (własny nagłówek, identyfikator w treści), sprawdzasz i rozstrzygasz. Najlepsze, o ile usługa to umożliwia.
- **przekaż klucz usłudze.** Część API przyjmuje klucz idempotencji i sama odrzuca powtórkę. Wtedy ponawiasz bez obaw. Sprawdź w dokumentacji — nie zakładaj.
- **nie zgaduj, zapytaj człowieka.** Zadanie przechodzi w stan wymagający decyzji: „nie wiem, czy oferta wyszła — sprawdź folder Wysłane". Dla nieodwracalnej akcji o niskiej częstości to często najuczciwsze rozwiązanie.

Czego nie robić: ponawiać „bo pewnie nie wyszło". Podwójna oferta z różnymi cenami (bo między próbami ktoś poprawił szkic) to gorszy błąd niż oferta opóźniona o godzinę.

Dwie rzeczy specyficzne dla agentów:

- **klucz musi pochodzić z kodu, nie z modelu.** Jeśli model generuje argumenty akcji, przy ponowieniu wygeneruje inne — i klucz liczony z treści uzna to za nową akcję. Klucz buduj z identyfikatora zatwierdzonego szkicu.
- **zatwierdzanie zamraża treść.** Ania zatwierdza konkretną wersję szkicu; zamiar wysyłki wskazuje tę wersję. Zmiana szkicu po zatwierdzeniu wymaga nowego zatwierdzenia — inaczej człowiek zatwierdził coś innego, niż wyszło.

</details>

### Krok 4 — Zbuduj endpoint harmonogramu, który sprawdza sekret

**Po co:** harmonogram to budzik. Platforma o ustalonej porze wywołuje adres w Twojej aplikacji. Ten adres jest jednak **zwykłym publicznym adresem** — bez kontroli każdy, kto go zna, uruchamia Twojego agenta na Twój koszt.

**Co robisz:**

1. Na Vercelu definiujesz harmonogram w `vercel.json`:

```json
{ "crons": [{ "path": "/api/cron/morning-review", "schedule": "0 6 * * *" }] }
```

Zapis `0 6 * * *` to notacja cron: minuta 0, godzina 6, każdego dnia. Sprawdź w dokumentacji platformy, w jakiej strefie czasowej liczy godziny — od tego zależy, jaką liczbę wpiszesz, żeby u Ani wyszła 7:00.

2. Ustawiasz zmienną środowiskową `CRON_SECRET`. Platforma wyśle ją w nagłówku `Authorization: Bearer …`, a endpoint ją sprawdza:

```ts
export async function GET(request: Request) {
  if (request.headers.get("authorization") !== `Bearer ${process.env.CRON_SECRET}`) {
    return new Response("Unauthorized", { status: 401 });
  }
  // 1. dla każdego użytkownika z włączonym przeglądem: wstaw zadanie z kluczem idempotencji
  // 2. NIE uruchamiaj tu agenta dla wszystkich po kolei — tylko zakolejkuj
  return Response.json({ ok: true });
}
```

3. Endpoint **tylko wstawia zadania**. Nie uruchamia agenta. Agent dla wszystkich użytkowników po kolei nie zmieści się w limicie czasu funkcji, a pierwsza awaria zatrzymałaby resztę.

Rzeczy, które zaskakują: zadania cykliczne uruchamiają się **tylko na wdrożeniu produkcyjnym**, nieudane wywołanie **nie jest ponawiane**, a na darmowym planie godzina jest przybliżona. Dla Ani „około siódmej" wystarczy; jeśli u Ciebie liczy się minuta, ten mechanizm nie wystarczy.

Alternatywa w tym samym stacku: Supabase Cron (oparty na `pg_cron`), który potrafi uruchomić zapytanie SQL albo wywołać adres HTTP i pozwala na częstsze harmonogramy.

> ⚠️ Sposób definiowania zadań cyklicznych na Vercelu jest w trakcie zmiany — przed wdrożeniem sprawdź aktualną dokumentację platformy.

**Prompt do asystenta:**

```text
Dodaj endpoint wywoływany przez harmonogram platformy.

NAJPIERW przeczytaj aktualną dokumentację zadań cyklicznych mojej platformy
wdrożeniowej ([Vercel / inna]) — sposób ich definiowania mógł się zmienić.
Jeśli różni się od tego, co opisuję niżej, zatrzymaj się i powiedz mi.
Sprawdź też, w jakiej strefie czasowej platforma liczy godziny harmonogramu.

Potem:
1. Utwórz trasę GET /api/cron/[nazwa-zadania].
2. Pierwsza rzecz w funkcji: porównaj nagłówek authorization z wartością
   `Bearer ${process.env.CRON_SECRET}`. Jeśli się nie zgadza — odpowiedz 401
   i nic więcej nie rób.
3. Dla każdego użytkownika, który ma włączone to zadanie, utwórz zadanie
   funkcją z kroku 3, z kluczem idempotencji: [kind]:[owner_id]:[dzisiejsza data].
4. NIE uruchamiaj w tym endpoincie agenta ani żadnego wywołania modelu.
   Endpoint tylko tworzy zadania i odpowiada { ok: true } z liczbą utworzonych.
5. Dodaj definicję harmonogramu w pliku konfiguracyjnym platformy
   (dla Vercela: wpis crons w vercel.json) z porą: [godzina].
6. Dopisz NAZWĘ zmiennej CRON_SECRET do .env.example — bez wartości.

Powiedz mi, jak przetestować endpoint lokalnie: raz bez nagłówka (ma być 401)
i raz z nagłówkiem. Nie czytaj i nie wypisuj wartości z plików .env.
```

**Gotowe, gdy:** wywołanie adresu bez sekretu zwraca 401, a z sekretem — tworzy wiersze w `jobs`; drugie wywołanie tego samego dnia nie tworzy nowych.

### Krok 5 — Uruchom pracownika kolejki

**Po co:** w `jobs` czekają zlecenia. Ktoś musi je brać po jednym i wykonywać — tak, żeby awaria przy jednym nie zatrzymała reszty i żeby dwóch pracowników nie zrobiło tego samego.

**Co robisz:** w polecanym stacku kolejkę daje Supabase Queues (oparte na `pgmq`), z operacjami `send`, `read`, `delete` i `archive`.

| Operacja | Co robi | Kiedy w Ofertowniku |
|---|---|---|
| `send` | wkłada wiadomość do kolejki | po utworzeniu zadania `draft_offer` — wiadomość zawiera `job_id` |
| `read` | wyjmuje wiadomości i ukrywa je przed innymi na czas okna niewidoczności | pracownik: „daj mi do 3 wiadomości" |
| `delete` | usuwa wiadomość na dobre | po zapisaniu szkicu i statusu `done` |
| `archive` | odkłada wiadomość na bok, poza kolejkę | po trzeciej nieudanej próbie |

Kluczowe pojęcie to **okno niewidoczności**: wiadomość odczytana przez pracownika znika dla innych na zadany czas. Jeśli pracownik ją w tym czasie usunie — koniec. Jeśli padnie — wiadomość wraca do kolejki i dostanie ją ktoś inny. Stąd dwie zasady:

- okno musi być **dłuższe niż najdłuższe przetwarzanie** jednej wiadomości — inaczej dwóch pracowników zrobi to samo,
- usuwasz wiadomość **po** zakończeniu pracy, nie przed.

W Ofertowniku okno ma 300 sekund — tyle, ile wynosi cały limit czasu funkcji. Skoro żadne wywołanie nie może żyć dłużej, to gdy wiadomość wraca, poprzedni pracownik na pewno już nad nią nie pracuje.

Pracownika uruchamia harmonogram (co minutę: „weź do N wiadomości i przetwórz") albo wyzwalacz. Na darmowym planie Vercela harmonogram działa najwyżej raz na dobę, więc do budzenia pracownika co minutę służy Supabase Cron, który wywołuje adres pracownika. Ten adres też sprawdza sekret — tak samo jak endpoint z kroku 4.

Wiadomość, która nie udała się kilka razy, przenosisz na bok (`archive`) i oznaczasz zadanie jako `failed` — nie ponawiasz w nieskończoność.

Kolejność pracy pracownika dla jednej wiadomości:

1. odczytaj wiadomość (`read`),
2. sprawdź bezpieczniki kosztowe (krok 8),
3. zajmij zadanie (krok 3) — nie udało się? ktoś inny pracuje; zostaw wiadomość,
4. `attempts` większe niż limit? → `archive`, status `failed`, koniec,
5. wykonaj pracę, zapisując wynik po każdym kroku (krok 6),
6. ustaw status `done`,
7. dopiero teraz usuń wiadomość (`delete`).

**Prompt do asystenta:**

```text
Dodaj kolejkę i pracownika, który wykonuje zadania z tabeli jobs.

NAJPIERW przeczytaj aktualną dokumentację Supabase Queues i Supabase Cron
dla mojego projektu (pakiet klienta Supabase w node_modules oraz dokumentacja
na stronie Supabase). Potwierdź, jak w mojej wersji wywołuje się operacje
send, read, delete i archive oraz jak ustawia się okno niewidoczności.
Jeśli coś działa inaczej, niż opisuję — zatrzymaj się i powiedz mi.

Potem:
1. Utwórz kolejkę o nazwie [nazwa]. Po utworzeniu zadania w jobs wysyłaj
   do kolejki wiadomość zawierającą wyłącznie job_id (operacja send).
2. Utwórz trasę pracownika, np. /api/worker/[nazwa], chronioną sekretem
   w nagłówku authorization — tak samo jak endpoint harmonogramu.
3. Pracownik w jednym wywołaniu: odczytuje do [N] wiadomości (read) z oknem
   niewidoczności [X] sekund. X musi być dłuższe niż najdłuższe możliwe
   przetwarzanie jednej wiadomości i nie krótsze niż limit czasu funkcji
   na mojej platformie. Tę samą wartość X ustaw jako próg „zadania porzuconego"
   w funkcji zajęcia z kroku 3.
4. Dla każdej wiadomości, w tej kolejności:
   a) zajmij zadanie; jeśli się nie udało — pomiń wiadomość, nie usuwaj jej,
   b) jeśli attempts przekracza 3 — archive, status failed, przejdź dalej,
   c) wykonaj zadanie (na razie wywołaj jedną funkcję wykonaj(job) — jej
      wnętrze napiszemy w następnym kroku),
   d) zapisz status done i wynik,
   e) DOPIERO TERAZ usuń wiadomość (delete).
   Błąd w jednej wiadomości nie może przerwać pozostałych.
5. Pilnuj czasu: jeśli do limitu funkcji zostało mało, nie zaczynaj kolejnej
   wiadomości — zostanie na następne wywołanie.
6. Podaj mi gotową definicję zadania Supabase Cron, które co minutę wywołuje
   adres pracownika z nagłówkiem autoryzacji. Sekret ma pochodzić z bezpiecznego
   miejsca, nie być wpisany na stałe w SQL w repozytorium.

Nie czytaj i nie wypisuj wartości z plików .env.
```

**Gotowe, gdy:** zadanie wstawione do `jobs` samo przechodzi przez `queued` → `running` → `done` bez otwartej przeglądarki, a po zakończeniu kolejka jest pusta.

### Krok 6 — Pokrój pracę agenta na kroki

**Po co:** w zwykłym programie powtórzenie kroku po awarii jest darmowe. W agencie każde powtórzone wywołanie modelu kosztuje i może dać inny wynik. Zadanie agentowe w tle dziel więc tak, żeby **wywołanie modelu nigdy nie było powtarzane bez potrzeby**.

**Co robisz:**

- jeden element (dokument, zgłoszenie, użytkownik) = jedno zadanie w kolejce, nie jedna wielka pętla,
- wynik każdego kosztownego kroku zapisujesz do bazy **zanim** przejdziesz dalej,
- przy wznowieniu najpierw sprawdzasz, co już jest zapisane,
- limit kroków agenta i limit czasu narzędzi z [Modułu 5](M5_MVP.md) obowiązują bez zmian.

Dobrze: 14 maili to 14 zadań `draft_offer`. Awaria przy piątym — trzynaście pozostałych jest gotowych, a piąte ponawia się samo. Źle: jedno zadanie z pętlą po 14 mailach. Awaria przy dwunastym — po wznowieniu płacisz drugi raz za jedenaście już zrobionych.

To samo wewnątrz jednego zadania. W Ofertowniku zadanie `draft_offer` ma dwa kosztowne kroki: ocenę, czy mail jest zapytaniem ofertowym, i właściwy przebieg agenta. Po każdym wynik ląduje w `jobs.result`. Jeśli proces padnie po ocenie, drugie podejście czyta zapisaną ocenę i od razu uruchamia agenta.

**Prompt do asystenta:**

```text
Napisz funkcję wykonaj(job) dla zadania rodzaju [kind], wywoływaną przez pracownika.
Przeczytaj kod pracownika, kod pętli agenta i tabelę jobs.

Zasada nadrzędna: żadne wywołanie modelu nie może zostać powtórzone, jeśli jego
wynik jest już zapisany.

1. Rozpisz mi najpierw (bez kodu) kroki tego zadania i przy każdym napisz:
   czy woła model, czy zapisuje coś na zewnątrz, ile mniej więcej trwa.
   Poczekaj na moją akceptację listy.
2. Dla każdego kroku: na początku sprawdź w jobs.result, czy wynik tego kroku
   już jest. Jeśli tak — pomiń krok i użyj zapisanego wyniku.
3. Po każdym kroku, który woła model, NATYCHMIAST zapisz wynik do jobs.result
   i dodaj koszt wywołania do jobs.cost_usd — zanim zaczniesz następny krok.
4. Jeśli zadanie przetwarza wiele elementów, NIE rób pętli w jednym zadaniu:
   dla każdego elementu utwórz osobne zadanie z własnym kluczem idempotencji
   ([kind]:[owner_id]:[identyfikator elementu]) i wyślij wiadomość do kolejki.
5. Każdy przebieg agenta w ramach zadania zapisuje job_id w śladzie.
6. Zachowaj istniejący limit kroków agenta i limity czasu narzędzi.
7. Zadanie w tle NIE wykonuje akcji nieodwracalnych. Jeśli scenariusz takiej
   wymaga, zadanie kończy się statusem waiting_approval.

Nie czytaj i nie wypisuj wartości z plików .env.
```

**Gotowe, gdy:** po zakończeniu zadania w `jobs.result` widać wynik każdego kroku osobno, a w `jobs.cost_usd` sumę kosztów.

<details>
<summary>🔬 Głębiej: jak kroić pracę agenta, żeby nie powtarzać wywołań modelu</summary>

„Zapisuj po każdym kroku" jest proste, dopóki krok to jedno wywołanie modelu. Pętla agenta to kilka–kilkanaście wywołań przeplecionych narzędziami. Gdzie stawiać punkty zapisu?

**Poziom 1 — całe zadanie.** Zapisujesz tylko wynik końcowy. Awaria w połowie = cała pętla od nowa. Wystarcza, gdy przebieg kosztuje centy i trwa sekundy. Dla Ofertownika (cel: 0,05 USD i 60 s na szkic) to rozsądny punkt startu.

**Poziom 2 — etapy przed i po pętli.** Klasyfikacja → pętla agenta → walidacja wyniku → zapis. Zapisujesz między etapami. Pętla nadal jest niepodzielna, ale nie powtarzasz tego, co wokół niej. To daje większość korzyści za ułamek złożoności.

**Poziom 3 — każdy krok pętli.** Po każdym wywołaniu modelu i każdym wyniku narzędzia zapisujesz pełną historię wiadomości. Wznowienie odtwarza historię i woła model od miejsca przerwania. Warto przy długich, drogich przebiegach (dziesiątki kroków, dolary na zadanie). Koszty: historia potrafi być duża, a wznowienie nie jest idealne — model dostaje tę samą historię, ale nic nie gwarantuje, że podejmie tę samą dalszą decyzję. To akurat nie szkodzi, o ile wszystkie narzędzia wywołane **przed** przerwaniem były idempotentne albo tylko czytały.

Tu jest sedno: granicę zapisu wyznaczają **skutki uboczne**, nie wywołania modelu. Powtórzone wywołanie modelu to strata kilku centów. Powtórzone narzędzie zapisujące to duplikat w danych. Praktyczna reguła: punkt zapisu **bezpośrednio po** każdym narzędziu, które coś zmienia; narzędzia czytające można powtarzać.

Trzy rzeczy, o których łatwo zapomnieć:

- **Wyniki narzędzi czytających starzeją się.** Cennik odczytany o 7:02 i zapisany w historii zostanie użyty przy wznowieniu o 7:40. Zwykle to w porządku; przy kalendarzu trenerów już niekoniecznie. Zdecyduj świadomie, które odczyty są „faktami z chwili" i przy wznowieniu trzeba je odświeżyć.
- **Pamięć podręczna promptu** zmienia rachunek. Jeśli dostawca tanio rozlicza powtórzony prefiks, ponowienie całej pętli bywa dużo tańsze, niż wynika z liczby tokenów — i poziom 3 opłaca się jeszcze rzadziej. Zmierz, zanim zbudujesz.
- **Krojenie na osobne zadania zrywa wspólny kontekst.** 14 niezależnych zadań nie zauważy, że trzy maile są od tego samego klienta. Jeśli to ważne, grupowanie zrób w kodzie **przed** kolejkowaniem (jedno zadanie na klienta), a nie przez wspólną wielką pętlę.

Pytanie otwarte: czy przy wznowieniu po kilku próbach zmieniać model na mocniejszy? Podnosi szansę powodzenia i koszt; bez pomiaru na własnych nieudanych przypadkach to zgadywanie.

</details>

### Krok 7 — Daj użytkownikowi widok i hamulec

**Po co:** zadania, którego nie widać, nikt nie pilnuje. A praca, której nie da się zatrzymać, prędzej czy później zrobi coś, czego nikt nie chciał.

**Co robisz:** minimum to cztery rzeczy.

- lista zadań użytkownika ze **statusem**, czasem i kosztem,
- **powiadomienie** o zakończeniu lub błędzie (w aplikacji wystarczy),
- przycisk **Anuluj** — zadanie sprawdza status między krokami i kończy się, gdy widzi `cancelled`,
- zatwierdzanie z Modułu 5 działa asynchronicznie: zadanie przechodzi w `waiting_approval` i **kończy wywołanie**; decyzja człowieka tworzy nowe zadanie, które wykonuje akcję.

Dwa wyjaśnienia. Anulowanie nie „zabija" działającego kodu — ustawia w bazie status `cancelled`, a zadanie samo to zauważa przed kolejnym krokiem. Dlatego krojenie z kroku 6 jest warunkiem działającego hamulca: zadanie z jedną wielką pętlą nie ma kiedy sprawdzić statusu.

Zatwierdzanie asynchroniczne: w czacie agent czekał na kliknięcie z otwartym połączeniem. W tle nie może — Ania kliknie za godzinę albo jutro, a funkcja żyje najwyżej kilka minut. Zadanie więc zapisuje szkic, ustawia `waiting_approval` i **kończy się**. Gdy Ania kliknie „Zatwierdź i wyślij", powstaje **nowe** zadanie (z kluczem `send_offer:<draft_id>`), które wykonuje wysyłkę.

**Prompt do asystenta:**

```text
Dodaj widok zadań w tle dla zalogowanego użytkownika.
Przeczytaj tabelę jobs, kod pracownika i istniejący widok zatwierdzania.

1. Strona „Zadania": lista zadań użytkownika od najnowszych, z kolumnami:
   czego dotyczy (czytelny opis na podstawie kind i input), status po polsku,
   czas rozpoczęcia i trwania, koszt z cost_usd, link do wyniku.
   Dla statusu failed pokaż czytelny powód z kolumny error, bez surowych
   komunikatów technicznych.
2. Lista odświeża się sama co kilka sekund, dopóki jakieś zadanie
   ma status queued albo running.
3. Powiadomienie w aplikacji, gdy zadanie przejdzie w done, failed
   albo waiting_approval.
4. Przycisk „Anuluj" przy zadaniach queued i running: po stronie serwera
   sprawdź, że zadanie należy do zalogowanego użytkownika, i ustaw status
   cancelled — tylko jeśli obecny status to queued albo running.
5. W funkcji wykonaj(job): PRZED każdym krokiem odczytaj aktualny status
   zadania z bazy. Jeśli to cancelled — zakończ natychmiast, nie wołaj modelu,
   usuń wiadomość z kolejki.
6. Zatwierdzanie: zadanie, które potrzebuje decyzji człowieka, ustawia
   waiting_approval i kończy wywołanie — nie czeka. Kliknięcie „Zatwierdź"
   tworzy NOWE zadanie z własnym kluczem idempotencji, które wykonuje akcję.
   Dwukrotne kliknięcie nie może utworzyć dwóch zadań.

Nie czytaj i nie wypisuj wartości z plików .env.
```

**Gotowe, gdy:** widzisz swoje zadanie na liście ze statusem i kosztem, a kliknięcie Anuluj w trakcie pracy zatrzymuje je przed następnym krokiem.

### Krok 8 — Ogranicz koszt tego, czego nikt nie ogląda

**Po co:** w czacie użytkownik zauważy, że coś poszło źle. W tle agent może mielić całą noc. Do bezpieczników z [Modułu 6](M6_BEZPIECZENSTWO_I_KOSZTY.md) dochodzą nowe — wszystkie sprawdzane przez kod, nie przez człowieka.

**Co robisz:**

| Bezpiecznik | Chroni przed |
|---|---|
| **budżet na jedno zadanie** — sumujesz koszt kroków w `jobs.cost_usd`, po przekroczeniu `failed` | jednym zadaniem, które się zapętliło |
| **limit prób** na zadanie (np. 3) | wiecznym ponawianiem tego samego błędu |
| **limit zadań na użytkownika** na dobę i jednocześnie | zalaniem kolejki |
| **dzienny budżet aplikacji i wyłącznik awaryjny** sprawdzane przez pracownika **przed** każdym zadaniem | wszystkim powyższym naraz |
| **alert**, gdy kolejka rośnie albo odsetek `failed` skacze | awarią, o której dowiesz się z rachunku |

Wartości dla Ofertownika: 0,10 USD na zadanie, 3 próby, 1 przegląd i najwyżej 30 szkiców na użytkownika na dobę. Rachunek kontrolny: typowy poranek to 14 maili × około 0,04 USD ≈ 0,56 USD. Najgorszy możliwy: 30 zadań × 0,10 USD = 3 USD. Znasz górną granicę, zanim cokolwiek uruchomisz — o to chodzi w bezpiecznikach.

> 🛑 Przy kluczu kursowym to nie jest formalność. Harmonogram uruchamiający agenta dla każdego użytkownika co minutę potrafi zużyć wspólny limit w jedną noc. Zaczynaj od najrzadszego harmonogramu, który ma sens, i od najtańszego modelu.

Zwróć uwagę na różnicę: pracownik budzony co minutę, który zastaje pustą kolejkę, nie woła modelu i nic nie kosztuje. Groźny jest harmonogram, który co minutę **tworzy nowe zadania dla agenta**.

**Prompt do asystenta:**

```text
Dodaj bezpieczniki kosztowe do zadań w tle. Przeczytaj kod pracownika,
funkcję wykonaj(job), docs/BEZPIECZENSTWO.md (jeśli istnieje) i miejsce,
w którym projekt liczy koszt wywołania modelu.

Wszystkie wartości umieść w JEDNYM pliku konfiguracyjnym jako nazwane stałe:
1. Budżet na jedno zadanie: [np. 0,10] USD. Po każdym wywołaniu modelu dodaj
   koszt do jobs.cost_usd; jeśli suma przekracza budżet — przerwij zadanie
   i ustaw failed z powodem „przekroczony budżet zadania".
2. Limit prób: 3. (Sprawdź, że pracownik już to robi, i użyj tej samej stałej.)
3. Limit zadań na użytkownika: [N] na dobę i [M] jednocześnie — sprawdzany
   przy tworzeniu zadania; nadmiarowe zadanie nie powstaje.
4. Dzienny budżet całej aplikacji: [kwota] USD oraz wyłącznik awaryjny
   (jedna flaga w bazie albo zmiennej środowiskowej). Pracownik sprawdza OBA
   przed każdym zadaniem; gdy któryś zadziała — nie zajmuje zadania i zostawia
   wiadomość w kolejce.
5. Alert: gdy liczba wiadomości w kolejce przekracza [X] albo odsetek zadań
   failed z ostatniej godziny przekracza [Y]% — zapisz ostrzeżenie w logu
   w sposób, który da się potem podpiąć pod powiadomienie.

Na końcu wypisz mi plik:linia dla każdego z pięciu bezpieczników i policz
najgorszy możliwy dzienny koszt jednego użytkownika przy tych wartościach.
Nie czytaj i nie wypisuj wartości z plików .env.
```

**Gotowe, gdy:** umiesz wskazać `plik:linia` dla budżetu na zadanie i limitu prób, a zadanie z budżetem ustawionym testowo na 0,001 USD kończy się statusem `failed`.

### Krok 9 — Sprawdź dwa razy i przerwij w połowie

**Po co:** idempotencja i wznawianie to obietnice. Dopóki ich nie sprawdzisz, nie wiesz, czy działają — a produkcja sprawdzi je za Ciebie w najgorszym momencie.

**Co robisz:** dwa testy.

**Test podwójnego uruchomienia.** Wywołaj wyzwalacz dwa razy z tym samym kluczem — najlepiej jednocześnie. Oczekiwany wynik: jeden wiersz w `jobs`, jeden przebieg agenta, jeden szkic. Pokaż to w logu.

**Test przerwania.** Przerwij zadanie w połowie (np. wymuszonym błędem po pierwszym kroku) i uruchom ponownie. Oczekiwany wynik: ukończone kroki nie są powtarzane — widać to po tym, że **koszt drugiego przebiegu jest niższy**.

**Prompt do asystenta:**

```text
Napisz dwa testy zadań w tle. Mają działać na bazie testowej, z modelem
zastąpionym atrapą, która liczy wywołania i zwraca stały koszt — testy
nie mogą wołać prawdziwego modelu ani zewnętrznych usług.

Test 1 — podwójne uruchomienie:
- wywołaj endpoint harmonogramu dwa razy RÓWNOLEGLE (z poprawnym sekretem),
- potem uruchom pracownika dwa razy równolegle,
- sprawdź: w jobs jest dokładnie jeden wiersz na klucz idempotencji,
  atrapa modelu została wywołana tyle razy, ile przy pojedynczym uruchomieniu,
  powstał dokładnie jeden wynik.
- wypisz czytelny log: ile prób wstawienia, ile odrzuconych, ile zajęć udanych.

Test 2 — przerwanie w połowie:
- uruchom zadanie z wymuszonym błędem PO zapisaniu wyniku pierwszego kroku,
- sprawdź, że zadanie nie ma statusu done, a wiadomość nie została usunięta,
- zasymuluj upływ okna niewidoczności i uruchom pracownika ponownie,
- sprawdź: krok pierwszy NIE został wykonany drugi raz (licznik atrapy),
  zadanie ma status done, a koszt dopisany w drugim podejściu jest niższy
  niż koszt pełnego przebiegu.

Test 3 — sekret: wywołanie endpointu harmonogramu i pracownika bez nagłówka
authorization zwraca 401 i nie tworzy żadnego wiersza.

Uruchom testy i pokaż wynik. Jeśli któryś nie przechodzi, NIE poprawiaj testu —
pokaż mi, co w kodzie zadania jest nie tak.
```

**Gotowe, gdy:** wszystkie trzy testy przechodzą, a w logu pierwszego widać odrzuconą drugą próbę.

### Krok 10 — Opisz zadanie

**Po co:** zadanie w tle działa wtedy, gdy nikt nie patrzy. Gdy o trzeciej w nocy coś pójdzie źle, ten plik odpowiada na pytanie „co to miało robić i co się stanie, jeśli uruchomię to jeszcze raz".

**Co robisz:** uzupełniasz `docs/ZADANIA_W_TLE.md` — jedna sekcja na zadanie:

```markdown
## <nazwa zadania>
**Wyzwalacz:** harmonogram codziennie 6:00 / zdarzenie / żądanie użytkownika
**Wzorzec:** harmonogram + kolejka
**Klucz idempotencji:** morning_review:<owner_id>:<data>
**Kroki i co jest zapisywane po każdym:** …
**Limity:** budżet 0,10 USD na zadanie, 3 próby, 1 zadanie na użytkownika na dobę
**Co widzi użytkownik:** status, wynik, koszt, przycisk Anuluj
**Co się dzieje przy awarii w połowie:** …
```

Wersję wypełnioną masz w sekcji „Przykład: Ofertownik". Najważniejsze są dwa pola z wielokropkiem: jeśli nie umiesz ich wypełnić konkretem, wróć do kroku 6.

**Prompt do asystenta:**

```text
Uzupełnij docs/ZADANIA_W_TLE.md na podstawie KODU, nie planów.
Dla każdego zadania w tle wypełnij pola: Wyzwalacz, Wzorzec,
Klucz idempotencji (dokładny format), Kroki i co jest zapisywane po każdym
(z nazwą pola w jobs.result), Limity (wartości z pliku konfiguracyjnego),
Co widzi użytkownik, Co się dzieje przy awarii w połowie (krok po kroku:
co wraca do kolejki, co jest pomijane przy wznowieniu, co po wyczerpaniu prób).

Jeśli w kodzie jest krok wołający model, po którym wynik NIE jest zapisywany
przed następnym krokiem — nie ukrywaj tego w dokumencie, tylko wypisz mi to
jako problem. Nie twórz innych plików.
```

**Gotowe, gdy:** obca osoba po przeczytaniu sekcji umie powiedzieć, co się stanie, gdy zadanie uruchomi się dwa razy i gdy padnie w połowie.

## Najczęstsze potknięcia

- **Harmonogram „nie działa".** Objaw: lokalnie i na wdrożeniu testowym nic się nie uruchamia. → Zadania cykliczne na Vercelu działają tylko na wdrożeniu produkcyjnym. Lokalnie wywołuj endpoint ręcznie, z nagłówkiem.
- **Zadanie znika bez śladu.** Objaw: harmonogram odpalił, platforma zwróciła błąd, nic się nie ponowiło. → Nieudane wywołanie harmonogramu nie jest ponawiane. Dlatego endpoint ma robić jak najmniej (tylko wstawić zadania), a ponawianie należy do kolejki.
- **Ta sama praca dwa razy.** Objaw: dwa szkice dla jednego maila, podwójny koszt. → Okno niewidoczności jest krótsze niż przetwarzanie, albo usuwasz wiadomość przed zakończeniem pracy, albo zajęcie zadania to dwa zapytania zamiast jednego.
- **Zadanie na zawsze w `running`.** Objaw: status nie zmienia się od godzin. → Pracownik padł, a zajęcie nie umie przejąć zadania porzuconego. Wróć do kroku 3.
- **Błąd 504 w połowie przeglądu.** Objaw: pierwsze elementy gotowe, reszta nie. → Jedna pętla po wszystkich elementach przekroczyła limit czasu funkcji. Jeden element = jedno zadanie w kolejce.
- **Anuluj nie zatrzymuje.** Objaw: status `cancelled`, a agent pracuje dalej. → Zadanie nie sprawdza statusu między krokami, albo nie ma kroków, między którymi mogłoby sprawdzić.

## ✅ Sprawdź, czy masz wszystko

- [ ] Zadanie uruchamia się bez udziału przeglądarki i zostawia wiersz w `jobs` ze statusem, kosztem i powiązanym śladem.
- [ ] **Test podwójnego uruchomienia:** wyzwalacz wywołany dwa razy z tym samym kluczem — agent pracuje raz. Widać to w logu.
- [ ] **Test przerwania:** zadanie przerwane w połowie i uruchomione ponownie nie powtarza ukończonych kroków (koszt drugiego przebiegu jest niższy).
- [ ] Endpoint wyzwalany przez platformę odrzuca żądanie bez sekretu.
- [ ] Budżet na zadanie i limit prób są w kodzie — umiesz wskazać `plik:linia`.
- [ ] Użytkownik widzi listę zadań ze statusem i kosztem i może zadanie anulować.
- [ ] Zadanie w tle nie wykonuje akcji nieodwracalnych bez zatwierdzenia.
- [ ] `docs/ZADANIA_W_TLE.md` jest wypełniony i zgodny z kodem.

## 🤖 Poproś asystenta o recenzję

```text
Przeczytaj kod zadania w tle mojego agenta (wyzwalacz, pracownik, schemat
tabeli zadań) oraz docs/ZADANIA_W_TLE.md.
Oceń jak inżynier, który będzie przy tym systemie dyżurował. Dla każdego punktu
napisz PASS albo FAIL i jedno zdanie uzasadnienia ze wskazaniem pliku:

1. Co się stanie, jeśli wyzwalacz odpali się dwa razy w tej samej sekundzie?
2. Co się stanie, jeśli proces padnie po wywołaniu modelu, a przed zapisaniem wyniku?
3. Czy istnieje ścieżka, na której to samo działanie zewnętrzne wykona się dwukrotnie?
4. Czy endpoint wyzwalany przez platformę da się wywołać z zewnątrz bez sekretu?
5. Co ograniczy koszt, jeśli zadanie zacznie się ponawiać w pętli o trzeciej w nocy?

Niczego nie zmieniaj w plikach. Nie odczytuj i nie wypisuj plików .env.
Na końcu podaj jedną rzecz do poprawienia w pierwszej kolejności.
```

## 🚀 Dla chętnych

- 🔵 **Postęp na żywo.** Zapisuj postęp zadania do bazy i pokaż go w interfejsie bez odświeżania strony („przetworzono 9 z 14 maili").
- 🔵 **Reakcja na zdarzenie.** Zamiast harmonogramu przyjmij webhook z zewnętrznej usługi: zweryfikuj podpis, wstaw zadanie z identyfikatorem zdarzenia jako kluczem idempotencji, odpowiedz natychmiast. Dla Ofertownika: szkic powstaje kilka minut po przyjściu maila, a nie następnego dnia rano.
- 🟣 **Zatwierdzanie następnego dnia.** Przenieś zatwierdzanie do trwałego przepływu: zadanie czeka na decyzję dowolnie długo i wznawia się po niej bez ponownego wykonywania wcześniejszych kroków. Porównaj z wersją na tabeli `jobs`: ile kodu ubyło, ile nowych rodzajów awarii doszło, co się dzieje z przepływem rozpoczętym przed wdrożeniem nowej wersji kodu.
- 🟣 **Zmierz cenę awarii.** Wstrzykuj losowe przerwania w różnych punktach zadania (przed wywołaniem modelu, po nim a przed zapisem, po zapisie a przed usunięciem wiadomości) na 100 przebiegach. Policz: ile powstało duplikatów, ile zadań utknęło, o ile wzrósł łączny koszt względem przebiegu bez awarii. Wynik powie Ci, czy potrzebujesz gęstszych punktów zapisu, czy już przepłacasz za złożoność.

## Co dalej

Agent pracuje sam, o stałej porze, a Ty wiesz, ile najwyżej może to kosztować. Czas wystawić go światu i zobaczyć, co robi, gdy nie patrzysz: [Moduł 7: Wdrożenie](M7_WDROZENIE.md).
