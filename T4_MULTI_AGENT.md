# Moduł techniczny 4: Multi-agent — orkiestrator i podagenci

> ⏱️ Ok. 6 godzin.
> 🧩 **Moduł do wyboru.** Weź go, jeśli **pomiar** pokazuje, że jeden agent nie wystarcza: zadanie nie mieści się w jednym kontekście albo składa się z części, które da się robić równolegle. Wymaga działającego MVP z ewaluacjami ([Moduł 4](M4_EWALUACJE.md), [Moduł 5](M5_MVP.md)).

## O co chodzi

Wyobraź sobie jednego pracownika, który robi wszystko sam, i kierownika z trzyosobowym zespołem. Zespół zrobi więcej naraz: trzy osoby sprawdzają trzy rzeczy w tym samym czasie. Ale ktoś musi dobrze rozdzielić zadania, bo inaczej dwie osoby zrobią to samo, a trzeciej rzeczy nie zrobi nikt. I każda dodatkowa osoba kosztuje.

Z agentami jest dokładnie tak samo. „Zespół agentów" brzmi jak naturalny następny krok po jednym agencie. W praktyce to najdroższa decyzja o budowie systemu, jaką możesz podjąć: więcej wywołań modelu, więcej miejsc na błąd, trudniejsze szukanie przyczyn. Zespół, który zbudował i opisał publicznie produkcyjny system badawczy z wieloma agentami, podał, że zużywa on około **15 razy więcej tokenów** niż zwykły czat. Pojedynczy agent zużywa około 4 razy więcej.

Mimo to są zadania, w których to się opłaca. Ten moduł uczy dwóch rzeczy: **rozpoznać, czy Twoje zadanie jest jednym z nich**, i zbudować najprostszą wersję, która przynosi korzyść. Ta wersja to agent-kierownik, który zleca wydzielone części pracy agentom-pomocnikom.

Ważne od początku: wynik „sprawdziłem, wracam do jednego agenta" jest **pełnoprawnym zakończeniem tego modułu**. Decyzja poparta liczbami jest warta więcej niż zespół agentów zbudowany dlatego, że brzmiał nowocześnie.

## Po tym module masz

- odpowiedź **popartą liczbami**, czy Twój projekt potrzebuje więcej niż jednego agenta,
- znajomość pięciu prostszych wzorców, które warto wypróbować wcześniej,
- działającego orkiestratora z co najmniej jednym podagentem (albo udokumentowany powód, dla którego go nie budujesz),
- plik `docs/MULTI_AGENT.md`: porównanie wersji z jednym agentem i z podagentami na tych samych ewaluacjach (skuteczność, koszt, czas) oraz decyzję.

## Słowa, które tu padną

| Słowo | Co to znaczy | Porównanie z życia |
|---|---|---|
| **Orkiestrator** | agent główny: dzieli zadanie, zleca części i składa wynik | kierownik zespołu |
| **Podagent** | osobne wywołanie agenta, uruchamiane przez orkiestratora do jednej części zadania | pracownik, który dostaje jedno zlecenie |
| **Kontekst** | wszystko, co model widzi w danej chwili: instrukcje, rozmowa, wyniki narzędzi | blat biurka: mieści się na nim ograniczona liczba kartek |
| **Izolacja kontekstu** | podagent pracuje na własnym, czystym kontekście; do orkiestratora wraca tylko wniosek | pracownik czyta 50 stron u siebie i przynosi kierownikowi pół strony notatki |
| **Zlecenie** (brief dla podagenta) | tekst, który podagent dostaje od orkiestratora; jedyne, co wie o zadaniu | kartka z poleceniem dla kogoś, kto nie był na zebraniu |
| **Łańcuch** | stała sekwencja wywołań modelu: wynik jednego jest wejściem następnego | taśma produkcyjna |
| **Routing** | pierwsze wywołanie rozpoznaje typ zadania i kieruje je na właściwą ścieżkę | recepcja w przychodni: „do internisty" albo „do okulisty" |
| **Równoległość** | kilka niezależnych wywołań uruchomionych jednocześnie | trzy kasy otwarte naraz zamiast jednej |
| **Oceniający i poprawiający** | jedno wywołanie tworzy, drugie ocenia według kryteriów i odsyła do poprawy | autor i redaktor |
| **Limit kroków** | najwyższa liczba kroków, po której agent musi się zatrzymać | „masz na to godzinę, potem oddajesz, co masz" |
| **Sygnał przerwania** (abort signal) | sposób, w jaki kod przekazuje „przerwij pracę" wszystkim trwającym wywołaniom | kierownik woła „stop" i wszyscy odkładają narzędzia |

Dwa pojęcia z wcześniejszych modułów, które tu wracają: **ślad** (trace) to zapis przebiegu agenta krok po kroku z [Modułu 7](M7_WDROZENIE.md), a **ewaluacje** to zestaw przykładowych zadań z oczekiwanym wynikiem z [Modułu 4](M4_EWALUACJE.md), którym sprawdzasz agenta po każdej zmianie.

## Przykład: Ofertownik

Przypomnienie: Ofertownik przygotowuje szkice ofert szkoleniowych dla Ani. Ma narzędzia `search_price_list`, `find_similar_offers`, `check_calendar` (odczyt) i `save_offer_draft` (zapis szkicu jako propozycji).

Scenariusze S1–S3 (jedno szkolenie, pakiet dwóch, „to samo co rok temu") jeden agent robi bez trudu. Kłopot zaczyna się przy **S4**:

> „Dzień dobry, potrzebujemy programu rozwojowego dla trzech działów: sprzedaż (15 osób, wrzesień), księgowość (8 osób, październik) i magazyn (20 osób, listopad). Każdy dział ma inne potrzeby, szczegóły poniżej…"

**Jak to wygląda w jednym agencie.** Ślad przebiegu:

```text
krok 1  search_price_list("negocjacje sprzedaż")      → 5 pozycji
krok 2  search_price_list("obsługa klienta")          → 5 pozycji
krok 3  find_similar_offers("dział sprzedaży 15 osób") → 4 oferty
krok 4  check_calendar(wrzesień)                       → 11 terminów
krok 5  search_price_list("Excel księgowość")         → 5 pozycji
krok 6  find_similar_offers("księgowość 8 osób")      → 3 oferty
krok 7  check_calendar(październik)                    → 9 terminów
krok 8  [limit kroków] szkic: sprzedaż OK, księgowość z terminem wrześniowym,
        magazyn pominięty
```

Na „biurku" agenta leży siedem dużych wyników. Zanim doszedł do magazynu, skończyły mu się kroki, a po drodze pomylił terminy działów. Podniesienie limitu kroków nie pomaga: kontekst rośnie dalej, a wcześniejsze ustalenia giną pod nowymi wynikami.

**Jak to wygląda z podagentem na każdy dział.**

```text
orkiestrator, krok 1: trzy zlecenia naraz (sprzedaż / księgowość / magazyn)
   ├─ podagent „sprzedaż":    5 kroków → wraca 6 linijek wniosku
   ├─ podagent „księgowość":  4 kroki  → wraca 5 linijek wniosku
   └─ podagent „magazyn":     5 kroków → wraca 6 linijek wniosku
orkiestrator, krok 2: składa trzy wnioski w jeden szkic
orkiestrator, krok 3: save_offer_draft(...)
```

Każdy podagent ma własne, czyste biurko. Orkiestrator widzi 17 linijek wniosków zamiast kilkudziesięciu surowych wyników.

**Zlecenie złe i dobre.** Podagent nie widzi maila klienta ani tego, co ustalił orkiestrator. Wie tylko to, co dostanie w zleceniu.

| ❌ Złe zlecenie | ✅ Dobre zlecenie |
|---|---|
| „Sprawdź szkolenia dla działu sprzedaży." | „**Cel:** przygotuj propozycję szkoleń dla działu sprzedaży klienta (15 osób, termin: wrzesień). Klient pisze, że handlowcy słabo radzą sobie z negocjacjami cenowymi. Wynik trafi do zbiorczej oferty dla trzech działów. **Granice:** zajmujesz się wyłącznie sprzedażą; księgowość i magazyn robią inni. Nie proponuj rabatów i niczego nie zapisuj. **Źródła:** cennik (`search_price_list`), podobne wcześniejsze oferty (`find_similar_offers`), kalendarz trenerów na wrzesień (`check_calendar`). **Format wyniku:** najwyżej 2 szkolenia: nazwa, identyfikator pozycji cennika, cena za osobę; do 2 wolnych terminów; jedna podobna oferta, jeśli istnieje; lista rzeczy, których nie udało się ustalić. Najwyżej 10 linijek." |

Po złym zleceniu podagent nie wie, ile osób, na kiedy, w jakiej formie oddać wynik ani czy ma też sprawdzić terminy. Zgadnie, a orkiestrator dostanie trzy wyniki w trzech różnych formatach.

**Wynik pomiaru i decyzja** w `docs/MULTI_AGENT.md`:

```markdown
# Multi-agent: Ofertownik

## Objaw, który uzasadnił próbę
W 3 z 4 przebiegów scenariusza S4 wyniki narzędzi zapełniały kontekst:
agent mylił terminy działów albo pomijał trzeci dział (ślady z 12–14.11).

## Czy wystarczy prostszy wzorzec
Routing: tak, do rozdzielenia zapytań prostych od szerokich.
Łańcuch i równoległość w kodzie: nie w tej wersji, bo liczba i zakres części
zależy od treści maila (raz 3 działy, raz 2 działy i osobno kadra kierownicza).

## Porównanie - 18.11, commit 4f2c9a1 (24 przypadki ewaluacji)
| | Jeden agent | Orkiestrator + podagenci |
|---|---|---|
| Ewaluacje ogółem | 20/24 (83%) | 22/24 (92%) |
| Scenariusze szerokie (S4, 4 przypadki) | 1/4 | 3/4 |
| Scenariusze proste (S1–S3, 9 przypadków) | 9/9 | 9/9 |
| Koszt na zadanie (średni / najwyższy) | 0,014 / 0,048 USD | 0,039 / 0,110 USD |
| Czas zadania (mediana) | 18 s | 26 s |

## Decyzja
Routing: S1–S3 i pozostałe scenariusze obsługuje jeden agent,
zapytania typu S4 trafiają do orkiestratora.

## Uzasadnienie
W prostych scenariuszach wynik jest ten sam, a orkiestrator kosztuje prawie
trzy razy więcej. W S4 różnica jest wyraźna (1/4 → 3/4). Zapytania typu S4
to około jedno na dziesięć, ale mają najwyższą wartość. Koszt S4 przekracza
limit 0,05 USD z briefu: dla tej ścieżki ustawiam osobny limit 0,15 USD
na całe drzewo i najwyżej 4 podagentów na zadanie.
```

## Kroki

### Krok 1 — Sprawdź, czy naprawdę tego potrzebujesz

**Po co:** podagenci leczą dwa konkretne objawy. Na wszystkie inne nie działają, a kosztują tak samo. Zanim cokolwiek zbudujesz, sprawdź, czy masz któryś z tych dwóch.

**Co robisz:** otwórz ślady przebiegów z [Modułu 7](M7_WDROZENIE.md) (albo logi z Modułu 3) i poszukaj w nich objawów z tabeli:

| Objaw w śladach | Co oznacza | Czy to powód na podagentów |
|---|---|---|
| wyniki narzędzi zapełniają kontekst, a agent „zapomina" wcześniejsze ustalenia | jedno zadanie nie mieści się rozsądnie w jednym kontekście | **tak** — podagent przerabia dużo materiału i oddaje krótki wniosek |
| zadanie ma części niezależne (pięć źródeł, dziesięć dokumentów) wykonywane po kolei | czas rośnie liniowo bez potrzeby | **tak** — równoległość |
| agent myli narzędzia z dwóch różnych dziedzin | za dużo narzędzi w jednym kontekście | **może** — najpierw popraw opisy i spróbuj routingu |
| agent źle rozumuje na jednym, prostym kontekście | problem z instrukcjami albo modelem | **nie** — więcej agentów tego nie naprawi |
| kroki zależą jeden od drugiego i wymagają wspólnej wiedzy | zadanie jest sekwencyjne | **nie** — podział zniszczy wspólny kontekst |

Podagenci pomagają przy zadaniach **szerokich**: dużo niezależnego szukania i czytania. Szkodzą przy zadaniach, w których każda część musi wiedzieć, co robią pozostałe. Dlatego większość zadań programistycznych źle się dzieli.

W Ofertowniku S4 pasuje do pierwszego i drugiego wiersza: trzy działy to trzy niezależne rozpoznania, a ich wyniki zalewają kontekst.

Jeśli nie masz ani jednego wiersza z „tak", przeczytaj krok 2 i na nim zakończ. To też jest poprawny wynik tego modułu.

**Prompt do asystenta:**

```text
Sprawdzam, czy mój agent potrzebuje podagentów. Przeanalizuj ślady przebiegów.

Przeczytaj ślady z ostatnich uruchomień ewaluacji [wskaż tabelę albo folder,
np. docs/przebiegi/]. Zacznij od przypadków, które NIE przeszły, i od tych
z największą liczbą kroków. Dla każdego sprawdź pięć objawów:
1. Wyniki narzędzi zapełniają kontekst, a agent gubi wcześniejsze ustalenia.
2. Zadanie ma części niezależne od siebie, wykonywane po kolei.
3. Agent myli narzędzia z różnych dziedzin.
4. Agent źle rozumuje mimo krótkiego, prostego kontekstu.
5. Kroki zależą od siebie i wymagają wspólnej wiedzy.

Dla każdego znalezionego objawu podaj identyfikator przebiegu i numer kroku
jako dowód. Na końcu powiedz wprost: czy objawy 1 lub 2 występują i w których
scenariuszach. Jeśli widzisz tylko 3, 4 lub 5, napisz, że podagenci tego nie
naprawią i co poprawić zamiast tego.
Niczego nie zmieniaj w plikach. Nie czytaj i nie wypisuj wartości z plików .env.
```

**Gotowe, gdy:** masz zapisany w `docs/MULTI_AGENT.md` konkretny objaw z numerem przebiegu albo zdanie „brak objawów 1 i 2, zostaję przy jednym agencie".

<details>
<summary>🔬 Głębiej: dlaczego większość zadań programistycznych źle się dzieli</summary>

Podział pracy działa, gdy części są **niezależne w trakcie** i **łatwe do złożenia na końcu**. Rozpoznanie trzech działów w Ofertowniku spełnia oba warunki: wybór szkolenia dla księgowości nie zależy od tego, co wyszło dla magazynu, a złożenie to wklejenie trzech sekcji.

Zmiana w kodzie zwykle nie spełnia żadnego. Każda decyzja (nazwa funkcji, kształt typu, sposób obsługi błędu) jest ukrytym założeniem dla reszty. Dwóch podagentów piszących równolegle dwa moduły podejmuje te decyzje niezależnie i sprzecznie. Koszt nie znika, tylko przenosi się do etapu scalania, gdzie orkiestrator musi pogodzić dwa spójne wewnętrznie, ale niezgodne ze sobą fragmenty, mając w kontekście jedynie ich streszczenia.

Co w takich zadaniach dzieli się dobrze:

- **Czytanie przed pisaniem.** „Znajdź wszystkie miejsca, w których wołamy tę funkcję, i opisz, czego oczekują" to zadanie szerokie, niezależne i zwracające krótki wniosek.
- **Weryfikacja po pisaniu.** Uruchomienie testów, przegląd pod jednym konkretnym kątem. Recenzent z czystym kontekstem ma tu wręcz przewagę: nie odziedziczył założeń autora.
- **Prace na rozłącznych plikach z ustalonym z góry kontraktem.** Działa, ale kontrakt musi powstać *przed* podziałem i być zapisany, nie domyślny.

Ogólniejsza reguła: dziel według **granic informacji**, nie według ról. „Badacz, pisarz, recenzent" to podział na role i zwykle wychodzi z niego łańcuch. „Dział sprzedaży, dział księgowości, dział magazynu" to podział według tego, kto czego nie musi wiedzieć.

Miejsce na spór: z czasem okna kontekstu rosną, a modele lepiej radzą sobie z długim kontekstem. Część dzisiejszych uzasadnień dla podagentów („nie mieści się") będzie się starzeć. Uzasadnienie „równolegle jest szybciej" i „czysty kontekst daje niezależną ocenę" zostanie.

</details>

### Krok 2 — Najpierw spróbuj czegoś prostszego

**Po co:** między jednym agentem a zespołem agentów jest kilka rozwiązań, w których przepływem steruje **Twój kod**, a nie model. Są tańsze, szybsze i łatwiejsze do testowania. Często wystarczają.

**Co robisz:** przejrzyj pięć wzorców i zapisz, który rozwiązałby Twój problem.

| Wzorzec | Jak działa | Kiedy |
|---|---|---|
| **Łańcuch** | stała sekwencja wywołań, wynik jednego jest wejściem następnego | etapy są zawsze te same |
| **Routing** | pierwsze wywołanie klasyfikuje wejście i wybiera ścieżkę, model albo zestaw instrukcji | kilka wyraźnie różnych typów zadań |
| **Równoległość** | niezależne wywołania uruchomione jednocześnie, wyniki łączone w kodzie | te same operacje na wielu elementach |
| **Orkiestrator i wykonawcy** | model planuje podział, wykonawcy realizują części, wyniki są scalane | podział zależy od zadania i nie da się go zapisać z góry |
| **Oceniający i poprawiający** | jedno wywołanie tworzy, drugie ocenia według kryteriów, pętla z limitem prób | istnieje jasne kryterium jakości |

Typowa sytuacja: „potrzebuję agenta-badacza, agenta-pisarza i agenta-recenzenta" okazuje się łańcuchem trzech wywołań z jedną pętlą oceny. Żadnego zespołu, tylko trzy kroki w kodzie.

Idź dalej tylko wtedy, gdy potrzebujesz czwartego wzorca, czyli takiego, w którym **podział ustala model**, bo nie da się go zapisać z góry.

W Ofertowniku: routing przydaje się na pewno (odróżnić S1–S3 od S4). Sam podział S4 raz oznacza trzy działy, a raz dwa działy i osobny program dla kierowników, więc w tej wersji ustala go model.

**Prompt do asystenta:**

```text
Zanim zbuduję podagentów, chcę sprawdzić prostsze wzorce.

Mój problem (ze śladów): [wklej objaw z kroku 1 i numery scenariuszy].
Przeczytaj docs/ARCHITEKTURA.md i kod pętli agenta.

Dla każdego z pięciu wzorców oceń, czy rozwiązałby TEN problem:
1. Łańcuch - stała sekwencja wywołań.
2. Routing - pierwsze wywołanie klasyfikuje zadanie i wybiera ścieżkę.
3. Równoległość - niezależne wywołania naraz, wynik łączony w kodzie.
4. Orkiestrator i wykonawcy - model sam planuje podział.
5. Oceniający i poprawiający - pętla z kryterium jakości i limitem prób.

Dla każdego: tak / nie / częściowo, jedno zdanie uzasadnienia i szacunek,
ile dodatkowych wywołań modelu oznacza na jedno zadanie.
Bądź sceptyczny wobec wzorca 4: wskaż go tylko wtedy, gdy podziału naprawdę
nie da się zapisać w kodzie z góry, i napisz dlaczego.
Dopisz wynik do docs/MULTI_AGENT.md w sekcji „Czy wystarczy prostszy wzorzec".
Nie pisz jeszcze kodu.
```

**Gotowe, gdy:** w `docs/MULTI_AGENT.md` jest zapisane, który wzorzec wybierasz i dlaczego prostsze nie wystarczają (albo że wystarczają i na tym kończysz).

<details>
<summary>🔬 Głębiej: kiedy równoległość w kodzie bije orkiestratora</summary>

Orkiestrator płaci za elastyczność trzy razy: dodatkowym wywołaniem na zaplanowanie podziału, ryzykiem złego podziału i ryzykiem mętnego zlecenia. Jeśli podział da się **wyliczyć**, cała ta elastyczność jest kosztem bez korzyści.

Weź S4 z Ofertownika. Alternatywa dla orkiestratora wygląda tak:

1. Jedno wywołanie z ustrukturyzowanym wynikiem: „wypisz części zapytania: nazwa grupy, liczba osób, termin, potrzeby".
2. **Kod** uruchamia równolegle tego samego wykonawcę dla każdej pozycji z listy, z szablonem zlecenia wypełnianym polami z kroku 1.
3. Jedno wywołanie składa wyniki w szkic.

Co zyskujesz: zlecenie jest szablonem w repozytorium, więc zawsze ma cel, granice, źródła i format. Liczba wykonawców wynika z danych, a jej górną granicę egzekwuje zwykły `if`. Każdy etap testujesz osobno. Koszt jest przewidywalny.

Co tracisz: zdolność do reakcji na wynik pośredni. Orkiestrator, który dostał od podagenta „w cenniku nie ma niczego dla magazynu", może zlecić drugie, inne rozpoznanie. Sztywny potok tego nie zrobi, chyba że tę gałąź też zapiszesz.

Praktyczna heurystyka: zacznij od potoku w kodzie. Przejdź do orkiestratora dopiero, gdy w śladach widzisz powtarzające się przypadki, w których **stały podział był zły** albo **potrzebna była druga runda**. Wtedy masz też od razu gotowy punkt odniesienia do porównania w kroku 7. Trzy kolumny w tabeli (jeden agent, potok w kodzie, orkiestrator) mówią więcej niż dwie.

Dobrze też wiedzieć, że granica jest płynna: orkiestrator z regułą skali, limitem podagentów w kodzie i wymuszonym formatem zlecenia jest w połowie drogi do potoku. Im więcej ograniczeń dokładasz, tym bardziej warto zapytać, czy model jeszcze o czymkolwiek decyduje.

</details>

### Krok 3 — Zaprojektuj podział

**Po co:** podagent ma sens tylko wtedy, gdy ma jedno wyraźne zadanie i oddaje krótki wynik. Bez projektu powstaje „drugi agent do wszystkiego", który dubluje pierwszego.

**Co robisz:** podagent to **osobne wywołanie agenta z czystym kontekstem**, uruchamiane przez orkiestratora jako narzędzie. Cała wartość bierze się z izolacji: podagent może przeczytać pięćdziesiąt stron, a do orkiestratora wraca pół strony wniosków.

Dla każdego podagenta odpowiedz na pięć pytań. Tu wypełnione dla Ofertownika:

| Pytanie | Ofertownik: podagent „rozpoznanie działu" |
|---|---|
| **Jedno zadanie**, które wykonuje | „dobierz szkolenia i terminy dla jednej grupy uczestników" |
| **Narzędzia** — tylko te potrzebne do tego zadania | `search_price_list`, `find_similar_offers`, `check_calendar` |
| **Co dostaje** od orkiestratora | cel, zakres, format wyniku |
| **Co oddaje** | do 2 szkoleń (nazwa, identyfikator pozycji, cena za osobę), do 2 terminów, jedna podobna oferta, lista braków; najwyżej 10 linijek |
| **Limit** kroków i model | 6 kroków; tańszy model niż orkiestrator |

**Zasada uprawnień: narzędzia zapisujące i wymagające zatwierdzenia zostają u orkiestratora.** Podagenci czytają i wnioskują. Są dwa powody:

- w wielu bibliotekach mechanizm zatwierdzania przez człowieka i tak nie działa wewnątrz podagenta,
- nawet gdyby działał, akcja podjęta głęboko w drzewie wywołań jest najtrudniejsza do prześledzenia.

W Ofertowniku `save_offer_draft` ma tylko orkiestrator. `send_offer` nie ma nikt: wysyłkę wykonuje zwykły kod po zatwierdzeniu przez Anię.

**Prompt do asystenta:**

```text
Projektuję podział pracy między orkiestratora i podagentów. Na razie tylko
dokument, bez kodu.

Problem do rozwiązania: [objaw i scenariusze z kroku 1].
Przeczytaj docs/ARCHITEKTURA.md (lista narzędzi) i docs/MULTI_AGENT.md.

Zaproponuj NAJMNIEJSZĄ liczbę rodzajów podagentów (najlepiej jeden).
Dla każdego wypełnij tabelę:
- jedno zadanie, które wykonuje (jedno zdanie),
- narzędzia: tylko te potrzebne do tego zadania, wyłącznie odczyt,
- co dostaje od orkiestratora,
- co oddaje: dokładny format i maksymalna długość,
- limit kroków i model (tańszy niż u orkiestratora, jeśli to możliwe).

Osobno wypisz, które narzędzia zostają WYŁĄCZNIE u orkiestratora. Wszystkie
narzędzia zapisujące i wymagające zatwierdzenia przez człowieka muszą być
na tej liście. Jeśli któryś podagent potrzebowałby zapisu, zatrzymaj się
i zapytaj mnie.
Dopisz projekt do docs/MULTI_AGENT.md w sekcji „Podział".
```

**Gotowe, gdy:** każdy podagent ma jedno zadanie w jednym zdaniu, a na jego liście narzędzi nie ma żadnego zapisującego.

### Krok 4 — Napisz zlecenie tak, jak dla człowieka

**Po co:** najczęstsza przyczyna porażki systemów wieloagentowych to nie model, tylko **mętne zlecenie**. Podagent nie widzi rozmowy z użytkownikiem ani ustaleń orkiestratora. Wie tylko to, co dostanie w zleceniu. „Zbadaj temat X" kończy się tym, że trzech podagentów robi to samo, a czwartego brakuje.

**Co robisz:** zleceń nie piszesz Ty, tylko orkiestrator, w trakcie pracy. Twoim zadaniem jest tak napisać **instrukcje orkiestratora**, żeby każde jego zlecenie zawierało cztery elementy:

1. **cel** — co ma zostać ustalone i po co,
2. **granice** — czego nie robić, czym zajmują się inni,
3. **źródła i narzędzia** — gdzie szukać,
4. **format wyniku** — dokładnie to, co ma wrócić, i jak długie.

Przykład dobrego i złego zlecenia masz w sekcji „Przykład: Ofertownik" wyżej. Prosty sprawdzian: czy nowy pracownik, który nie był na zebraniu, wykonałby to zlecenie bez zadawania pytań?

Dopisz też regułę **skali**: ile podagentów na jakie zadanie. Bez niej orkiestrator uruchomi pięciu do pytania, na które wystarcza jedno wyszukanie. Przykład ogólny: proste pytanie — bez podagentów; porównanie kilku rzeczy — 2–3 podagentów; szerokie rozpoznanie — do 5.

Reguła skali Ofertownika:

```text
- Zapytanie o jedną grupę uczestników: NIE zlecaj, zrób to sam.
- Zapytanie o 2–4 odrębne grupy (działy, lokalizacje): jeden podagent na grupę.
- Więcej niż 4 grupy: nie zlecaj; przygotuj pytania do klienta o priorytety.
```

**Prompt do asystenta:**

```text
Napisz instrukcje systemowe dla orkiestratora mojego agenta.
Przeczytaj obecne instrukcje agenta oraz sekcję „Podział" w docs/MULTI_AGENT.md.

Instrukcje orkiestratora mają zawierać:
1. Regułę skali: kiedy NIE zlecać niczego, kiedy zlecić 2–3 podagentom,
   jaki jest górny limit. Zaproponuj progi dla mojego zadania i pokaż mi je
   przed zapisaniem.
2. Wymagania wobec zlecenia. Każde zlecenie MUSI zawierać cztery elementy:
   cel (co ustalić i po co), granice (czego nie robić, czym zajmują się inni),
   źródła i narzędzia (gdzie szukać), format wyniku (co dokładnie ma wrócić
   i jak długie). Podagent nie widzi rozmowy z użytkownikiem - zlecenie musi
   być zrozumiałe bez niej.
3. Jeden przykład dobrego zlecenia i jeden złego, z mojej dziedziny.
4. Co robić z wynikami: jak je złożyć i co zrobić, gdy podagent zgłosi braki.
5. Przypomnienie, że zapisuje wyłącznie orkiestrator.

Zachowaj wszystkie dotychczasowe zasady agenta (zakazy, limity, traktowanie
treści od użytkownika jako danych). Zapisz instrukcje w osobnym pliku obok
obecnych, niczego nie usuwaj. Nie pisz jeszcze kodu podagentów.
```

**Gotowe, gdy:** instrukcje orkiestratora zawierają regułę skali z liczbami i cztery wymagane elementy zlecenia.

### Krok 5 — Zbuduj orkiestratora z jednym podagentem

**Po co:** jeden rodzaj podagenta wystarczy, żeby zmierzyć, czy cały pomysł coś daje. Więcej dołożysz, gdy pomiar to uzasadni.

**Co robisz:** w Vercel AI SDK agentem jest `ToolLoopAgent`, a podagent to zwykłe narzędzie, którego funkcja uruchamia innego agenta. Kształt rozwiązania dla Ofertownika:

```ts
import { ToolLoopAgent, tool, isStepCount } from "ai";
import { z } from "zod";

const departmentResearcher = new ToolLoopAgent({
  model: workerModel,                       // tańszy model z konfiguracji
  instructions:
    "Wykonujesz jedno zlecenie: dobór szkoleń i terminów dla jednej grupy. Na końcu ZAWSZE podaj zwięzłe podsumowanie w formacie ze zlecenia.",
  tools: { search_price_list, find_similar_offers, check_calendar }, // tylko odczyt
  stopWhen: isStepCount(6),
});

const delegateDepartment = tool({
  description:
    "Zleca rozpoznanie potrzeb jednej grupy uczestników podagentowi z czystym kontekstem. Użyj, gdy zapytanie dotyczy kilku odrębnych grup. Zlecenie musi zawierać: cel, granice, źródła, format wyniku.",
  inputSchema: z.object({ task: z.string().min(40) }),
  execute: async ({ task }, { abortSignal }) => {
    const result = await departmentResearcher.generate({ prompt: task, abortSignal });
    return result.text;                     // do orkiestratora wraca tylko wniosek
  },
});

const orchestrator = new ToolLoopAgent({
  model: leadModel,
  instructions: orchestratorInstructions,   // z regułą skali i wymaganiami wobec zlecenia
  tools: { delegateDepartment, save_offer_draft },
  stopWhen: isStepCount(8),
});
```

> ⚠️ Fragment pokazuje **kształt** rozwiązania według dokumentacji AI SDK z listopada 2026. Sprawdź https://ai-sdk.dev/docs/agents/subagents dla wersji, którą masz zainstalowaną — znajdziesz tam także strumieniowanie postępu podagenta do interfejsu i zawężanie tego, co wraca do modelu nadrzędnego.

Cztery rzeczy, których ten szkic pilnuje i których musisz pilnować w każdym stacku:

- **limit kroków na każdym poziomie** — domyślne limity bibliotek bywają wysokie (w AI SDK 20 kroków); orkiestrator × podagenci × kroki mnoży się szybko. Tu: 8 kroków orkiestratora, 6 kroków podagenta.
- **sygnał przerwania przekazany w dół** — `abortSignal` trafia do podagenta, więc anulowanie zadania zatrzymuje także jego. Bez tego użytkownik klika „stop", a podagenci pracują i kosztują dalej.
- **wraca wniosek, nie cała praca** — `result.text`, nie pełna historia kroków. Inaczej izolacja kontekstu nic nie daje.
- **podagent kończy podsumowaniem** — wymuszone w jego instrukcjach, bo bez tego orkiestrator dostaje ostatnią myśl zamiast wyniku.

**Równoległość i odmowy.** Jeśli model wywoła kilka narzędzi `delegateDepartment` w jednym kroku, podagenci ruszają równolegle. Przy wielu równoległych wywołaniach dostawca może chwilowo odmówić części z nich (limity zależne od modelu, a w OpenRouterze także rezerwacja środków na trwające żądania). Obsłuż to ponowieniem po krótkiej przerwie, z limitem prób.

**Prompt do asystenta:**

```text
Zaimplementuj orkiestratora z JEDNYM rodzajem podagenta, według sekcji
„Podział" w docs/MULTI_AGENT.md i instrukcji orkiestratora z poprzedniego kroku.

NAJPIERW przeczytaj dokumentację AI SDK o podagentach:
https://ai-sdk.dev/docs/agents/subagents
oraz sprawdź w package.json, którą wersję pakietu ai mam zainstalowaną.
Jeśli dokumentacja dla tej wersji różni się od tego, co pamiętasz,
obowiązuje dokumentacja. Napisz mi w dwóch zdaniach, co znalazłeś.

Wymagania:
1. Podagent: osobny agent z tańszym modelem z konfiguracji, wyłącznie
   narzędzia odczytu, limit kroków z dokumentu, instrukcje kończące się
   wymogiem zwięzłego podsumowania w formacie ze zlecenia.
2. Narzędzie zlecające: przyjmuje tekst zlecenia (z minimalną długością),
   uruchamia podagenta, przekazuje mu sygnał przerwania i zwraca TYLKO tekst
   końcowego wniosku.
3. Orkiestrator: mocniejszy model, własny limit kroków, narzędzie zlecające
   plus narzędzia zapisujące. Podagent NIE dostaje narzędzi zapisujących.
4. W kodzie narzędzia zlecającego: maksymalna liczba podagentów na jedno
   zadanie (z reguły skali) oraz ponowienie po krótkiej przerwie, z limitem
   prób, gdy dostawca chwilowo odmówi.
5. Dotychczasowa wersja z jednym agentem ma zostać i dać się uruchomić
   przełącznikiem w konfiguracji - będę porównywać obie.

Nie uruchamiaj jeszcze pełnych ewaluacji. Pokaż jeden przebieg na najtrudniejszym
scenariuszu i wskaż plik:linię z limitami kroków.
Nie czytaj i nie wypisuj wartości z plików .env.
```

**Gotowe, gdy:** jeden przebieg trudnego scenariusza kończy się wynikiem, a Ty umiesz wskazać w kodzie limit kroków orkiestratora, limit kroków podagenta i limit liczby podagentów.

<details>
<summary>🔬 Głębiej: co wraca do orkiestratora i ile informacji ginie w streszczeniu</summary>

Izolacja kontekstu to kompresja stratna. Podagent przeczytał kilkadziesiąt wyników, a oddaje dziesięć linijek. Wszystko, czego w nich nie ma, **dla orkiestratora nie istnieje**. To jest zarazem cała korzyść i całe ryzyko wzorca.

Typowe straty:

- **Niepewność.** Podagent znalazł pasujące szkolenie „mniej więcej". W podsumowaniu zostaje nazwa i cena, znika „mniej więcej". Orkiestrator składa ofertę z pewnością, której nikt nie miał. Lekarstwo: pole „czego nie udało się ustalić" i „na ile pasuje" w wymaganym formacie wyniku.
- **Odrzucone alternatywy.** Podagent rozważył trzy szkolenia i wybrał jedno. Jeśli wybrane okaże się niedostępne w terminie pozostałych działów, orkiestrator nie wie, co było drugie w kolejce, i zleca rozpoznanie od nowa.
- **Źródła.** Streszczenie mówi „cena 450 zł za osobę". Jeśli nie niesie identyfikatora pozycji cennika, na końcu nie da się sprawdzić, że cena pochodzi z cennika. W Ofertowniku to złamanie miary „0 ofert z ceną spoza cennika", więc identyfikator jest obowiązkowym polem wyniku, a końcową weryfikację cen robi kod, nie model.
- **Zależności między gałęziami.** Trzy podagenty niezależnie wybrały tego samego trenera na ten sam tydzień. Każdy wynik z osobna jest poprawny. Konflikt widać dopiero po złożeniu, o ile orkiestrator dostał terminy i nazwiska, a nie „termin dostępny".

Dwie strategie formatu wyniku i ich koszty:

- **Wynik ustrukturyzowany** (pola, identyfikatory): mało ginie z tego, co przewidziałeś, wszystko ginie z tego, czego nie przewidziałeś. Dobre, gdy wiesz, czego orkiestrator potrzebuje.
- **Wynik prozą z limitem długości:** elastyczny, ale orkiestrator musi go interpretować i robi to gorzej, niż się wydaje, gdy trzy wyniki mają trzy różne układy.

Wariant pośredni wart rozważenia: pełna praca podagenta zostaje zapisana w śladach albo w bazie, a do orkiestratora wraca streszczenie **z odnośnikiem**. Orkiestrator może sięgnąć po szczegół narzędziem odczytu, gdy naprawdę go potrzebuje. Kosztuje to dodatkowe narzędzie i dyscyplinę, żeby orkiestrator nie czytał wszystkiego „na wszelki wypadek".

Sprawdzian jakości streszczenia, który da się zautomatyzować: weź wynik podagenta i zadaj o niego trzy pytania, na które orkiestrator będzie musiał umieć odpowiedzieć przy składaniu. Jeśli odpowiedzi nie da się wyczytać z wyniku, format jest za wąski.

</details>

### Krok 6 — Rozszerz ślady

**Po co:** przebieg z podagentami to drzewo, nie lista. Jeśli Twoje ślady pokazują tylko płaską listę kroków, przy pierwszym złym wyniku nie dojdziesz, kto zawinił: kierownik, który źle zlecił, czy pracownik, który źle wykonał.

**Co robisz:** do śladów z [Modułu 7](M7_WDROZENIE.md) dodaj trzy rzeczy:

- `parent_run_id` — każdy przebieg podagenta wskazuje wywołanie orkiestratora, które go uruchomiło,
- **treść zlecenia** i **treść wyniku** podagenta — to tu szukasz przyczyn błędów,
- koszt i czas **sumowane w górę** drzewa.

Gdy wynik jest zły, czytaj w tej kolejności: **zlecenie → wynik podagenta → jego kroki**. W większości przypadków problem widać już w zleceniu.

Przy tej okazji zaktualizuj budżety z [Modułu 6](M6_BEZPIECZENSTWO_I_KOSZTY.md): limit kosztu **na całe drzewo**, nie na pojedyncze wywołanie, oraz maksymalną liczbę podagentów na zadanie egzekwowaną w kodzie narzędzia zlecającego. Limit na pojedyncze wywołanie niczego tu nie chroni: dziesięć tanich wywołań daje jedno drogie zadanie.

**Prompt do asystenta:**

```text
Rozszerz ślady przebiegów tak, żeby pokazywały drzewo orkiestrator → podagenci.

Przeczytaj obecny kod zapisu śladów i migrację tabeli przebiegów. Potem:
1. Dodaj do tabeli przebiegów kolumnę parent_run_id (puste dla orkiestratora;
   u podagenta wskazuje przebieg orkiestratora, który go uruchomił).
2. Dla każdego przebiegu podagenta zapisuj pełną treść zlecenia i pełną treść
   wyniku, który wrócił do orkiestratora.
3. Koszt i czas: zapisuj dla każdego przebiegu osobno ORAZ przygotuj zapytanie,
   które sumuje je w górę drzewa dla podanego przebiegu głównego.
4. Limit kosztu: sprawdzaj łączny koszt CAŁEGO drzewa przed każdym kolejnym
   wywołaniem modelu (orkiestratora i podagentów). Po przekroczeniu przerwij
   całe drzewo sygnałem przerwania.
5. Dodaj prosty widok albo skrypt: dla identyfikatora przebiegu wypisz drzewo:
   orkiestrator → zlecenia → wyniki, z kosztem i czasem każdej gałęzi i sumą.

Do śladów nie trafiają klucze ani zawartość plików .env.
Nie czytaj i nie wypisuj wartości z plików .env.
```

**Gotowe, gdy:** dla dowolnego przebiegu umiesz pokazać drzewo: orkiestrator → zlecenia → wyniki, z kosztem zsumowanym.

<details>
<summary>🔬 Głębiej: debugowanie drzewa przebiegów</summary>

Błędy w systemie wieloagentowym mają inną naturę niż w pojedynczym agencie: rzadko jeden krok jest wyraźnie zły. Częściej każdy węzeł zachował się rozsądnie wobec tego, co dostał, a wynik końcowy i tak jest zły. Kilka technik, które skracają szukanie:

- **Czytaj od krawędzi, nie od węzłów.** Zlecenie i wynik to interfejsy między agentami i tam siedzi większość błędów. Kroki wewnątrz podagenta czytaj na końcu. Warto mieć widok, który pokazuje same krawędzie całego drzewa na jednym ekranie.
- **Odtwarzaj jedną gałąź.** Mając zapisane zlecenie, możesz uruchomić samego podagenta z tym samym tekstem, bez orkiestratora. Jeśli wynik jest dobry przy pięciu powtórzeniach, wina leży w składaniu. Jeśli jest chwiejny, wina leży w zleceniu albo w podagencie. To zamienia nierozstrzygalne „coś nie działa" w dwa osobne, tanie testy.
- **Ewaluuj zlecenia osobno.** Zlecenie jest tekstem, więc da się je ocenić rubryką (cel, granice, źródła, format: jest / nie ma) bez uruchamiania podagentów. To najtańsza ewaluacja w całym systemie i wyłapuje najczęstszą klasę błędów.
- **Szukaj duplikatów i dziur.** Dwa zlecenia o prawie tej samej treści albo część zadania, której nikt nie dostał. Proste porównanie treści zleceń w obrębie jednego drzewa wyłapuje pierwsze; drugie wymaga porównania z listą części, którą orkiestrator sam wypisał w planie.
- **Niedeterminizm się mnoży.** Jeśli każdy węzeł daje dobry wynik w 9 na 10 uruchomień, drzewo z czterech węzłów udaje się w całości wyraźnie rzadziej. Jeden przebieg niczego nie dowodzi. Porównuj rozkłady z kilku powtórzeń na przypadek, zwłaszcza dla scenariuszy szerokich.
- **Koszt jako sygnał.** Gałąź, która kosztowała trzy razy więcej niż pozostałe przy tym samym typie zlecenia, prawie zawsze oznacza zapętlenie albo zbyt szerokie zlecenie. Alarm na koszt gałęzi wykrywa problemy jakości wcześniej niż ewaluacje.

Jedna pułapka organizacyjna: gdy ślady zawierają pełne treści zleceń i wyników, zawierają też dane użytkowników (w Ofertowniku: treść zapytań klientów). Czas przechowywania i dostęp do śladów stają się tematem z [Modułu 6](M6_BEZPIECZENSTWO_I_KOSZTY.md), nie tylko wygodą programisty.

</details>

### Krok 7 — Zmierz i zdecyduj

**Po co:** bez pomiaru nie wiesz, czy zbudowałeś coś lepszego, czy tylko droższego. Ten krok jest jedynym, który odpowiada na pytanie z początku modułu.

**Co robisz:** uruchom **te same ewaluacje** na wersji z jednym agentem i na wersji z podagentami. Zapisz w `docs/MULTI_AGENT.md` tabelę jak w przykładzie Ofertownika: skuteczność ogółem, skuteczność w scenariuszach szerokich i prostych osobno, koszt na zadanie (średni i najwyższy), czas zadania (mediana, czyli wartość środkowa: połowa zadań trwa krócej, połowa dłużej).

Dlaczego osobno szerokie i proste: wynik ogółem może ukryć, że w prostych scenariuszach nic się nie zmieniło, a płacisz trzy razy więcej.

Trzy uczciwe wyniki tego porównania:

- **lepiej i warto** — wersja z podagentami zostaje,
- **lepiej tylko w części scenariuszy** — routing: proste zadania idą do jednego agenta, szerokie do orkiestratora (tak wyszło w Ofertowniku),
- **nie lepiej albo za drogo** — wracasz do jednego agenta i masz to udokumentowane liczbami. **To pełnoprawny rezultat.** Wiesz coś, czego nie wiedziałeś, i nie utrzymujesz systemu, który nic nie daje.

> 🛑 Wersja wieloagentowa potrafi zużyć kilkukrotnie więcej tokenów na tych samych ewaluacjach. Na kluczu kursowym uruchom porównanie na podzbiorze przypadków i na najtańszych modelach, które dają sensowny wynik.

**Prompt do asystenta:**

```text
Porównaj wersję z jednym agentem i wersję z orkiestratorem na TYCH SAMYCH
ewaluacjach.

1. Zanim cokolwiek uruchomisz: policz, ile wywołań modelu oznacza pełne
   porównanie, i podaj szacunkowy koszt. Zaproponuj podzbiór przypadków
   (wszystkie szerokie + po kilka z pozostałych kategorii) i najtańsze modele,
   które dają sensowny wynik. Czekaj na moją zgodę.
2. Po zgodzie uruchom ten sam podzbiór na obu wersjach, przełączając je
   w konfiguracji. Nie zmieniaj przy tym instrukcji ani przypadków.
3. Dopisz do docs/MULTI_AGENT.md tabelę „Porównanie - <data>, commit <id>":
   - ewaluacje ogółem,
   - scenariusze szerokie osobno, scenariusze proste osobno,
   - koszt na zadanie (średni / najwyższy), liczony dla całego drzewa,
   - czas zadania (mediana).
4. Wypisz przypadki, w których wersje dały RÓŻNY wynik, z identyfikatorami
   przebiegów.
5. Zaproponuj jedną z trzech decyzji i uzasadnij liczbami:
   zostaje / routing (które scenariusze dokąd) / powrót do jednego agenta.
   „Powrót do jednego agenta" traktuj jako równie dobry wynik jak pozostałe.
   Nie naciągaj wniosków: jeśli różnica mieści się w rozrzucie między
   uruchomieniami, napisz to wprost.

Decyzję i uzasadnienie zapiszę sam. Nie czytaj i nie wypisuj wartości z plików .env.
```

**Gotowe, gdy:** w `docs/MULTI_AGENT.md` jest tabela porównania na tych samych przypadkach oraz decyzja z uzasadnieniem w dwóch–trzech zdaniach.

## Najczęstsze potknięcia

- **Podagenci „bo tak się teraz robi".** Objaw: nie umiesz wskazać w śladach objawu, który mają leczyć. → Wróć do kroku 1. Bez objawu nie ma czego mierzyć.
- **Mętne zlecenia.** Objaw: dwóch podagentów zwraca to samo albo wyniki mają różne formaty. → Otwórz treść zleceń w śladach. Brakuje granic albo formatu wyniku; popraw instrukcje orkiestratora (krok 4).
- **Orkiestrator zleca wszystko.** Objaw: proste zapytanie uruchamia trzech podagentów, koszt rośnie kilkukrotnie. → Dopisz regułę skali i limit liczby podagentów w kodzie narzędzia zlecającego.
- **Do orkiestratora wraca cała praca.** Objaw: kontekst orkiestratora jest tak samo pełny jak wcześniej u jednego agenta. → Zwracaj tylko końcowy wniosek i wymuś podsumowanie w instrukcjach podagenta.
- **Brak limitu na drzewo.** Objaw: jedno zadanie kosztuje dziesięć razy więcej niż zwykle, choć żadne pojedyncze wywołanie nie przekroczyło limitu. → Limit kosztu licz dla całego drzewa (krok 6).
- **Podagent z narzędziem zapisu.** Objaw: szkic zapisany dwa razy albo zapis bez zatwierdzenia. → Narzędzia zapisujące zostają wyłącznie u orkiestratora.

## ✅ Sprawdź, czy masz wszystko

- [ ] `docs/MULTI_AGENT.md` zawiera objaw ze śladów, który uzasadnił próbę (albo stwierdzenie, że objawu nie ma).
- [ ] Zapisane, który z pięciu wzorców wybierasz i dlaczego prostsze nie wystarczają.
- [ ] Podagenci nie mają narzędzi zapisujących ani wymagających zatwierdzenia — umiesz wskazać `plik:linia`.
- [ ] Limit kroków istnieje na każdym poziomie, a limit liczby podagentów i kosztu całego drzewa jest w kodzie.
- [ ] Sygnał przerwania jest przekazywany do podagentów.
- [ ] Dla dowolnego przebiegu umiesz pokazać drzewo: orkiestrator → zlecenia → wyniki, z kosztem zsumowanym.
- [ ] Tabela porównania na tych samych przypadkach i decyzja (zostaje / routing / powrót) z uzasadnieniem.

## 🤖 Poproś asystenta o recenzję

```text
Przeczytaj instrukcje orkiestratora i podagentów, kod narzędzia zlecającego,
jeden przykładowy ślad przebiegu z podagentami oraz docs/MULTI_AGENT.md.
Oceń sceptycznie, jak ktoś, kto widział wiele niepotrzebnie skomplikowanych
systemów. Dla każdego punktu napisz PASS albo FAIL i jedno zdanie uzasadnienia:

1. Czy objaw, który miał uzasadnić podagentów, rzeczywiście wynika z ograniczeń
   jednego kontekstu, a nie ze słabych instrukcji lub opisów narzędzi?
2. Czy ten sam efekt dałby łańcuch, routing albo równoległe wywołania
   sterowane kodem?
3. Czy zlecenia w śladzie zawierają cel, granice, źródła i format wyniku?
4. Czy podagent może wykonać akcję zapisującą albo ominąć zatwierdzenie?
5. Czy różnica w skuteczności uzasadnia różnicę w koszcie i czasie?

Niczego nie zmieniaj w plikach. Na końcu podaj jedną rzecz,
którą poprawiłbyś w pierwszej kolejności.
```

## 🚀 Dla chętnych

- 🔵 **Oceniający i poprawiający.** Dodaj wywołanie, które ocenia wynik końcowy według rubryki i odsyła do poprawy, z limitem dwóch prób. Zmierz, ile to daje i ile kosztuje.
- 🔵 **Różne modele na różnych poziomach.** Mocniejszy model jako orkiestrator, najtańszy u podagentów, a potem odwrotnie. Porównaj na ewaluacjach.
- 🟣 **Trzecia kolumna.** Zbuduj wariant „równoległość w kodzie": jedno wywołanie wypisuje części zadania, kod uruchamia wykonawców, jedno wywołanie składa wynik. Dopisz go jako trzecią kolumnę do tabeli porównania. W których przypadkach orkiestrator wygrywa i czy jest ich dość, żeby uzasadnić jego koszt?
- 🟣 **Podagent jako serwer MCP.** Wystaw podagenta przez [własny serwer MCP](T3_WLASNY_SERWER_MCP.md) i pozwól wywołać go asystentowi kodującemu. Kto wtedy płaci za jego kroki i gdzie egzekwujesz limit?

## Co dalej

Wiesz już, ile agentów naprawdę potrzebuje Twoje zadanie. Jeśli chcesz, żeby za agenta dało się zapłacić, zobacz [Moduł techniczny 5: Płatności — Stripe](T5_PLATNOSCI_STRIPE.md).
