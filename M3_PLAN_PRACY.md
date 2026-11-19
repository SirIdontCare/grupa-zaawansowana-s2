# Moduł 3: Plan pracy i pierwszy przekrój

> ⏱️ Ok. 4 godziny. Pierwszy działający kod.
> 🔒 **Wejście:** `docs/SPEC.md` i `docs/ARCHITEKTURA.md`.

## O co chodzi

Agent kodujący (Codex, Claude Code) napisze większość tego projektu szybciej niż Ty. To nie zmienia tego, kto za niego odpowiada. Różnica między projektem, który po trzech tygodniach działa, a takim, który jest stertą wygenerowanego kodu, leży w trzech rzeczach: **co agent wie o projekcie**, **jak duże zadania dostaje** i **czy ktoś czyta to, co oddał**.

W tym module ustawiasz te trzy rzeczy i budujesz **pierwszy pionowy przekrój**: jeden scenariusz działający od wejścia do wyjścia, brzydko, ale naprawdę.

## Cel

Po tym module:
- w repo jest **plik instrukcji** dla agenta kodującego,
- masz `docs/PLAN.md` z **kamieniami milowymi**, z których każdy da się sprawdzić,
- działa **walking skeleton**: scenariusz S1 przechodzi od początku do końca z jednym prawdziwym narzędziem.

## Kroki

### 1. Napisz plik instrukcji dla agenta kodującego
Agent kodujący zaczyna każdą sesję bez pamięci o projekcie. Plik instrukcji w katalogu głównym repo to jedyne, co wie na starcie:

| Narzędzie | Plik |
|---|---|
| Codex | `AGENTS.md` |
| Claude Code | `CLAUDE.md` |

Jeśli używasz obu — trzymaj treść w jednym i odwołaj się do niego z drugiego, żeby nie utrzymywać dwóch wersji.

Dobry plik instrukcji jest **krótki** (jedna–dwie strony) i zawiera to, czego nie da się wyczytać z kodu:

```markdown
# <nazwa projektu>

## Co to jest
Dwa zdania. Szczegóły: docs/BRIEF.md, docs/SPEC.md, docs/ARCHITEKTURA.md.

## Komendy
- uruchomienie: `npm run dev`
- testy: `npm test`
- ewaluacje: `npm run eval` (kosztują — nie uruchamiaj bez prośby)
- typy i lint: `npm run check`

## Struktura
- `app/api/agent/` — endpoint i pętla agenta
- `lib/tools/` — jedno narzędzie = jeden plik + test
- `evals/` — przypadki testowe i runner

## Zasady
- Nie odczytuj i nie wypisuj `.env*`.
- Nie dodawaj zależności bez pytania.
- Każde narzędzie zwraca błędy jako dane, nie rzuca wyjątków do pętli.
- Nazwy modeli tylko z konfiguracji, nigdy na sztywno.
- Jedno zadanie = jedna gałąź = mały diff. Nie refaktoryzuj przy okazji.

## Definicja „zrobione"
Kod się buduje, testy przechodzą, scenariusz z zadania działa uruchomiony ręcznie.
```

Pisz tam **zasady, które już złamał** — plik instrukcji rośnie z doświadczenia, nie z wyobraźni. Wszystko, co da się wymusić narzędziem (formatowanie, typy, testy), wymuszaj narzędziem, a nie zdaniem w pliku.

### 2. Potnij projekt na kamienie milowe
Kamień milowy to **pionowy przekrój**: coś, co da się uruchomić i sprawdzić, przechodzące przez wszystkie warstwy. Nie „baza danych", nie „wszystkie narzędzia" — tylko „scenariusz S2 działa".

```markdown
## M1: Walking skeleton
**Zakres:** scenariusz S1 od wejścia do wyjścia; jedno prawdziwe narzędzie (search_issues),
reszta zaślepiona; bez logowania, bez UI poza surowym formularzem.
**Sprawdzenie:** dla wejścia z docs/przyklady/s1.json wynik zawiera numer #41.
**Poza zakresem:** obsługa błędów, pozostałe narzędzia, wygląd.

## M2: Wszystkie narzędzia tylko do odczytu
**Zakres:** read_file, search_issues z paginacją; scenariusze S1–S3.
**Sprawdzenie:** 3 scenariusze przechodzą ręcznie.

## M3: Ewaluacje i punkt odniesienia   ← Moduł 4
## M4: Zatwierdzanie przez człowieka + scenariusze negatywne
## M5: Zabezpieczenia i budżety        ← Moduł 6
## M6: Wdrożenie                        ← Moduł 7
```

Każdy kamień ma **zakres**, **sprawdzenie** i **poza zakresem**. Jeśli nie umiesz napisać sprawdzenia w jednym zdaniu — kamień jest za duży.

Zapisz to jako `docs/PLAN.md`.

### 3. Ustal rytm pracy
Jedna pętla, powtarzana do końca projektu:

1. **Plan** — poproś agenta kodującego o plan zadania **bez pisania kodu** (oba narzędzia mają tryb planowania). Przeczytaj go. Tu poprawki są najtańsze.
2. **Małe zadanie** — jedno narzędzie, jeden endpoint, jeden scenariusz. Zadanie, którego diffu nie jesteś w stanie przeczytać w 10 minut, jest za duże.
3. **Przegląd diffu** — czytasz każdą zmienioną linię. Nie „wygląda dobrze", tylko: czy rozumiem, dlaczego to tak jest.
4. **Uruchomienie** — sam odpalasz scenariusz. „Testy przechodzą" w raporcie agenta to twierdzenie, nie dowód.
5. **Commit** — mały, z opisem *co i po co*. Dopiero potem następne zadanie.

Dwie rzeczy, które warto przyjąć od początku:
- **Nowe zadanie = nowa sesja.** Długa sesja niesie stary kontekst i stare błędy. Stan projektu ma być w repo (kod, plan, plik instrukcji), nie w historii rozmowy.
- **Gdy agent kręci się w kółko — zatrzymaj go.** Trzecia próba naprawienia tego samego błędu tą samą metodą nie zadziała. Cofnij zmiany, zawęź zadanie albo znajdź przyczynę sam.

### 4. Zbuduj walking skeleton
Zrealizuj **M1** powyższym rytmem. Ma być minimalny:
- wejście przyjmowane w najprostszy możliwy sposób (formularz, skrypt, `curl`),
- jedno wywołanie pętli agenta z **limitem kroków**,
- **jedno prawdziwe narzędzie**, pozostałe zwracają dane na sztywno,
- wynik wypisany na ekran,
- **log przebiegu**: każde wywołanie narzędzia z wejściem i wyjściem — do konsoli wystarczy.

Jeśli startujesz ze startera kursowego: najpierw **usuń** strony i endpointy, których Twój agent nie potrzebuje, w osobnym commicie. Mniej kodu to mniej kontekstu dla agenta kodującego i mniej miejsc na błędy.

Ten log przebiegu nie jest ozdobą. Od tej chwili każdy problem zaczynasz od przeczytania, **co agent faktycznie zrobił**, a nie od zgadywania.

### 5. Skonfrontuj plan z rzeczywistością
Po M1 wróć do specyfikacji i architektury. Co okazało się inne, niż zakładałeś? Popraw dokumenty **teraz** — dokument, który rozjechał się z kodem, szkodzi bardziej niż jego brak, bo agent kodujący mu uwierzy.

## ✅ Dowód „done"

- W repo jest `AGENTS.md` lub `CLAUDE.md` z komendami, strukturą i zasadami.
- `docs/PLAN.md` ma co najmniej 5 kamieni milowych, każdy ze sprawdzeniem.
- Historia git pokazuje **kilka małych commitów**, nie jeden duży.
- Scenariusz S1 działa: uruchom go i zapisz log przebiegu do `docs/przebiegi/m1.txt`.

## 🤖 Sprawdź się przez AI

> *„Oto plik instrukcji dla agenta kodującego, plan projektu i wynik `git log --stat` z mojego repo: [wklej]. Oceń — PASS/FAIL + jedno zdanie:*
> *1. Czy plik instrukcji zawiera komendy do uruchomienia i testów oraz zasady specyficzne dla tego projektu (nie ogólniki)?*
> *2. Czy każdy kamień milowy jest pionowym przekrojem z jednoznacznym sprawdzeniem?*
> *3. Czy commity są małe i opisują cel zmiany?*
> *4. Czy w pierwszym kamieniu jest coś, co można było odłożyć?*
> *Nie odczytuj i nie wypisuj plików .env."*

## 🚀 Idź dalej (nieobowiązkowe)

1. **Dodaj kontrolę automatyczną.** Skonfiguruj lokalnie formatowanie, sprawdzanie typów i testy tak, żeby jedna komenda uruchamiała wszystko — i wpisz ją do pliku instrukcji jako warunek „zrobione".
2. **Zrób zadanie dwa razy.** To samo małe zadanie zleć raz z trybem planowania, raz bez. Porównaj diffy.
3. **Napisz własne polecenie wielokrotnego użytku.** Jeśli łapiesz się na wklejaniu tego samego promptu (np. „zrób przegląd tego diffu według zasad projektu") — zapisz go jako plik w repo, zgodnie z mechanizmem Twojego narzędzia.
