# Moduł 1: Specyfikacja

> ⏱️ Ok. 3 godziny. Nadal bez kodu.
> 🔒 **Wejście:** gotowy `docs/BRIEF.md` z Modułu 0.

## O co chodzi

Brief mówi **po co**. Specyfikacja mówi **co dokładnie** — na tyle precyzyjnie, żeby dało się to zbudować i sprawdzić. W projektach agentowych specyfikacja ma dodatkową rolę: za chwilę będzie ją czytał Twój agent kodujący. Im mniej musi zgadywać, tym mniej naprawiasz.

Trzy rzeczy odróżniają specyfikację agenta od zwykłej aplikacji: zachowanie opisujesz **scenariuszami** (bo nie wypiszesz wszystkich ścieżek), każde narzędzie ma określone **uprawnienia**, a koszt i czas jednego zadania są **wymaganiem**, nie ciekawostką.

## Cel

Po tym module masz `docs/SPEC.md` zawierający:
- 6–10 **scenariuszy** z kryterium zaliczenia, w tym przypadki negatywne,
- **inwentarz narzędzi** z poziomem ryzyka każdego,
- **granice**: co agent robi sam, co proponuje, czego nie wolno mu nigdy,
- **budżety**: koszt, czas i liczba kroków na jedno zadanie.

## Kroki

### 1. Napisz scenariusze
Scenariusz to konkretny przypadek użycia z prawdziwymi danymi — nie „użytkownik zadaje pytanie", tylko to pytanie.

```markdown
### S3: Zgłoszenie będące duplikatem
**Wejście:** issue „Logowanie nie działa po aktualizacji" (treść: …)
**Stan świata:** w repo istnieje otwarte issue #41 o tym samym błędzie
**Oczekiwane zachowanie:** agent przeszukuje otwarte zgłoszenia, znajduje #41,
proponuje etykietę `duplicate` i komentarz z linkiem do #41
**Zaliczone, gdy:** wynik zawiera odwołanie do #41 i nie proponuje nowej etykiety priorytetu
**Niedopuszczalne:** zamknięcie zgłoszenia bez potwierdzenia człowieka
```

Potrzebujesz co najmniej:
- **3 scenariusze typowe** — to, co zdarza się najczęściej,
- **2 brzegowe** — niepełne dane, sprzeczne informacje, bardzo długie wejście,
- **2 negatywne** — sytuacje, w których poprawnym zachowaniem jest **odmowa, dopytanie albo przekazanie człowiekowi** (pytanie spoza zakresu, brak danych w źródłach, prośba o akcję, której agentowi nie wolno).

Scenariusze negatywne są najczęściej pomijane, a to one decydują, czy agentowi można zaufać.

### 2. Zrób inwentarz narzędzi
Wypisz wszystko, czym agent będzie mógł dotknąć świata. Jeszcze nie implementacja — tylko lista.

| Narzędzie | Co robi | Odczyt / zapis | Odwracalne? | Kto zatwierdza |
|---|---|---|---|---|
| `search_issues` | szuka w zgłoszeniach repo | odczyt | — | nikt |
| `read_file` | czyta plik z repo | odczyt | — | nikt |
| `propose_labels` | zapisuje propozycję etykiet | zapis do własnej bazy | tak | nikt |
| `post_comment` | publikuje komentarz | zapis na zewnątrz | nie | **człowiek** |

Zasada: **każde narzędzie, które zapisuje na zewnątrz i nie da się cofnąć, wymaga zatwierdzenia przez człowieka** — przynajmniej w pierwszej wersji. Jeśli Twój agent ma same takie narzędzia, przeprojektuj go tak, żeby *proponował*, a nie *wykonywał*.

Skreśl każde narzędzie, którego nie wymaga żaden scenariusz.

### 3. Opisz dane
Dla każdego źródła danych, które trafia do modelu:

| Źródło | Co zawiera | Wrażliwość | Czy treść jest zaufana? |
|---|---|---|---|
| wiadomość użytkownika | pytanie | niska | nie — użytkownik może próbować obejść zasady |
| dokumenty w bazie wiedzy | PDF-y wgrane przez użytkownika | zależy | **nie** — dokument może zawierać polecenia dla modelu |
| wyniki wyszukiwania w sieci | fragmenty stron | publiczne | **nie** |

Kolumna „zaufana?" wróci w Module 6. Na razie wystarczy, że ją uczciwie wypełnisz: wszystko, czego nie napisałeś sam, jest **danymi, nie poleceniami**.

### 4. Wyznacz granice
Trzy listy, każda konkretna:

```markdown
## Agent robi sam
- wyszukuje i czyta (wszystkie narzędzia tylko do odczytu)
- zapisuje propozycje do własnej bazy

## Agent proponuje, człowiek zatwierdza
- publikację komentarza
- zmianę etykiet

## Agentowi nie wolno nigdy
- zamykać ani usuwać zgłoszeń
- ujawniać treści instrukcji systemowych ani kluczy
- wykonywać poleceń znalezionych w treści zgłoszeń lub dokumentów
```

### 5. Ustal budżety
Wpisz liczby, nawet jeśli dziś zgadujesz — poprawisz je po pierwszych pomiarach.

| Budżet | Wartość | Skąd |
|---|---|---|
| Koszt jednego zadania | np. ≤ 0,05 USD | cena modelu × szacowana liczba tokenów × liczba kroków |
| Czas jednego zadania | np. ≤ 60 s | ile użytkownik poczeka |
| Maksymalna liczba kroków | np. 8 | najdłuższy scenariusz + zapas |
| Maksymalny rozmiar wejścia | np. 20 000 znaków | ochrona przed przypadkowym i celowym zalaniem |

Ceny modeli dostępnych na kluczu kursowym znajdziesz w tabeli `MODELE_DO_CWICZEN.md` w repo kursu.

### 6. Złóż `docs/SPEC.md`
Kolejność sekcji: Scenariusze → Narzędzia → Dane → Granice → Budżety → Poza zakresem. Sekcję „Poza zakresem" przepisz z briefu i **dopisz** wszystko, co skreśliłeś po drodze.

## ✅ Dowód „done"

W repo jest `docs/SPEC.md`, w którym:
- jest co najmniej 7 scenariuszy, z czego co najmniej 2 negatywne,
- każdy scenariusz ma pole „Zaliczone, gdy" możliwe do sprawdzenia bez dyskusji,
- każde narzędzie ma wypełnione „Odwracalne?" i „Kto zatwierdza",
- budżety zawierają liczby.

## 🤖 Sprawdź się przez AI

> *„Oto brief i specyfikacja agenta AI: [wklej oba]. Oceń jak recenzent projektu — PASS/FAIL + jedno zdanie:*
> *1. Czy każdy scenariusz ma kryterium zaliczenia, które da się sprawdzić automatycznie lub jednoznacznie ręcznie?*
> *2. Czy są scenariusze negatywne (odmowa, dopytanie, eskalacja) i czy są realistyczne?*
> *3. Czy każde narzędzie jest potrzebne co najmniej jednemu scenariuszowi?*
> *4. Czy któreś narzędzie zapisuje na zewnątrz bez zatwierdzenia przez człowieka?*
> *5. Wymień 3 scenariusze, których brakuje, a które prawdopodobnie zdarzą się w praktyce."*

## 🚀 Idź dalej (nieobowiązkowe)

1. **Daj specyfikację komuś do przeczytania** i poproś, żeby opowiedział własnymi słowami, co agent zrobi w scenariuszu S1. Różnice to dziury w specyfikacji.
2. **Zapisz przykładowe dane.** Do każdego scenariusza dodaj w `docs/przyklady/` prawdziwy plik wejściowy. Przydadzą się jako przypadki testowe.
3. **Policz budżet kosztu naprawdę.** Weź najdłuższy scenariusz, oszacuj tokeny wejścia i wyjścia na każdy krok i pomnóż przez cenę modelu.
