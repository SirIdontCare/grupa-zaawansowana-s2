# Moduł 8: Demo i retrospektywa

> ⏱️ Ok. 3 godziny przygotowania + 5 minut pokazu.
> 🔒 **Wejście:** wdrożony agent ze śladami z Modułu 7.

## O co chodzi

Projekt, którego nie umiesz pokazać w pięć minut, dla większości odbiorców nie istnieje. A projekt, z którego nie wyciągnąłeś wniosków, nauczył Cię mniej, niż mógł.

Ten moduł zamyka ścieżkę dwiema rzeczami: **pokazem** opartym na liczbach, nie na wrażeniu, oraz **retrospektywą**, w której uczciwie porównujesz to, co zaplanowałeś w Modułach 0–2, z tym, co powstało.

## Cel

Po tym module:
- `README.md` projektu pozwala obcej osobie zrozumieć i uruchomić agenta,
- masz **pięciominutowe demo** z jednym przypadkiem udanym i jednym trudnym,
- masz `docs/RETRO.md` — co się sprawdziło, co nie i co zrobiłbyś inaczej.

## Kroki

### 1. Napisz README projektu
To wizytówka. Kolejność ma znaczenie — najważniejsze na górze:

```markdown
# <nazwa>
Jedno zdanie: dla kogo i co robi.

## Jak to wygląda
Zrzut ekranu albo krótki zapis jednego przebiegu.

## Wyniki
| | Punkt odniesienia | Teraz |
|---|---|---|
| Ewaluacje ogółem | 38% | 88% |
| Scenariusze negatywne i ataki | 25% | 100% |
| Koszt na zadanie | 0,0013 USD | 0,0034 USD |
| Czas zadania (mediana) | 4 s | 9 s |

## Jak działa
Diagram z docs/ARCHITEKTURA.md + 3–5 zdań.

## Czego nie robi i znane ograniczenia
Uczciwie. Z docs/BEZPIECZENSTWO.md i docs/POTEM.md.

## Uruchomienie
Kroki od `git clone` do działającego agenta. Lista zmiennych środowiskowych — nazwy, bez wartości.

## Dokumentacja
Linki do docs/.
```

Sprawdź instrukcję uruchomienia na czysto: nowy folder, `git clone`, kroki z README — dokładnie tak, jak napisane.

### 2. Ułóż demo
Pięć minut, w tej kolejności:

| Czas | Co | Wskazówka |
|---|---|---|
| 0:00–0:40 | **Problem i użytkownik** | jedno zdanie z briefu, bez technologii |
| 0:40–2:10 | **Scenariusz typowy na żywo** | prawdziwe dane; pokaż ślad przebiegu, nie tylko odpowiedź |
| 2:10–3:20 | **Scenariusz trudny** | odmowa, dopytanie, zatwierdzenie przez człowieka albo zablokowany atak |
| 3:20–4:10 | **Liczby** | tabela z README: punkt odniesienia → teraz, koszt na zadanie |
| 4:10–5:00 | **Jedna decyzja i jedna lekcja** | co wybrałeś i co byś dziś zmienił |

Zasady:
- **Scenariusz trudny jest ważniejszy niż typowy.** To, że agent odpowiada na proste pytanie, nikogo nie zaskoczy. To, że wie, kiedy nie odpowiadać, buduje zaufanie.
- **Miej nagranie zapasowe.** Model na żywo potrafi odpowiedzieć inaczej niż na próbie. Nagraj udany przebieg i miej go pod ręką.
- **Nie pokazuj kodu**, chyba że ktoś zapyta. Pokaż zachowanie i ślad.
- **Nie pokazuj sekretów.** Zamknij panele ze zmiennymi środowiskowymi i pliki `.env` przed udostępnieniem ekranu.

### 3. Napisz retrospektywę
`docs/RETRO.md`. Otwórz obok brief, specyfikację i architekturę z pierwszych modułów i porównaj z rzeczywistością:

```markdown
## Plan a rzeczywistość
| Założenie (skąd) | Co wyszło |
|---|---|
| miara sukcesu: 17/20 (brief) | … |
| koszt na zadanie ≤ 0,05 USD (specyfikacja) | … |
| jeden agent, 4 narzędzia (architektura) | … |
| 6 kamieni milowych w 4 tygodnie (plan) | … |

## Trzy decyzje, które się obroniły
Co, dlaczego, po czym to poznaję.

## Trzy decyzje, które zmieniłbym
Co, dlaczego, co zrobiłbym zamiast.

## Gdzie agent kodujący pomógł najbardziej, a gdzie zaszkodził
Konkretne sytuacje z dziennika.

## Czego nie wiem do dziś
Pytania, na które ewaluacje i ślady nie odpowiedziały.

## Następny krok, gdybym miał jeszcze miesiąc
Jedna rzecz, nie lista.
```

Sekcja „Czego nie wiem do dziś" jest najcenniejsza. Wypełnij ją naprawdę.

### 4. Zamknij repozytorium jak projekt
- [ ] `README.md` zgodne ze stanem kodu,
- [ ] wszystkie dokumenty w `docs/` aktualne albo wyraźnie oznaczone jako historyczne,
- [ ] ostatni przebieg ewaluacji zapisany na aktualnym commicie,
- [ ] w historii git nie ma sekretów,
- [ ] wiadomo, co dzieje się z wdrożeniem po zakończeniu kursu: zostaje (na jakim kluczu?) czy jest wyłączane.

> 🛑 Klucz kursowy jest na czas kursu — nie zakładaj, że będzie działał po jego zakończeniu. Jeśli agent ma żyć dalej — podmień klucz na własny i ustaw na nim limit wydatków, zanim komukolwiek udostępnisz adres.

## ✅ Dowód „done"

- README zawiera tabelę wyników z punktem odniesienia i stanem obecnym.
- Instrukcja uruchomienia przeszła test na czystym folderze.
- Demo mieści się w 5 minutach i zawiera scenariusz trudny.
- `docs/RETRO.md` ma wypełnione wszystkie sekcje.

## 🤖 Sprawdź się przez AI

> *„Oto README mojego projektu, scenariusz demo i retrospektywa: [wklej]. Oceń jak osoba, która widzi ten projekt pierwszy raz i ma zdecydować, czy mu zaufać — PASS/FAIL + jedno zdanie:*
> *1. Czy po pierwszym akapicie README wiem, dla kogo to jest i co robi?*
> *2. Czy wyniki są podane liczbami z punktem odniesienia?*
> *3. Czy ograniczenia są opisane uczciwie i konkretnie?*
> *4. Czy retrospektywa zawiera prawdziwe błędy decyzyjne, a nie tylko „zabrakło czasu"?*
> *5. Jakie pytanie zadałbyś autorowi po tym demo?"*

## Pokaz na 12. zajęciach

Jeśli zbudowałeś swojego agenta, możesz go zaprezentować grupie na **12. zajęciach** kursu. Pokaz jest dobrowolny — to okazja, nie obowiązek. Przygotuj demo według kroku 2 i daj znać prowadzącemu przed zajęciami, że chcesz wystąpić.

## Gdzie jesteś

Przeszedłeś pełny cykl: od problemu, przez specyfikację, architekturę, plan, pomiar, budowę i zabezpieczenia, do wdrożenia i wniosków. Kod tego agenta zestarzeje się szybko — modele i biblioteki zmieniają się co kilka miesięcy. Kolejność pytań, które zadałeś po drodze, zostaje: **dla kogo, co dokładnie, jak najprościej, po czym poznam, co może pójść źle, skąd będę wiedział**.
