# Moduł 0: Problem i brief

> ⏱️ Ok. 2 godziny. Bez pisania kodu.

## O co chodzi

Większość projektów agentowych przegrywa, zanim powstanie pierwsza linia kodu — bo rozwiązują problem, którego nikt nie ma, albo taki, do którego agent w ogóle nie jest potrzebny. Na kursie głównym temat był dany. Tutaj wybierasz go sam i to jest najważniejsza decyzja całej ścieżki.

Efektem modułu jest jednostronicowy **brief**: dokument, który pozwala obcej osobie w dwie minuty zrozumieć, co budujesz, dla kogo i po czym poznasz, że działa.

## Cel

Po tym module:
- masz **jeden konkretny problem** z prawdziwym użytkownikiem (możesz nim być Ty),
- umiesz uzasadnić, dlaczego to zadanie dla **agenta**, a nie dla skryptu albo pojedynczego promptu,
- masz **mierzalną definicję sukcesu** i listę rzeczy, których agent **nie** robi.

## Kroki

### 1. Wypisz kandydatów
Zapisz 5–10 zadań, które Ty albo ktoś, kogo znasz, wykonuje **regularnie** i które wymagają czytania, szukania, porównywania albo decydowania. Nie oceniaj jeszcze — tylko lista. Dobre źródła: własna praca, zespół, projekt open source, w którym uczestniczysz, sprawy domowe i organizacyjne.

### 2. Odsiej test „czy to w ogóle agent?"
Dla każdego kandydata odpowiedz, na którym szczeblu drabiny da się go rozwiązać. Zawsze wybieraj **najniższy**, który wystarcza:

| Szczebel | Kiedy wystarcza | Przykład |
|---|---|---|
| **Skrypt / zapytanie** | kroki są zawsze te same i nie wymagają rozumienia tekstu | raport z bazy co poniedziałek |
| **Jedno wywołanie modelu** | trzeba zrozumieć lub napisać tekst, ale w jednym kroku | streszczenie dokumentu, klasyfikacja zgłoszenia |
| **Workflow** (stała sekwencja wywołań) | kroków jest kilka, ale ich kolejność znasz z góry | wyciągnij dane → zweryfikuj → sformatuj |
| **Agent** (model sam wybiera kroki i narzędzia) | liczba i kolejność kroków **zależy od tego, co wyjdzie po drodze** | diagnoza błędu, research z dopytywaniem źródeł |

Jeśli Twój pomysł ląduje na dwóch pierwszych szczeblach — to dobry pomysł na narzędzie, ale nie na tę ścieżkę. Wybierz inny albo poszerz go tak, żeby naprawdę wymagał decyzji po drodze.

### 3. Sprawdź wykonalność
Dla dwóch najlepszych kandydatów odpowiedz uczciwie:

| Pytanie | 🟢 | 🔴 |
|---|---|---|
| Czy mam dostęp do danych i systemów, których agent potrzebuje? | tak, dziś | „załatwi się" |
| Czy wolno mi tych danych użyć i wysłać je do modelu? | tak / dane publiczne lub własne | dane klienta, pracodawcy, osób trzecich bez zgody |
| Czy da się sprawdzić, że wynik jest dobry? | tak, mam kryterium | „będzie widać" |
| Czy błąd agenta jest odwracalny? | tak, albo człowiek zatwierdza | agent sam wysyła, płaci, kasuje |
| Czy najmniejsza przydatna wersja zmieści się w 3–4 tygodniach wieczorów? | tak | wymaga integracji z pięcioma systemami |

Jedno 🔴 nie przekreśla pomysłu — ale musisz je nazwać w briefie i powiedzieć, co z nim robisz.

### 4. Zdefiniuj sukces liczbą
„Agent pomaga w researchu" to nie jest miara. Miara to coś, co da się policzyć na zestawie przykładów:
- *„Dla 20 zgłoszeń testowych przypisuje poprawną kategorię i priorytet w co najmniej 17."*
- *„Każde twierdzenie w odpowiedzi ma cytat ze źródła; 0 cytatów zmyślonych na 15 pytań."*
- *„Koszt jednego zadania poniżej 0,05 USD, czas poniżej 60 s."*

Te liczby wrócą w Module 4 jako Twoje ewaluacje.

### 5. Napisz brief
Utwórz **nowe, puste repozytorium** na swój projekt i dodaj plik `docs/BRIEF.md`:

```markdown
# Brief: <nazwa agenta>

## Użytkownik
Kto konkretnie (rola, sytuacja). Jedna osoba albo jeden typ osoby.

## Problem
Jak to zadanie wygląda dziś, ile trwa, co w nim boli.

## Co robi agent
3 zdania. Wejście → co się dzieje → wyjście.

## Dlaczego agent, a nie skrypt lub jeden prompt
Które decyzje zależą od tego, co wyjdzie po drodze.

## Czego agent NIE robi
Minimum 3 punkty. To jest ochrona przed rozrostem zakresu.

## Miara sukcesu
1–3 liczby do sprawdzenia na zestawie przykładów.

## Dane i dostępy
Skąd dane, czy wolno ich użyć, jakie konta i klucze są potrzebne.

## Ryzyka
Co może pójść źle i co wtedy (zwłaszcza akcje nieodwracalne).

## Najmniejsza przydatna wersja
Jeden scenariusz, który sam w sobie komuś pomaga.
```

## Jeśli nie masz własnego tematu

Możesz wziąć jeden z poniższych i dopasować go do siebie. To punkty startu, nie gotowe specyfikacje — brief i tak piszesz sam.

| Temat | Sedno | Co w nim trudne |
|---|---|---|
| **Triage zgłoszeń w repozytorium** | agent czyta nowe issue, szuka duplikatów i powiązanego kodu, proponuje etykiety i pierwszą odpowiedź | odróżnienie duplikatu od podobnego zgłoszenia; agent proponuje, człowiek zatwierdza |
| **Research z cytatami nad własnym zbiorem dokumentów** | agent odpowiada na pytania wyłącznie na podstawie wgranych materiałów i wskazuje źródło każdego twierdzenia | odmowa, gdy w źródłach nie ma odpowiedzi; zero zmyślonych cytatów |
| **Recenzent zmian według zasad zespołu** | agent dostaje diff i spisane zasady projektu, zwraca listę uwag z odwołaniem do konkretnej reguły i linii | mało fałszywych alarmów; uwagi konkretne, nie ogólniki |

## ✅ Dowód „done"

W repo jest `docs/BRIEF.md`, w którym:
- każda sekcja jest wypełniona konkretem (żadnych „do ustalenia"),
- sekcja „Czego agent NIE robi" ma co najmniej 3 punkty,
- miara sukcesu zawiera **liczbę**.

## 🤖 Sprawdź się przez AI

> *„Oto brief projektu agenta AI: [wklej]. Oceń krytycznie jak doświadczony tech lead — PASS/FAIL + jedno zdanie uzasadnienia:*
> *1. Czy użytkownik i problem są konkretne, a nie ogólne?*
> *2. Czy to zadanie naprawdę wymaga agenta, czy wystarczy skrypt, jeden prompt albo stały workflow?*
> *3. Czy miara sukcesu jest policzalna na zestawie przykładów?*
> *4. Czy zakres najmniejszej przydatnej wersji jest realny na 3–4 tygodnie pracy po godzinach?*
> *5. Jakie największe ryzyko pominąłem?"*

## 🚀 Idź dalej (nieobowiązkowe)

1. **Porozmawiaj z użytkownikiem.** Jeśli agent nie jest dla Ciebie — pokaż brief osobie, dla której ma być, i zapisz, co ją zdziwiło.
2. **Zrób to raz ręcznie.** Wykonaj zadanie samodzielnie, notując każdy krok i każde źródło. To najlepszy materiał do specyfikacji.
3. **Napisz anty-brief.** Trzy zdania o tym, jak ten projekt mógłby się nie udać mimo działającego kodu.
