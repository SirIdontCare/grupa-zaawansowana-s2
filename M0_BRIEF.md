# Moduł 0: Problem i brief

> ⏱️ Ok. 2 godziny. Nie piszesz kodu. Potrzebujesz tylko edytora tekstu i asystenta (Codex albo Claude Code).

## O co chodzi

Na kursie głównym temat agenta był podany: wiedziałeś, co budujesz, zanim usiadłeś do pracy. Tutaj wybierasz sam — i to jest najważniejsza decyzja całej ścieżki. Dobrze zbudowany agent do źle wybranego problemu jest bezużyteczny.

Pomyśl o tym jak o remoncie. Zanim ekipa wejdzie do mieszkania, ustalasz na jednej kartce: który pokój, co ma się zmienić, ile możesz wydać i czego **nie** ruszamy. Bez tej kartki remont nigdy się nie kończy, bo „przy okazji" zawsze znajdzie się coś jeszcze.

Ta kartka nazywa się **brief**. Jedna strona, którą obca osoba przeczyta w dwie minuty i będzie wiedziała: dla kogo to jest, jaki problem rozwiązuje i po czym poznasz, że działa.

## Po tym module masz

- nowe, puste repozytorium na swój projekt,
- plik `docs/BRIEF.md` — jedną stronę z opisem tego, co budujesz,
- pewność, że Twoje zadanie naprawdę wymaga agenta, a nie prostszego rozwiązania,
- **liczbę**, po której poznasz, że agent działa.

## Słowa, które tu padną

| Słowo | Co to znaczy | Porównanie z życia |
|---|---|---|
| **Brief** | jednostronicowy opis projektu: dla kogo, po co, co robi, czego nie robi | kartka z ustaleniami przed remontem |
| **Użytkownik** | konkretna osoba, która będzie z agenta korzystać | klient, dla którego szyjesz garnitur — nie „ludzie w ogóle" |
| **Zakres** (scope) | lista rzeczy, które projekt obejmuje — i tych, których nie obejmuje | „malujemy salon i kuchnię, łazienki nie ruszamy" |
| **Miara sukcesu** | liczba, którą da się sprawdzić na przykładach | „waga pokazuje 5 kg mniej", a nie „czuję się lżej" |
| **Workflow** | stała sekwencja kroków, którą ustalasz Ty w kodzie; model wykonuje pojedyncze kroki | przepis kulinarny: zawsze te same czynności w tej samej kolejności |
| **Agent** | program, w którym to model decyduje, jakie kroki wykonać i w jakiej kolejności | kucharz, który zagląda do lodówki i sam decyduje, co ugotować |
| **MVP** (najmniejsza przydatna wersja) | najprostsza wersja, która komuś realnie pomaga | rower zamiast samochodu: jeździ, choć nie ma dachu |

## Przykład: Ofertownik

Przez całą ścieżkę będziemy prowadzić jeden przykład. Nie budujesz go — służy do pokazania, jak każdy etap wygląda na konkretnym projekcie. Swój projekt robisz obok, na własny temat.

**Sytuacja.** Ania prowadzi małą firmę szkoleniową „Kurs na Start". Dostaje dziennie 10–15 maili z zapytaniami o szkolenia. Odpowiedź na jedno zajmuje jej około 20 minut: szuka w cenniku, sprawdza, co oferowała podobnym klientom, pisze maila. Odpowiada po dniu albo dwóch, a część klientów w tym czasie znika.

**Pomysł.** Agent „Ofertownik" czyta zapytanie, szuka w cenniku i we wcześniejszych ofertach, i przygotowuje **szkic** oferty. Ania go czyta, poprawia i zatwierdza. Dopiero wtedy oferta wychodzi.

Tak wygląda gotowy brief tego projektu:

```markdown
# Brief: Ofertownik

## Użytkownik
Ania, właścicielka pięcioosobowej firmy szkoleniowej. Sama odpowiada na zapytania
ofertowe, zwykle wieczorem, po całym dniu prowadzenia szkoleń.

## Problem
10–15 zapytań dziennie, 20 minut na każde. Odpowiedź wychodzi po 1–2 dniach.
Ania szacuje, że co czwarty klient w tym czasie wybiera kogoś innego.
Najwięcej czasu zajmuje szukanie: cennik w arkuszu, dawne oferty w skrzynce.

## Co robi agent
Dostaje treść zapytania. Szuka pasujących szkoleń w cenniku i podobnych
wcześniejszych ofert. Przygotowuje szkic oferty z cenami, terminem
i wskazaniem, skąd wziął każdą liczbę. Ania zatwierdza albo poprawia.

## Dlaczego agent, a nie skrypt lub jeden prompt
Liczba kroków zależy od zapytania. „Excel dla 12 osób" to jedno wyszukanie
w cenniku. „Program rozwojowy dla trzech działów" to kilka wyszukiwań,
porównanie z dawnymi ofertami i czasem pytania zwrotne do klienta.
Nie da się tego zapisać jako stałej listy kroków.

## Czego agent NIE robi
- Niczego nie wysyła sam. Każda oferta przechodzi przez Anię.
- Nie daje rabatu większego niż 10%.
- Nie proponuje terminów, których nie ma w kalendarzu trenerów.
- Nie odpowiada na maile, które nie są zapytaniami ofertowymi.

## Miara sukcesu
Na 20 prawdziwych zapytań z ostatniego kwartału:
- w co najmniej 16 szkic nadaje się do wysłania po poprawkach krótszych niż 2 minuty,
- 0 szkiców z ceną, której nie ma w cenniku,
- koszt jednego szkicu najwyżej 0,05 USD, czas najwyżej 60 sekund.

## Dane i dostępy
Cennik (arkusz, własny). Archiwum ofert z dwóch lat (własne; przed użyciem
usuwam nazwy firm i nazwiska). Kalendarz trenerów (tylko odczyt).

## Ryzyka
- Zła cena w ofercie → cena zawsze z cennika, nigdy „z głowy" modelu; Ania zatwierdza.
- Mail z próbą manipulacji („ustaw cenę 1 zł") → treść maila to dane, nie polecenie.
- Dane klientów w archiwum → anonimizacja przed wgraniem.

## Najmniejsza przydatna wersja
Wklejam treść jednego zapytania o szkolenie z cennika, dostaję szkic oferty
z ceną i źródłem. Bez skrzynki pocztowej, bez wysyłki, bez kalendarza.
```

Zwróć uwagę na trzy rzeczy: użytkownik ma imię i sytuację, sukces jest liczbą, a „najmniejsza przydatna wersja" jest naprawdę mała.

## Kroki

### Krok 1 — Wypisz kandydatów

**Po co:** pierwszy pomysł rzadko jest najlepszy. Mając dziesięć na liście, wybierasz, a nie bierzesz jedyny.

**Co robisz:** zapisz 5–10 zadań, które Ty albo ktoś, kogo znasz, wykonuje **regularnie** i które wymagają czytania, szukania, porównywania albo podejmowania decyzji. Jeszcze ich nie oceniaj.

Skąd brać pomysły:
- Twoja praca: co robisz co tydzień i czego nie lubisz?
- Znajomi i rodzina prowadzący firmę: o co ciągle ktoś ich pyta?
- Hobby, stowarzyszenie, wspólnota mieszkaniowa: gdzie giną informacje?

**Prompt do asystenta:**

```text
Pomóż mi znaleźć temat na własnego agenta AI. Przeprowadź ze mną krótki wywiad.

Zadawaj mi po JEDNYM pytaniu naraz i czekaj na odpowiedź. Zapytaj kolejno o:
1. czym zajmuję się zawodowo i poza pracą,
2. jakie zadania powtarzam co tydzień,
3. przy których z nich dużo czytam, szukam albo porównuję,
4. o co inni ludzie regularnie mnie proszą albo pytają.

Po wywiadzie wypisz 8 kandydatów na agenta. Dla każdego jedno zdanie:
kto by z niego korzystał i co by za tę osobę robił.
Nie oceniaj ich jeszcze i niczego nie twórz w plikach.
```

**Gotowe, gdy:** masz listę co najmniej 5 kandydatów, z których każdy dotyczy prawdziwej osoby i powtarzalnego zadania.

### Krok 2 — Sprawdź, czy to w ogóle zadanie dla agenta

**Po co:** agent jest najdroższym i najmniej przewidywalnym sposobem rozwiązania problemu. Jeśli wystarczy coś prostszego, wybierz prostsze — będzie tańsze, szybsze i łatwiejsze do naprawienia.

**Co robisz:** dla każdego kandydata ustal, na którym szczeblu tej drabiny da się go rozwiązać. Zawsze wybieraj **najniższy**, który wystarcza.

| Szczebel | Kiedy wystarcza | Przykład |
|---|---|---|
| **1. Zwykły program** | kroki są zawsze te same i nie trzeba rozumieć tekstu | cotygodniowy raport sprzedaży z bazy |
| **2. Jedno pytanie do modelu** | trzeba zrozumieć albo napisać tekst, ale w jednym kroku | streszczenie dokumentu; ustalenie, czy mail to reklamacja |
| **3. Workflow** | kroków jest kilka, ale zawsze te same i w tej samej kolejności | wyciągnij dane z faktury → sprawdź je → zapisz |
| **4. Agent** | liczba i kolejność kroków **zależy od tego, co wyjdzie po drodze** | Ofertownik: raz jedno wyszukanie, raz pięć i pytania do klienta |

Prosty test: spróbuj zapisać kroki jako listę ponumerowaną. Jeśli lista wychodzi taka sama dla każdego przypadku — to szczebel 3 albo niżej. Jeśli co chwilę piszesz „to zależy, co znajdzie" — to szczebel 4.

Pomysł, który ląduje na szczeblach 1–3, nie jest zły. Jest po prostu zadaniem na inne narzędzie. Na tę ścieżkę wybierz kandydata ze szczebla 4 albo poszerz pomysł tak, żeby naprawdę wymagał decyzji po drodze.

**Prompt do asystenta:**

```text
Mam listę kandydatów na agenta AI: [wklej listę].

Dla każdego oceń, na którym szczeblu da się go rozwiązać:
1 = zwykły program, 2 = jedno pytanie do modelu, 3 = workflow o stałych krokach,
4 = agent, który sam decyduje o krokach.

Bądź sceptyczny: jeśli wystarczy niższy szczebel, powiedz to wprost.
Dla każdego kandydata podaj szczebel i jedno zdanie uzasadnienia —
w szczególności: która decyzja zależy od tego, co wyjdzie po drodze?
Na końcu wskaż dwóch kandydatów, którzy najbardziej potrzebują agenta.
```

**Gotowe, gdy:** masz dwóch kandydatów, dla których umiesz powiedzieć jednym zdaniem, **która decyzja zależy od tego, co agent znajdzie po drodze**.

<details>
<summary>🔬 Głębiej: gdzie naprawdę przebiega granica między workflow a agentem</summary>

Podział „workflow albo agent" jest uproszczeniem. W działających systemach prawie zawsze spotkasz hybrydę: sztywny szkielet w kodzie (walidacja wejścia → pętla agenta → walidacja wyniku → zapis) i autonomię modelu tylko w środku.

Praktyczne pytanie brzmi więc nie „czy agent", tylko „**ile autonomii i w którym miejscu**". Każdy fragment, który da się zapisać jako kod, warto zapisać jako kod: jest darmowy, deterministyczny i testowalny zwykłym testem jednostkowym. Modelowi zostaw wyłącznie decyzje, których nie umiesz zapisać regułą.

Warto też pamiętać, że granica przesuwa się z czasem. Coś, co rok temu wymagało pętli z pięcioma krokami, dziś bywa jednym wywołaniem mocniejszego modelu z dobrym kontekstem. Jeśli w Module 4 okaże się, że pojedyncze wywołanie przechodzi Twoje ewaluacje — to wygrana, nie porażka projektu.

Sygnał ostrzegawczy przy wyborze tematu: jeśli jedynym uzasadnieniem agenta jest „bo użytkownik może zapytać o cokolwiek", to nie masz zadania, tylko czat. Agent potrzebuje **celu, który da się ocenić**.

</details>

### Krok 3 — Sprawdź, czy da się to zrobić

**Po co:** wiele dobrych pomysłów rozbija się nie o technologię, tylko o dostęp do danych, prawo albo rozmiar. Lepiej dowiedzieć się tego dziś niż za trzy tygodnie.

**Co robisz:** dla dwóch najlepszych kandydatów odpowiedz uczciwie na pięć pytań.

| Pytanie | 🟢 dobrze | 🔴 problem |
|---|---|---|
| Czy mam dostęp do danych i systemów, których agent potrzebuje? | tak, już dziś | „jakoś się załatwi" |
| Czy wolno mi tych danych użyć i wysłać je do modelu? | dane własne albo publiczne | dane klientów, pracodawcy albo innych osób bez ich zgody |
| Czy da się sprawdzić, że wynik jest dobry? | tak, mam kryterium | „będzie widać" |
| Czy błąd agenta da się cofnąć? | tak, albo człowiek zatwierdza przed wykonaniem | agent sam wysyła, płaci albo kasuje |
| Czy najmniejsza wersja zmieści się w 3–4 tygodniach pracy wieczorami? | tak | wymaga połączenia z pięcioma systemami |

Jedno 🔴 nie przekreśla pomysłu. Musisz je jednak nazwać w briefie i napisać, co z nim robisz. W Ofertowniku czerwone było drugie pytanie (dane klientów w archiwum ofert) — rozwiązaniem jest usunięcie nazw i nazwisk przed wgraniem.

> 🛑 **Dane z pracy.** Jeśli chcesz użyć danych pracodawcy albo klientów, najpierw zapytaj, czy wolno. Do nauki w zupełności wystarczą dane zmyślone albo zanonimizowane — a ewaluacje i tak będziesz pisać na przykładach.

**Prompt do asystenta:**

```text
Rozważam dwa pomysły na agenta AI:
A: [opisz w 2 zdaniach]
B: [opisz w 2 zdaniach]

Zadaj mi po kolei pięć pytań o każdy z nich (po jednym naraz):
1. Skąd agent weźmie dane i czy mam do nich dostęp już dziś?
2. Czy wolno mi tych danych użyć i wysłać je do zewnętrznego modelu?
3. Po czym poznam, że wynik agenta jest dobry?
4. Co się stanie, gdy agent się pomyli — czy da się to cofnąć?
5. Jaka jest najmniejsza wersja, która już komuś pomaga?

Po moich odpowiedziach powiedz, który pomysł jest bardziej wykonalny
w 3–4 tygodnie pracy po godzinach i jakie jest jego największe ryzyko.
Nie bądź uprzejmy kosztem prawdy.
```

**Gotowe, gdy:** masz jeden wybrany pomysł i listę jego czerwonych pól z pomysłem, co z każdym zrobisz.

### Krok 4 — Zapisz sukces jako liczbę

**Po co:** „agent pomaga w ofertach" nie da się sprawdzić. Bez liczby nigdy nie będziesz wiedzieć, czy zmiana coś poprawiła, i nigdy nie skończysz projektu.

**Co robisz:** ułóż 1–3 miary, które da się policzyć na zestawie przykładów.

| ❌ Nie da się sprawdzić | ✅ Da się sprawdzić |
|---|---|
| „agent przygotowuje dobre oferty" | „na 20 zapytań co najmniej 16 szkiców nadaje się do wysłania po poprawkach krótszych niż 2 minuty" |
| „agent nie zmyśla" | „0 szkiców z ceną, której nie ma w cenniku" |
| „agent jest tani i szybki" | „koszt szkicu najwyżej 0,05 USD, czas najwyżej 60 sekund" |

Dobra miara ma trzy elementy: **na ilu przykładach**, **co liczymy** i **jaki wynik uznajemy za dobry**. Nie musisz teraz mieć tych przykładów — zbierzesz je w [Module 1](M1_SPECYFIKACJA.md), a w [Module 4](M4_EWALUACJE.md) zamienisz na automatyczne testy.

**Gotowe, gdy:** masz co najmniej jedną miarę zawierającą liczbę przykładów i próg.

<details>
<summary>🔬 Głębiej: jak nie oszukać się własną miarą</summary>

Miara, którą sam wybierasz, sam mierzysz i sam optymalizujesz, bardzo łatwo przestaje mierzyć to, o co chodziło. Kilka typowych pułapek:

- **Miara zastępcza zamiast celu.** „Szkic zawiera cenę z cennika" da się zaliczyć szkicem bezużytecznym. Dlatego główna miara Ofertownika to czas poprawek człowieka, a zgodność ceny jest tylko warunkiem koniecznym.
- **Średnia ukrywa ogon.** „Średnio 85% trafności" może znaczyć 100% na łatwych przypadkach i 30% na tych, o które naprawdę chodzi. Od początku myśl kategoriami: typowe, brzegowe, negatywne.
- **Brak miary kosztu błędu.** Dwa błędy nie są sobie równe: literówka w powitaniu i cena niższa o 30% to różne światy. Warto mieć osobną miarę typu „zero" dla błędów niedopuszczalnych.
- **Próg ustalony po fakcie.** Jeśli najpierw mierzysz, a potem decydujesz, ile wystarczy — zawsze wystarczy tyle, ile wyszło. Próg zapisz teraz.

Dobrą praktyką jest dopisanie do briefu jednej **miary przeciwwagi**: liczby, która ma się nie pogorszyć, gdy poprawiasz główną. Dla Ofertownika: odsetek szkiców, w których agent poprosił o dopytanie, mimo że dane były kompletne. Bez niej łatwo „poprawić" trafność, ucząc agenta odmawiać.

</details>

### Krok 5 — Napisz brief

**Po co:** żeby wszystko, co ustaliłeś, było w jednym miejscu — dla Ciebie za miesiąc i dla asystenta, który będzie ten projekt pisał.

**Co robisz:**

1. Utwórz **nowe, puste repozytorium** na swój projekt (na GitHubie albo lokalnie przez `git init`). Nie pracuj w folderze projektu z kursu głównego.
2. Utwórz w nim folder `docs` i plik `docs/BRIEF.md`.
3. Wypełnij go według wzoru Ofertownika powyżej. Sekcje są zawsze te same:

| Sekcja | Co wpisać | Najczęstszy błąd |
|---|---|---|
| **Użytkownik** | jedna konkretna osoba albo jeden typ osoby, w konkretnej sytuacji | „firmy", „wszyscy, którzy…" |
| **Problem** | jak to wygląda dziś, ile trwa, co w tym boli | opis rozwiązania zamiast problemu |
| **Co robi agent** | trzy zdania: co dostaje → co się dzieje → co oddaje | lista dziesięciu funkcji |
| **Dlaczego agent** | która decyzja zależy od tego, co wyjdzie po drodze | „bo AI" |
| **Czego agent NIE robi** | co najmniej 3 punkty | pominięcie tej sekcji |
| **Miara sukcesu** | 1–3 liczby z kroku 4 | przymiotniki zamiast liczb |
| **Dane i dostępy** | skąd dane, czy wolno ich użyć, jakie konta są potrzebne | „dane się znajdą" |
| **Ryzyka** | co może pójść źle i co wtedy | brak ryzyk |
| **Najmniejsza przydatna wersja** | jeden scenariusz, który sam w sobie komuś pomaga | „wersja 1.0 ze wszystkim" |

Sekcja **„Czego agent NIE robi"** jest ważniejsza, niż wygląda. To Twoja obrona przed rozrostem projektu: za każdym razem, gdy przyjdzie Ci do głowy „a mógłby jeszcze…", sprawdzasz tę listę.

**Prompt do asystenta:**

```text
Pomóż mi napisać brief projektu agenta AI. Pracujemy w tym repozytorium.

Mój pomysł: [opisz w 3–4 zdaniach: dla kogo, jaki problem, co agent ma robić].

Zadawaj mi po JEDNYM pytaniu naraz, żeby uzupełnić sekcje:
Użytkownik, Problem, Co robi agent, Dlaczego agent a nie skrypt lub jeden prompt,
Czego agent NIE robi (minimum 3 punkty), Miara sukcesu (z liczbami),
Dane i dostępy, Ryzyka, Najmniejsza przydatna wersja.

Zasady:
- Gdy odpowiem ogólnikiem („firmy", „dobre odpowiedzi", „szybko"), dopytaj o konkret.
- Niczego nie wymyślaj za mnie. Jeśli czegoś nie wiem, wpisz „DO USTALENIA: …".
- Po wywiadzie zapisz wynik w pliku docs/BRIEF.md. Ma się zmieścić na jednej stronie.
- Nie twórz żadnych innych plików i nie pisz kodu.
```

**Gotowe, gdy:** w repozytorium jest `docs/BRIEF.md`, w którym nie ma ani jednego „DO USTALENIA", a miara sukcesu zawiera liczbę.

## Jeśli nie masz własnego tematu

Możesz wziąć jeden z poniższych i dopasować go do siebie. To punkty startu — brief i tak piszesz sam, ze swoim użytkownikiem i swoimi liczbami.

| Temat | Co robi agent | Co w nim trudne |
|---|---|---|
| **Szkice ofert** (jak Ofertownik, w Twojej branży) | z zapytania klienta przygotowuje szkic oferty na podstawie cennika i dawnych ofert | zero zmyślonych cen; dopytanie zamiast zgadywania |
| **Odpowiedzi z własnych dokumentów** | odpowiada na pytania wyłącznie na podstawie wgranych materiałów i wskazuje źródło każdego zdania | odmowa, gdy w dokumentach nie ma odpowiedzi |
| **Porządkowanie zgłoszeń** | czyta nowe zgłoszenie, szuka podobnych, proponuje kategorię, priorytet i pierwszą odpowiedź | odróżnienie duplikatu od podobnej sprawy; agent proponuje, człowiek zatwierdza |
| **Recenzent według zasad** | dostaje tekst albo zmianę w kodzie i spisane zasady zespołu, zwraca uwagi z odwołaniem do konkretnej zasady | mało fałszywych alarmów; konkrety zamiast ogólników |

## Najczęstsze potknięcia

- **„Asystent do wszystkiego."** Objaw: w sekcji „Co robi agent" jest siedem punktów. → Zostaw jeden. Resztę przenieś do „Czego NIE robi" z dopiskiem „na razie".
- **Użytkownik bez twarzy.** Objaw: „małe firmy", „studenci". → Wpisz jedną osobę, najlepiej taką, do której możesz zadzwonić.
- **Sukces bez liczby.** Objaw: „agent ma odpowiadać poprawnie". → Wróć do kroku 4 i dopisz: na ilu przykładach, co liczysz, jaki próg.
- **Za duża najmniejsza wersja.** Objaw: pierwsza wersja wymaga skrzynki pocztowej, kalendarza i wysyłki. → Zacznij od „wklejam tekst, dostaję szkic". Integracje przyjdą później ([T1](T1_INTEGRACJE.md)).
- **Rozwiązanie w poszukiwaniu problemu.** Objaw: wiesz, jakiej technologii chcesz użyć, ale nie umiesz opisać, kogo boli brak tego agenta. → Wróć do kroku 1.

## ✅ Sprawdź, czy masz wszystko

- [ ] Nowe repozytorium na projekt, oddzielne od projektu z kursu głównego.
- [ ] `docs/BRIEF.md` z wypełnionymi wszystkimi dziewięcioma sekcjami.
- [ ] Użytkownik to konkretna osoba albo jeden typ osoby.
- [ ] „Czego agent NIE robi" ma co najmniej 3 punkty.
- [ ] Miara sukcesu zawiera liczbę przykładów i próg.
- [ ] Najmniejsza przydatna wersja to jeden scenariusz.

## 🤖 Poproś asystenta o recenzję

```text
Przeczytaj docs/BRIEF.md i oceń go krytycznie, jak doświadczony kierownik projektu,
który widział wiele nieudanych projektów AI. Dla każdego punktu napisz PASS albo FAIL
i jedno zdanie uzasadnienia:

1. Czy użytkownik i problem są konkretne?
2. Czy to zadanie naprawdę wymaga agenta, czy wystarczy zwykły program,
   jedno pytanie do modelu albo workflow o stałych krokach?
3. Czy miarę sukcesu da się policzyć na zestawie przykładów?
4. Czy najmniejsza przydatna wersja jest realna na 3–4 tygodnie pracy po godzinach?
5. Jakie największe ryzyko pominąłem?

Niczego nie zmieniaj w plikach. Na końcu podaj jedną rzecz,
którą poprawiłbyś w pierwszej kolejności.
```

## 🚀 Dla chętnych

- 🔵 **Porozmawiaj z użytkownikiem.** Jeśli agent nie jest dla Ciebie, pokaż brief osobie, dla której ma być. Zapisz, co ją zdziwiło i czego jej zabrakło.
- 🔵 **Zrób to zadanie raz ręcznie.** Wykonaj je samodzielnie, notując każdy krok i każde miejsce, do którego zajrzałeś. To najlepszy materiał do następnego modułu.
- 🟣 **Napisz anty-brief.** Opisz w pół strony, jak ten projekt może się nie udać **mimo działającego kodu**: nikt nie używa, koszt przerasta wartość, wynik wymaga tyle poprawek co praca od zera. Do każdego scenariusza dopisz najtańszy eksperyment, który wykryłby go w pierwszym tygodniu.
- 🟣 **Policz opłacalność.** Ile kosztuje to zadanie dziś (czas × stawka × liczba powtórzeń w miesiącu)? Ile może kosztować agent, żeby miało to sens, wliczając czas człowieka na sprawdzanie? Przy jakiej trafności agent zaczyna *spowalniać* pracę zamiast ją przyspieszać?

## Co dalej

Masz kartkę z ustaleniami. W następnym module rozpisujesz ją na konkrety: [Moduł 1: Specyfikacja](M1_SPECYFIKACJA.md).
