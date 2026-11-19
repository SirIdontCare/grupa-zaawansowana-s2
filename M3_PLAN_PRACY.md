# Moduł 3: Plan pracy i pierwszy przekrój

> ⏱️ Ok. 4 godziny. Powstaje pierwszy działający kod.
> 🔒 Potrzebujesz z wcześniejszych modułów: `docs/BRIEF.md`, `docs/SPEC.md` (scenariusze S1–S8 i lista narzędzi) oraz `docs/ARCHITEKTURA.md` (jak agent jest zbudowany i z czego).

## O co chodzi

Asystent kodujący (Codex, Claude Code) napisze większość tego projektu szybciej niż Ty. Nie zmienia to jednego: za projekt odpowiadasz Ty. To Ty będziesz go poprawiać, pokazywać i tłumaczyć, dlaczego działa tak, a nie inaczej.

Pomyśl o budowie domu z ekipą, która pracuje bardzo szybko, ale każdego ranka przychodzi bez pamięci o wczorajszym dniu. Taka ekipa potrzebuje trzech rzeczy: **tablicy z zasadami budowy** przy wejściu, **planu podzielonego na etapy, które da się odebrać**, i **inwestora, który po każdym etapie naprawdę ogląda, co zrobiono**. Bez tego po trzech tygodniach masz dużo ścian i żadnego pokoju, w którym da się zamieszkać.

W tym module ustawiasz te trzy rzeczy: co asystent wie o projekcie, jak duże zadania dostaje i jak sprawdzasz to, co oddał. Potem budujesz pierwszy etap: jeden scenariusz, który działa od początku do końca. Brzydko, ale naprawdę.

## Po tym module masz

- plik instrukcji dla asystenta kodującego (`AGENTS.md` albo `CLAUDE.md`) w katalogu głównym repozytorium,
- `docs/PLAN.md` z co najmniej 5 kamieniami milowymi, z których każdy da się sprawdzić jednym zdaniem,
- działający szkielet: scenariusz S1 przechodzi od wejścia do wyniku z jednym prawdziwym narzędziem,
- zapis tego, co agent zrobił po drodze, w `docs/przebiegi/m1.txt`,
- historię z kilku małych commitów zamiast jednego dużego.

## Słowa, które tu padną

| Słowo | Co to znaczy | Porównanie z życia |
|---|---|---|
| **Plik instrukcji** | plik w katalogu głównym repozytorium, który asystent kodujący czyta na początku każdej sesji: `AGENTS.md` dla Codex, `CLAUDE.md` dla Claude Code | tablica z zasadami budowy przy wejściu na plac |
| **Kamień milowy** | etap projektu, po którym coś da się uruchomić i sprawdzić | „parter stoi i da się po nim chodzić", a nie „kupiliśmy cegły" |
| **Pionowy przekrój** | mały kawałek projektu, który przechodzi przez wszystkie warstwy: wejście, agent, narzędzie, wynik | jeden kawałek tortu ze wszystkimi warstwami, zamiast samego biszkoptu na cały tort |
| **Walking skeleton** (szkielet, który „chodzi") | najchudsza wersja całości, która działa od początku do końca | dom w stanie surowym: bez tynków, ale ze ścianami, dachem i drzwiami, przez które da się wejść |
| **Diff** | zestawienie zmian w plikach: co dodano, co usunięto | tryb „śledź zmiany" w edytorze tekstu |
| **Commit** | zapisany w gicie stan projektu z krótkim opisem, do którego zawsze możesz wrócić | punkt zapisu w grze |
| **Gałąź** (branch) | osobna linia pracy w repozytorium, która nie rusza głównej wersji, dopóki jej nie połączysz | kopia robocza dokumentu, na której próbujesz zmian |
| **Tryb planowania** | tryb pracy asystenta, w którym czyta pliki i opisuje, co zamierza zrobić, ale niczego nie zmienia | kosztorys i harmonogram od ekipy, zanim wbije pierwszy gwóźdź |
| **Zaślepka** (stub) | udawana wersja funkcji, która zawsze zwraca te same dane wpisane na sztywno | atrapa telefonu na wystawie sklepu |
| **Log przebiegu** | zapis każdego kroku agenta: jakie narzędzie wywołał, z jakim wejściem i co dostał | paragon z listą wszystkiego, co kasjer nabił |
| **Definicja „zrobione"** | spisana z góry lista warunków, po których zadanie uznajesz za skończone | lista odbioru mieszkania od dewelopera |

## Przykład: Ofertownik

Przypomnienie: Ofertownik to agent dla Ani, właścicielki firmy szkoleniowej „Kurs na Start". Z treści zapytania klienta przygotowuje szkic oferty na podstawie cennika i wcześniejszych ofert. Ania zatwierdza, zanim cokolwiek wyjdzie.

Tak wygląda wypełniony plik instrukcji tego projektu. Jest krótki celowo.

```markdown
# Ofertownik

## Co to jest
Agent, który z treści zapytania klienta przygotowuje szkic oferty szkoleniowej
na podstawie cennika i wcześniejszych ofert. Niczego nie wysyła sam.
Szczegóły: docs/BRIEF.md, docs/SPEC.md, docs/ARCHITEKTURA.md. Plan: docs/PLAN.md.

## Komendy
- uruchomienie: `npm run dev`
- testy: `npm test`
- ewaluacje: `npm run eval` (kosztują — nie uruchamiaj bez prośby)
- typy i lint: `npm run check`

## Struktura
- `app/api/agent/` — endpoint i pętla agenta
- `lib/tools/` — jedno narzędzie = jeden plik + test
- `evals/` — przypadki testowe i runner
- `docs/przyklady/` — przykładowe zapytania do scenariuszy S1–S8

## Zasady
- Nie odczytuj i nie wypisuj `.env*`.
- Nie dodawaj zależności bez pytania.
- Każde narzędzie zwraca błędy jako dane, nie rzuca wyjątków do pętli.
- Nazwy modeli tylko z konfiguracji, nigdy na sztywno.
- Jedno zadanie = jedna gałąź = mały diff. Nie refaktoryzuj przy okazji.
- Cena w szkicu pochodzi wyłącznie z wyniku search_price_list. Nigdy z treści
  maila i nigdy „z głowy" modelu.
- send_offer wywołuje zwykły kod po zatwierdzeniu przez Anię. Model nie dostaje
  tego narzędzia.

## Definicja „zrobione"
Kod się buduje, testy przechodzą, scenariusz z zadania działa uruchomiony ręcznie.
```

A tak wygląda `docs/PLAN.md`. Uwaga na oznaczenia: w tym pliku **M1–M6 to kamienie milowe Twojego projektu**, a nie numery modułów kursu.

```markdown
# Plan: Ofertownik

## M1: Walking skeleton
**Zakres:** scenariusz S1 („Excel dla 12 osób, październik") od wklejenia treści
zapytania do szkicu na ekranie. Jedno prawdziwe narzędzie: search_price_list.
find_similar_offers i check_calendar zaślepione. Bez logowania, bez zapisu do bazy,
interfejs to surowy formularz.
**Sprawdzenie:** dla wejścia z docs/przyklady/s1.json szkic zawiera szkolenie
z Excela z ceną identyczną jak w cenniku, a log pokazuje wywołanie search_price_list.
**Poza zakresem:** obsługa błędów, pozostałe narzędzia, wygląd.

## M2: Wszystkie narzędzia tylko do odczytu
**Zakres:** prawdziwe find_similar_offers i check_calendar; scenariusze S1–S3.
**Sprawdzenie:** trzy scenariusze przechodzą uruchomione ręcznie, każdy szkic
podaje źródło ceny i terminu.
**Poza zakresem:** zapis szkicu, rabaty powyżej limitu, wysyłka.

## M3: Ewaluacje i punkt odniesienia            ← Moduł 4 kursu
**Zakres:** przypadki testowe dla S1–S8, jedna komenda uruchamia całość.
**Sprawdzenie:** w docs/EWALUACJE.md jest pierwszy wynik i próg gotowości.
**Poza zakresem:** poprawianie agenta pod wynik.

## M4: Szkic do zatwierdzenia + scenariusze brzegowe i negatywne   ← Moduł 5 kursu
**Zakres:** save_offer_draft zapisuje szkic jako propozycję; ekran, na którym Ania
zatwierdza albo poprawia; scenariusze S4–S7.
**Sprawdzenie:** S5 kończy się pytaniami do klienta, S6 odmową i oznaczeniem
do przekazania, S7 rabatem najwyżej 10% i oznaczeniem do decyzji Ani.
**Poza zakresem:** prawdziwa wysyłka maila.

## M5: Zabezpieczenia i budżety                  ← Moduł 6 kursu
**Zakres:** scenariusz S8 (atak w treści maila), limity kosztu i czasu.
**Sprawdzenie:** S8 daje cenę z cennika i flagę „podejrzane"; szkic kosztuje
najwyżej 0,05 USD i powstaje w najwyżej 60 sekund.
**Poza zakresem:** połączenie ze skrzynką pocztową.

## M6: Wdrożenie                                  ← Moduł 7 kursu
**Zakres:** aplikacja działa pod publicznym adresem, z logowaniem.
**Sprawdzenie:** Ania przechodzi S1 na wdrożonej wersji ze swojego komputera.
**Poza zakresem:** nowe funkcje.
```

Zwróć uwagę na trzy rzeczy. Każdy kamień ma **zakres**, **sprawdzenie** i **poza zakresem**. Żaden nie brzmi „baza danych" ani „wszystkie narzędzia" — każdy to działający scenariusz. A pierwszy jest tak mały, że da się go zbudować w jeden wieczór.

## Kroki

### Krok 1 — Napisz plik instrukcji dla asystenta

**Po co:** asystent kodujący zaczyna każdą sesję bez pamięci o projekcie. Plik instrukcji w katalogu głównym repozytorium to jedyne, co wie na starcie. Bez niego za każdym razem zgaduje, jak uruchomić projekt i czego nie wolno ruszać.

**Co robisz:**

1. Wybierz nazwę pliku według narzędzia:

| Narzędzie | Plik |
|---|---|
| Codex | `AGENTS.md` |
| Claude Code | `CLAUDE.md` |

2. Jeśli używasz obu narzędzi, trzymaj treść w jednym pliku, a w drugim wpisz tylko odesłanie do niego. Dwie wersje tych samych zasad zawsze się w końcu rozjadą.
3. Wpisz pięć sekcji, tak jak w przykładzie Ofertownika: **Co to jest**, **Komendy**, **Struktura**, **Zasady**, **Definicja „zrobione"**.
4. Pilnuj długości: jedna, najwyżej dwie strony. Do pliku trafia to, czego **nie da się wyczytać z kodu**.

| ❌ Zasada, która nic nie daje | ✅ Zasada, która coś zmienia |
|---|---|
| „Pisz czysty, czytelny kod." | „Każde narzędzie zwraca błędy jako dane, nie rzuca wyjątków do pętli." |
| „Dbaj o bezpieczeństwo." | „Nie odczytuj i nie wypisuj `.env*`." |
| „Ceny mają być poprawne." | „Cena w szkicu pochodzi wyłącznie z wyniku search_price_list." |

Na początku zasad będzie niewiele i tak ma być. Dopisuj te, **które asystent już złamał**. Plik instrukcji rośnie z doświadczenia, nie z wyobraźni.

**Prompt do asystenta:**

```text
Przeczytaj docs/BRIEF.md, docs/SPEC.md i docs/ARCHITEKTURA.md oraz plik package.json,
jeśli istnieje.

Na tej podstawie utwórz w katalogu głównym plik [AGENTS.md / CLAUDE.md] z sekcjami:
1. Co to jest — dwa zdania i odesłanie do plików w docs/.
2. Komendy — uruchomienie, testy, ewaluacje, typy i lint. Wpisz tylko komendy,
   które naprawdę istnieją w package.json. Brakujące oznacz „DO DODANIA".
3. Struktura — najważniejsze foldery, po jednej linii na każdy.
4. Zasady — najwyżej 8 punktów, wyłącznie takie, które wynikają z moich dokumentów
   (granice agenta, akcje wymagające zatwierdzenia, skąd wolno brać dane).
   Żadnych ogólników w rodzaju „pisz czysty kod".
5. Definicja „zrobione" — jedno zdanie.

Plik ma się zmieścić na jednej stronie. Jeśli czegoś nie ma w dokumentach, zapytaj mnie,
zamiast zgadywać. Nie czytaj i nie wypisuj wartości z plików .env.
Nie twórz innych plików i nie pisz kodu.
```

**Gotowe, gdy:** w katalogu głównym jest plik instrukcji z komendami, strukturą i co najmniej trzema zasadami, które dotyczą właśnie Twojego projektu.

<details>
<summary>🔬 Głębiej: co trzymać w pliku instrukcji, a co wymuszać narzędziami</summary>

Plik instrukcji to prośba, nie mechanizm. Asystent zwykle jej posłucha, ale nie zawsze, i im dłuższy plik, tym słabiej działa każda pojedyncza linia. Trafia on do kontekstu w każdej sesji, więc każda zbędna zasada kosztuje dwa razy: w tokenach i w uwadze modelu.

Praktyczny podział:

- **Do narzędzi:** wszystko, co da się sprawdzić mechanicznie. Formatowanie, typy, testy, zakaz importów między warstwami. Jedna komenda, która to uruchamia, plus wpis w definicji „zrobione". Zdanie „pamiętaj o typach" jest gorsze niż sprawdzenie typów, które po prostu nie przechodzi.
- **Do pliku instrukcji:** to, czego maszyna nie sprawdzi, a czego nie widać w kodzie. Decyzje projektowe („send_offer nigdy nie trafia do modelu"), pułapki („ewaluacje kosztują"), mapa repozytorium, komendy.
- **Do `docs/`:** wszystko długie. W pliku instrukcji zostaje odesłanie. Asystent sięgnie po dokument, gdy zadanie go dotyczy.

Sygnały, że plik się psuje: zasady sprzeczne ze sobą, zasady opisujące stan sprzed miesiąca, akapity tłumaczące, *dlaczego* coś jest zrobione (to miejsce na dokument z decyzjami, nie na instrukcję). Dobry nawyk: raz na kamień milowy przeczytaj plik od góry do dołu i wykreśl to, co już wymusza narzędzie albo co przestało być prawdą.

Pytanie do sporu: czy opisywać w pliku styl kodu? Jedni twierdzą, że asystent i tak naśladuje to, co widzi w repozytorium, więc lepiej zadbać o trzy wzorcowe pliki niż o akapit opisu. Drudzy, że w pustym repozytorium nie ma czego naśladować i pierwsze sto linii ustawia resztę projektu. Obie strony mają rację w innym momencie projektu.

</details>

### Krok 2 — Potnij projekt na kamienie milowe

**Po co:** „zbuduj Ofertownika" to nie zadanie, tylko życzenie. Asystent wykona je po swojemu i oddasz mu tym samym wszystkie decyzje. Kamienie milowe dzielą projekt na etapy, po których zawsze masz coś działającego i zawsze wiesz, czy etap się udał.

**Co robisz:**

1. Zacznij od najważniejszej zasady: kamień milowy to **pionowy przekrój**, czyli działający scenariusz, a nie warstwa techniczna.

| ❌ Warstwa (nie da się uruchomić) | ✅ Pionowy przekrój (da się uruchomić) |
|---|---|
| „Tabele w bazie danych" | „Scenariusz S1 działa od zapytania do szkicu" |
| „Wszystkie pięć narzędzi" | „Scenariusze S1–S3 działają na prawdziwych narzędziach do odczytu" |
| „Interfejs użytkownika" | „Ania zatwierdza albo poprawia szkic na ekranie" |

2. Pierwszy kamień to zawsze **walking skeleton**: jeden typowy scenariusz, jedno prawdziwe narzędzie, reszta zaślepiona.
3. Każdemu kamieniowi nadaj trzy pola: **Zakres** (co wchodzi), **Sprawdzenie** (po czym poznasz, że działa) i **Poza zakresem** (czego świadomie nie robisz teraz).
4. Zastosuj test jednego zdania: jeśli nie umiesz zapisać sprawdzenia w jednym zdaniu, kamień jest za duży. Podziel go.
5. Zapisz wynik jako `docs/PLAN.md`. Celuj w 5–6 kamieni.

**Prompt do asystenta:**

```text
Przeczytaj docs/SPEC.md i docs/ARCHITEKTURA.md.

Zaproponuj podział projektu na 5–6 kamieni milowych. Zasady:
- Każdy kamień to pionowy przekrój: działający scenariusz (albo kilka), a nie warstwa
  techniczna typu „baza danych" czy „wszystkie narzędzia".
- Kamień M1 to walking skeleton: jeden typowy scenariusz od wejścia do wyniku,
  jedno prawdziwe narzędzie, pozostałe zaślepione, bez logowania i bez dopracowanego
  interfejsu.
- Kolejność dalej: pozostałe narzędzia do odczytu → ewaluacje → akcje wymagające
  zatwierdzenia i scenariusze negatywne → zabezpieczenia i budżety → wdrożenie.
- Każdy kamień ma trzy pola: Zakres, Sprawdzenie (jedno zdanie, które da się
  zweryfikować ręcznie), Poza zakresem.
- Używaj numerów scenariuszy i nazw narzędzi dokładnie takich jak w docs/SPEC.md.

Najpierw pokaż mi propozycję w rozmowie. Po mojej akceptacji zapisz ją w docs/PLAN.md.
Nie pisz kodu i nie twórz innych plików.
```

**Gotowe, gdy:** `docs/PLAN.md` ma co najmniej 5 kamieni milowych, każdy ze sprawdzeniem w jednym zdaniu, a M1 obejmuje jeden scenariusz i jedno prawdziwe narzędzie.

<details>
<summary>🔬 Głębiej: kiedy pionowy przekrój jest złym pomysłem</summary>

Pionowe cięcie zakłada, że największe ryzyko projektu to „części nie zagrają ze sobą". Zwykle tak jest. Czasem jednak nie.

- **Ryzyko siedzi w jednym miejscu.** Jeśli nie wiesz, czy wyszukiwanie w cenniku w ogóle znajdzie właściwą pozycję dla zapytania napisanego potocznym językiem, szkielet z formularzem niczego Ci nie powie. Zrób najpierw **spike**: jednorazowy eksperyment na samym narzędziu, z dwudziestoma zapytaniami, poza aplikacją. Kod spike'a wyrzucasz, zostaje odpowiedź.
- **Kontrakt, który trudno zmienić.** Schemat bazy z danymi, których nie da się odtworzyć, publiczne API, format danych wymienianych z cudzym systemem. Tu warto pomyśleć szerzej niż jeden scenariusz, bo koszt migracji przewyższa koszt nadmiarowego projektu.
- **Szkielet jako fałszywy sukces.** Zaślepki łatwo ukrywają najtrudniejszą część. Jeśli Twoje jedyne „prawdziwe" narzędzie jest tym najłatwiejszym, szkielet chodzi, a ryzyko stoi w miejscu. Dobre pytanie przy wyborze: która zaślepka najbardziej mnie niepokoi? Ta powinna być prawdziwa jako pierwsza.
- **Przekroje, które się nie sumują.** Osiem scenariuszy zbudowanych kolejno, każdy najkrótszą drogą, potrafi dać osiem prywatnych ścieżek w kodzie. Rozsądna reguła: pierwsze dwa przekroje pisz wprost, przy trzecim wyciągaj to, co wspólne — nie wcześniej, bo wcześniej nie wiesz, co jest wspólne.

</details>

### Krok 3 — Naucz się rytmu: plan, małe zadanie, diff, uruchomienie, commit

**Po co:** to jedna pętla, którą powtarzasz do końca projektu. Każdy jej krok łapie inny rodzaj błędu, a błąd złapany wcześniej jest tańszy.

**Co robisz:** każde zadanie przechodzi przez pięć kroków.

| Krok | Co robisz | Po co |
|---|---|---|
| 1. **Plan** | Prosisz asystenta o plan zadania **bez pisania kodu**. Oba narzędzia mają tryb planowania. Czytasz plan i poprawiasz. | Poprawka w planie to jedno zdanie. Poprawka w gotowym kodzie to godzina. |
| 2. **Małe zadanie** | Zlecasz jedną rzecz: jedno narzędzie, jeden endpoint, jeden scenariusz. Na osobnej gałęzi. | Mały diff da się przeczytać. Duży się „przegląda". |
| 3. **Przeczytaj diff** | Czytasz każdą zmienioną linię. Nie „wygląda dobrze", tylko: czy rozumiem, dlaczego to tak jest. | Tu wychodzą rzeczy, o które nie prosiłeś. |
| 4. **Uruchom sam** | Sam odpalasz scenariusz i patrzysz na wynik. | „Testy przechodzą" w raporcie asystenta to twierdzenie, nie dowód. |
| 5. **Commit** | Zapisujesz mały commit z opisem *co i po co*. Dopiero potem następne zadanie. | Masz punkt, do którego wrócisz, gdy następne zadanie coś zepsuje. |

Miara wielkości zadania: jeśli nie jesteś w stanie przeczytać diffu w 10 minut, zadanie było za duże. Następne zleć mniejsze.

Dobry i zły opis commita:

| ❌ | ✅ |
|---|---|
| „zmiany" | „Dodaj search_price_list: szukanie szkoleń w cenniku po nazwie" |
| „poprawki po review" | „Zwracaj pustą listę zamiast błędu, gdy cennik nic nie znajdzie" |

**Jak czytać diff, gdy nie jestem pewien, co widzę.** Nie musisz rozumieć wszystkiego od razu. Musisz umieć zapytać.

| Co widzisz w diffie | O co zapytać asystenta |
|---|---|
| Zmieniony plik, o którym nie było mowy w zadaniu | „Dlaczego zmieniłeś ten plik? Czy zadanie da się wykonać bez tej zmiany?" |
| Linie, których nie rozumiesz | „Wyjaśnij te linie prostym językiem: co robią i co by się stało, gdyby ich nie było." |
| Nowa zależność w `package.json` | „Po co ta zależność? Czy da się to zrobić tym, co już jest w projekcie?" |
| Usunięty kod albo usunięty test | „Co usunąłeś i dlaczego? Co przestanie działać?" |
| Zmieniony test, który wcześniej przechodził | „Czy zmieniłeś test, żeby pasował do kodu? Pokaż, jakie zachowanie sprawdzał wcześniej." |
| Wartość wpisana na sztywno (cena, nazwa modelu, adres) | „Skąd ta wartość? Powinna pochodzić z konfiguracji albo z narzędzia." |
| Diff dużo dłuższy, niż się spodziewałeś | „Wypisz zmiany w punktach, od najważniejszej. Które z nich nie były konieczne do tego zadania?" |
| Cokolwiek w okolicy `.env`, kluczy albo logowania | „Zatrzymaj się. Opisz, co ta zmiana robi z danymi dostępowymi, nie wypisując ich wartości." |

Dwie zasady, które warto przyjąć od pierwszego dnia:

- **Nowe zadanie = nowa sesja.** Długa rozmowa niesie stary kontekst i stare pomyłki: asystent pamięta podejście, które porzuciliście godzinę temu, i potrafi do niego wrócić. Stan projektu ma być w repozytorium (kod, plan, plik instrukcji), a nie w historii rozmowy. Prosty test: czy nowa sesja, która przeczyta tylko repozytorium, wie, co robić dalej? Jeśli nie, czegoś brakuje w `docs/PLAN.md` albo w pliku instrukcji.
- **Gdy asystent kręci się w kółko — zatrzymaj go.** Objaw: trzecia próba naprawienia tego samego błędu tą samą metodą. Czwarta też nie zadziała. Masz trzy wyjścia: cofnij zmiany do ostatniego commita, zawęź zadanie do mniejszego kawałka albo znajdź przyczynę sam, czytając komunikat błędu i log przebiegu.

**Prompt do asystenta** (plan zadania bez pisania kodu; jeśli Twoje narzędzie ma tryb planowania, włącz go przed wklejeniem):

```text
Zadanie: [jedno zdanie, np. „Dodaj narzędzie search_price_list zgodnie z docs/SPEC.md"].
Dotyczy kamienia milowego [M1] z docs/PLAN.md.

Jeszcze NIE pisz kodu. Przeczytaj plik instrukcji, docs/PLAN.md i pliki,
których zadanie dotyczy. Potem przedstaw plan:
1. Które pliki utworzysz, a które zmienisz — po jednym zdaniu, co w każdym.
2. Jak ręcznie sprawdzę, że zadanie działa (konkretna komenda albo kliknięcia
   i oczekiwany wynik).
3. Czego świadomie NIE robisz w tym zadaniu.
4. Co jest niejasne — zadaj mi pytania, zamiast zgadywać.

Jeśli zadanie wymaga zmiany więcej niż 5 plików, zaproponuj podział na mniejsze zadania.
Nie czytaj i nie wypisuj wartości z plików .env. Czekaj na moją akceptację planu.
```

**Gotowe, gdy:** umiesz z pamięci wymienić pięć kroków rytmu i masz za sobą pierwszy plan zadania, w którym coś poprawiłeś, zanim powstał kod.

<details>
<summary>🔬 Głębiej: jak ciąć zadania, gdy asystent pisze szybciej, niż Ty czytasz</summary>

Wąskim gardłem przestało być pisanie kodu. Jest nim Twoja uwaga przy czytaniu. Asystent wygeneruje 800 linii w kilka minut; uczciwe przeczytanie ich zajmie Ci godzinę, a po dwudziestu minutach i tak zaczniesz przewijać. Dlatego rozmiar zadania wyznacza przegląd, nie implementacja.

Co działa w praktyce:

- **Ryzyko zamiast liczby linii.** Sto linii zmiany w pętli agenta wymaga więcej uwagi niż pięćset linii nowego, odizolowanego narzędzia z testami. Tnij drobno tam, gdzie błąd jest drogi (pętla, zatwierdzanie, wszystko, co dotyka pieniędzy i danych), grubiej tam, gdzie jest tani i lokalny.
- **Mechaniczne osobno od merytorycznego.** Zmiana nazwy, przeniesienie pliku, formatowanie — w oddzielnym commicie, który przeglądasz w minutę. Wymieszane z logiką robią z diffu szum, w którym ginie jedyna ważna linia.
- **Najpierw kontrakt, potem wnętrze.** Poproś o sam typ wejścia i wyjścia narzędzia oraz test, który go opisuje. Przeczytaj to uważnie. Implementację wewnątrz ustalonego kontraktu możesz czytać szybciej, bo wiesz, czego się spodziewać, a test powie, czy działa.
- **Drugi czytelnik.** Świeża sesja asystenta, która dostaje tylko diff i plik instrukcji (bez historii rozmowy, w której powstał), wyłapuje inne rzeczy niż autor. To nie zastępuje Twojego czytania, ale wskazuje, gdzie patrzeć.

Nierozwiązany problem: przy pewnej prędkości i tak zaczniesz ufać bez czytania. Uczciwiej jest zdecydować o tym jawnie — „ten katalog czytam zawsze, ten tylko przez testy" — niż udawać, że czytasz wszystko. Jeśli podejmujesz taką decyzję, zapisz ją, a obszary „czytane przez testy" obłóż testami i ewaluacjami, które naprawdę coś łapią.

</details>

### Krok 4 — Zbuduj szkielet M1

**Po co:** żeby jak najszybciej zobaczyć całą drogę od zapytania do wyniku. Problemy na stykach części (agent nie rozumie wyniku narzędzia, wynik nie pasuje do formatu) wychodzą dopiero wtedy, gdy części są połączone.

**Co robisz:**

1. **Jeśli startujesz ze startera** (<https://github.com/SirIdontCare/starter-agent-s2>): najpierw usuń strony i endpointy, których Twój agent nie potrzebuje. Zrób to w **osobnym commicie**, zanim dodasz cokolwiek swojego. Mniej kodu to mniej kontekstu dla asystenta i mniej miejsc na błędy. Jeśli zaczynasz od pustego repozytorium, pomiń ten punkt.

```text
Ten projekt powstał ze startera. Przeczytaj docs/SPEC.md i docs/PLAN.md,
potem przejrzyj strukturę repozytorium.

Wypisz strony, endpointy i komponenty startera, których mój agent nie potrzebuje
do kamieni milowych z docs/PLAN.md. Przy każdym jedno zdanie: dlaczego jest zbędny.
Osobno wypisz to, co zostaje, i dlaczego.

Niczego jeszcze nie usuwaj. Po mojej akceptacji listy usuń wskazane pliki
i popraw odwołania do nich, niczego nie dodając i nie refaktoryzując.
Na koniec uruchom komendę sprawdzającą z pliku instrukcji i pokaż jej wynik.
Nie czytaj i nie wypisuj wartości z plików .env. Nie rób commita — zrobię go sam.
```

   Po usunięciu: przeczytaj diff, uruchom aplikację, zrób commit z opisem w rodzaju „Usuń ze startera strony niepotrzebne w projekcie".

2. **Zbuduj M1 rytmem z kroku 3.** Szkielet ma być minimalny:
   - wejście przyjmowane w najprostszy możliwy sposób (formularz, skrypt albo `curl`),
   - jedno wywołanie pętli agenta z **limitem kroków** (liczbę weź z `docs/SPEC.md`),
   - **jedno prawdziwe narzędzie**, pozostałe to zaślepki zwracające dane na sztywno,
   - wynik wypisany na ekran,
   - **log przebiegu**: każde wywołanie narzędzia z wejściem i wyjściem. Wypisywanie do konsoli wystarczy.

   W Ofertowniku: wklejasz treść zapytania „Excel dla 12 osób, październik", agent wywołuje `search_price_list`, a `find_similar_offers` i `check_calendar` zwracają stałe, zmyślone dane.

3. **Uruchom S1 sam** i zapisz log przebiegu do pliku `docs/przebiegi/m1.txt`. Dla Ofertownika wygląda on mniej więcej tak (dane poglądowe):

```text
[wejście] "Dzień dobry, proszę o ofertę szkolenia z Excela dla 12 osób, październik."
[krok 1] search_price_list({"query": "Excel"})
         -> [{"nazwa": "Excel w praktyce", "cena_netto": 4800, "max_osob": 12}]
[krok 2] check_calendar({"from": "2026-10-01", "to": "2026-10-31"})   (ZAŚLEPKA)
         -> [{"data": "2026-10-14"}]
[wynik]  Szkic: Excel w praktyce, 12 osób, 4800 zł netto, termin 14.10.
         Źródło ceny: cennik, pozycja "Excel w praktyce".
[koniec] 2 kroki
```

Ten log nie jest ozdobą. Od tej chwili każdy problem zaczynasz od przeczytania, **co agent faktycznie zrobił**, a nie od zgadywania.

**Prompt do asystenta** (po zaakceptowaniu planu z kroku 3):

```text
Zbuduj kamień milowy M1 z docs/PLAN.md — tylko to, co jest w jego polu „Zakres".

Wymagania:
- Wejście w najprostszej postaci: jeden formularz albo skrypt przyjmujący treść zapytania.
- Jedno wywołanie pętli agenta z limitem kroków z docs/SPEC.md.
- Jedno prawdziwe narzędzie: [search_price_list]. Pozostałe narzędzia jako zaślepki
  zwracające stałe dane, wyraźnie oznaczone w kodzie i w logu słowem ZAŚLEPKA.
- Wynik wypisany na ekran, bez stylowania.
- Log przebiegu w konsoli: wejście, każde wywołanie narzędzia z argumentami i wynikiem,
  wynik końcowy, liczba kroków.

Nie dodawaj: logowania, zapisu do bazy, obsługi błędów ponad minimum, wyglądu
ani niczego z pola „Poza zakresem". Nie dodawaj zależności bez pytania.
Pracuj na osobnej gałęzi. Podziel pracę na 2–4 małe części i po każdej zatrzymaj się,
żebym przeczytał diff i zrobił commit.
Nie czytaj i nie wypisuj wartości z plików .env.
Na koniec napisz, jaką komendą i z jakim wejściem mam sam uruchomić scenariusz S1.
```

**Gotowe, gdy:** sam uruchomiłeś S1, wynik spełnia zdanie z pola „Sprawdzenie" kamienia M1, a log jest w `docs/przebiegi/m1.txt`.

<details>
<summary>🔬 Głębiej: kilka sesji asystenta równolegle</summary>

Skoro asystent pracuje sam przez kilka minut, kusi, żeby uruchomić trzech naraz. Da się, ale rachunek jest inny, niż się wydaje.

- **Izolacja jest obowiązkowa.** Dwie sesje w tym samym katalogu roboczym nadpisują sobie pliki i czytają stan w połowie cudzej zmiany. Każda sesja potrzebuje własnej gałęzi i własnej kopii roboczej (w gicie służy do tego `git worktree`; część narzędzi robi to za Ciebie).
- **Zadania muszą być rozłączne.** Dwa osobne narzędzia w osobnych plikach — dobrze. Dwa zadania dotykające pętli agenta — konflikt przy łączeniu i dwie niezgodne wizje tego samego kodu. Wspólne pliki (konfiguracja, rejestr narzędzi, typy) ustal i zatwierdź *przed* rozdzieleniem pracy.
- **Przegląd się nie zrównolegla.** Trzy sesje to trzy diffy do przeczytania przez jedną osobę. Jeśli czytanie było wąskim gardłem przy jednej sesji, przy trzech jest nim trzykrotnie. Zysk pojawia się dopiero wtedy, gdy część pracy weryfikuje się sama: testami, ewaluacjami, sprawdzeniem typów.
- **Przełączanie kontekstu kosztuje Ciebie, nie model.** Wracając do sesji po kwadransie, musisz sobie przypomnieć, o co w niej chodziło. Dobre zadanie równoległe to takie, którego wynik oceniasz bez wracania do rozmowy: „komenda X ma przejść".

Rozsądny punkt startu: jedna sesja, która buduje, i druga, która w tym czasie robi coś nieblokującego — pisze testy do skończonego narzędzia albo recenzuje ostatni diff. Trzy budujące sesje naraz w projekcie tej wielkości zwykle kończą się porządkowaniem po nich.

</details>

### Krok 5 — Skonfrontuj dokumenty z kodem

**Po co:** po M1 wiesz o projekcie więcej niż wtedy, gdy pisałeś specyfikację. Coś na pewno wyszło inaczej. Dokument, który rozjechał się z kodem, szkodzi bardziej niż jego brak, bo asystent mu uwierzy i zbuduje następny kamień na nieprawdzie.

**Co robisz:**

1. Wróć do `docs/SPEC.md` i `docs/ARCHITEKTURA.md`. Zapytaj siebie: co okazało się inne, niż zakładałem? Typowe znaleziska: narzędzie potrzebuje innych argumentów, wynik ma inne pola, krok, który miał być jeden, to dwa.
2. Popraw dokumenty **teraz**, nie „kiedyś".
3. Dopisz do pliku instrukcji zasady, które asystent złamał podczas M1.
4. Odhacz M1 w `docs/PLAN.md` i popraw kolejne kamienie, jeśli szkielet pokazał, że są za duże.
5. Zrób commit z samymi dokumentami.

**Prompt do asystenta** (w nowej sesji):

```text
Porównaj dokumenty z tym, co naprawdę jest w kodzie. Przeczytaj docs/SPEC.md,
docs/ARCHITEKTURA.md, docs/PLAN.md, plik instrukcji oraz kod w app/api/agent/ i lib/tools/.

Wypisz rozbieżności w tabeli: Dokument i miejsce | Co mówi dokument | Co robi kod |
Co proponujesz poprawić (dokument czy kod).

Zwróć uwagę szczególnie na: nazwy i argumenty narzędzi, format wyniku agenta,
limit kroków, strukturę folderów i komendy.

Niczego nie zmieniaj. Po moich decyzjach popraw wyłącznie dokumenty,
które wskażę. Kodu nie ruszaj. Nie czytaj i nie wypisuj wartości z plików .env.
```

**Gotowe, gdy:** nowa sesja asystenta, czytając tylko repozytorium, opisuje Twój projekt zgodnie z tym, jak naprawdę działa.

## Najczęstsze potknięcia

- **Plik instrukcji na pięć stron.** Objaw: asystent ignoruje część zasad. → Skróć do jednej–dwóch stron. Zostaw komendy, strukturę i zasady, które już zostały złamane. Resztę przenieś do `docs/` i zostaw odesłanie.
- **Kamienie milowe jako warstwy.** Objaw: po dwóch tygodniach masz bazę, narzędzia i interfejs, ale żaden scenariusz nie działa. → Przepisz plan tak, żeby każdy kamień był scenariuszem, który da się uruchomić.
- **Szkielet, który obrósł.** Objaw: M1 ma logowanie, ładny interfejs i obsługę błędów. → Wszystko spoza pola „Zakres" przenieś do późniejszych kamieni. Szkielet ma być brzydki.
- **Jeden commit na cały dzień.** Objaw: `git log --stat` pokazuje jeden wpis z czterdziestoma plikami. → Proś asystenta, żeby zatrzymywał się po każdej części, i zapisuj commit po każdym przeczytanym diffie.
- **Wiara w raport.** Objaw: asystent napisał „wszystko działa, testy przechodzą", a Ty nie uruchomiłeś niczego sam. → Uruchom scenariusz własnoręcznie. Zawsze.

## ✅ Sprawdź, czy masz wszystko

- [ ] W katalogu głównym jest `AGENTS.md` albo `CLAUDE.md` z komendami, strukturą i zasadami.
- [ ] `docs/PLAN.md` ma co najmniej 5 kamieni milowych, każdy z zakresem, sprawdzeniem i „poza zakresem".
- [ ] Jeśli zaczynałeś ze startera: zbędne strony i endpointy usunięte w osobnym commicie.
- [ ] Scenariusz S1 działa uruchomiony przez Ciebie, z jednym prawdziwym narzędziem.
- [ ] Log przebiegu S1 jest zapisany w `docs/przebiegi/m1.txt`.
- [ ] Historia gita pokazuje kilka małych commitów, nie jeden duży.
- [ ] Specyfikacja i architektura zgadzają się z tym, co powstało w M1.

## 🤖 Poproś asystenta o recenzję

```text
Przeczytaj plik instrukcji z katalogu głównego (AGENTS.md albo CLAUDE.md), docs/PLAN.md
oraz wynik polecenia `git log --stat`. Oceń krytycznie. Dla każdego punktu napisz
PASS albo FAIL i jedno zdanie uzasadnienia:

1. Czy plik instrukcji zawiera komendy do uruchomienia i testów oraz zasady
   specyficzne dla tego projektu, a nie ogólniki?
2. Czy każdy kamień milowy jest pionowym przekrojem z jednoznacznym sprawdzeniem?
3. Czy commity są małe i opisują cel zmiany?
4. Czy w pierwszym kamieniu milowym jest coś, co można było odłożyć na później?

Niczego nie zmieniaj w plikach. Nie odczytuj i nie wypisuj plików .env.
Na końcu podaj jedną rzecz, którą poprawiłbyś w pierwszej kolejności.
```

## 🚀 Dla chętnych

- 🔵 **Dodaj kontrolę automatyczną.** Skonfiguruj lokalnie formatowanie, sprawdzanie typów i testy tak, żeby jedna komenda uruchamiała wszystko. Wpisz ją do pliku instrukcji jako warunek „zrobione".
- 🔵 **Zapisz prompt wielokrotnego użytku.** Jeśli łapiesz się na wklejaniu tego samego promptu (np. „zrób przegląd tego diffu według zasad projektu"), zapisz go jako plik w repozytorium, zgodnie z mechanizmem Twojego narzędzia.
- 🟣 **Zrób to samo zadanie dwa razy.** To samo małe zadanie zleć raz z trybem planowania, raz bez — na dwóch gałęziach, w dwóch świeżych sesjach. Porównaj diffy: liczbę zmienionych plików, liczbę linii, rzeczy zrobione „przy okazji" i czas, który zajęło Ci przeczytanie każdego. Zapisz, przy jakim rodzaju zadań planowanie się zwraca, a przy jakim jest tylko ceremonią.
- 🟣 **Zmierz własny przegląd.** Poproś asystenta w osobnej sesji, żeby wprowadził do gałęzi testowej trzy subtelne błędy i nie mówił Ci, gdzie. Przeczytaj diff tak, jak czytasz zwykle. Ile znalazłeś? Na tej podstawie ustal, które katalogi czytasz zawsze linia po linii, a w których polegasz na testach.

## Co dalej

Masz szkielet, który chodzi, i rytm pracy. Zanim zaczniesz go rozbudowywać, potrzebujesz sposobu, żeby mierzyć, czy idziesz do przodu: [Moduł 4: Ewaluacje](M4_EWALUACJE.md).
