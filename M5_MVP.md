# Moduł 5: Budowa MVP

> ⏱️ Najdłuższy moduł — 1–2 tygodnie pracy po godzinach.
> 🔒 **Wejście:** ewaluacje z punktem odniesienia i progiem gotowości z Modułu 4.

## O co chodzi

Masz plan, szkielet i miernik. Teraz budujesz — kamień po kamieniu, aż ewaluacje przekroczą próg, który sam ustaliłeś. Ten moduł nie uczy nowej techniki. Uczy dyscypliny: **nie dodawać niczego, czego nie wymaga scenariusz, i nie wierzyć w nic, czego nie zmierzyłeś**.

## Cel

Po tym module:
- wszystkie scenariusze ze specyfikacji działają,
- ewaluacje przekraczają **próg gotowości** z Modułu 4,
- akcje wymagające zatwierdzenia **czekają na człowieka**,
- masz dziennik decyzji pokazujący, co zmieniałeś i z jakim skutkiem.

## Kroki

### 1. Idź kamieniami milowymi
Dla każdego kamienia z `docs/PLAN.md`:

1. gałąź,
2. plan od agenta kodującego → Twoja korekta,
3. małe zadania, każde z przeczytanym diffem,
4. ewaluacje dla scenariuszy tego kamienia (pojedyncze przypadki w trakcie, całość na koniec),
5. wpis do dziennika, scalenie.

Jeśli kamień trwa dłużej niż trzy sesje pracy — podziel go.

Kamienie dotyczące zewnętrznych usług realizuj z modułem [T1: Integracje](T1_INTEGRACJE.md), a zadania uruchamiane bez udziału użytkownika — z [T2: Praca w tle](T2_PRACA_W_TLE.md).

### 2. Narzędzia: odporność przed funkcjami
Każde narzędzie, zanim uznasz je za gotowe:

| Wymaganie | Dlaczego |
|---|---|
| **limit czasu** na wywołanie zewnętrzne | jedno zawieszone API blokuje całe zadanie |
| **błąd jako dane** z informacją, co model może zrobić dalej | model potrafi zareagować na `{"error": "not_found"}`; na wyjątek — nie |
| **walidacja wejścia** schematem | model czasem poda zły typ albo zmyślony identyfikator |
| **przycięty wynik** | patrz tabela kontekstu z Modułu 2 |
| **test jednostkowy** bez udziału modelu | narzędzie to zwykła funkcja — testuj ją jak zwykłą funkcję |

Narzędzia testuj **bez modelu**. Wtedy, gdy ewaluacja nie przechodzi, wiesz, że problem leży w decyzjach agenta, a nie w funkcji.

### 3. Instrukcje systemowe: krótko i z powodem
Instrukcje pisz jak wdrożenie dla nowego, bystrego współpracownika:
- **rola i cel** w dwóch zdaniach,
- **jak pracować**: kiedy szukać, kiedy dopytać, kiedy odmówić,
- **granice** ze specyfikacji — z uzasadnieniem; model lepiej stosuje regułę, którą rozumie,
- **format wyniku**.

Trzymaj je w osobnym pliku w repo, nie w środku kodu. Każda zmiana instrukcji to commit i przebieg ewaluacji.

Gdy przypadek nie przechodzi, **nie dopisuj kolejnego „WAŻNE: zawsze…"**. Najpierw przeczytaj przebieg i ustal przyczynę:

| Co widzisz w przebiegu | Gdzie poprawić |
|---|---|
| agent nie użył narzędzia, choć powinien | opis narzędzia („kiedy użyć") |
| użył z błędnymi argumentami | schemat wejścia, przykłady w opisie |
| dostał dobre dane i źle wywnioskował | instrukcje albo mocniejszy model |
| zgubił się po wielu krokach | za dużo w kontekście — przytnij wyniki narzędzi |
| wynik w złym formacie | walidacja schematem w kodzie + ponowienie, nie prośba w prompcie |

### 4. Wymuś strukturę wyniku w kodzie
Jeśli wynik agenta trafia dalej do programu (baza, API, interfejs) — wymagaj **ustrukturyzowanej odpowiedzi** według schematu i waliduj ją. Niepoprawną odrzuć i poproś model o poprawę, z limitem prób. Parsowanie wolnego tekstu wyrażeniami regularnymi to dług, który spłacisz w najgorszym momencie.

### 5. Zbuduj zatwierdzanie
Dla każdego narzędzia oznaczonego w specyfikacji „człowiek zatwierdza":
- agent **nie wykonuje** akcji, tylko zapisuje **propozycję** (co, z jakimi parametrami, dlaczego),
- człowiek widzi propozycję z uzasadnieniem i źródłami, akceptuje lub odrzuca,
- dopiero akceptacja uruchamia właściwą akcję — **w zwykłym kodzie, poza modelem**,
- odrzucenie z komentarzem zapisujesz; to gotowe przypadki do ewaluacji.

Interfejs może być surowy. Liczy się to, że model nie ma technicznej możliwości wykonania akcji samodzielnie.

### 6. Prowadź dziennik decyzji
`docs/DZIENNIK.md`, jeden wpis na zmianę wartą zapamiętania:

```markdown
## <data> — opis narzędzia search_issues
**Problem:** S1 przechodzi w 2/4; agent wyszukuje całym zdaniem i nie znajduje duplikatów.
**Zmiana:** w opisie narzędzia: „query to 2–4 słowa kluczowe, nie zdanie; spróbuj dwóch wariantów".
**Wynik:** S1 4/4, ogółem 71% → 79%, średnio kroków 3,1 → 3,6, koszt bez istotnej zmiany.
```

Zapisuj też zmiany, które **nie pomogły** — to one oszczędzają najwięcej czasu przy następnym problemie.

### 7. Wiedz, kiedy przestać
MVP jest gotowe, gdy:
- ewaluacje ≥ próg z Modułu 4,
- scenariusze negatywne dotyczące zakazanych akcji przechodzą w 100%,
- koszt i czas zadania mieszczą się w budżecie ze specyfikacji.

Wszystko, co przychodzi Ci do głowy ponad to, trafia do `docs/POTEM.md`, nie do kodu.

## ✅ Dowód „done"

- Przebieg ewaluacji na aktualnym commicie przekracza próg; wynik zapisany w `docs/EWALUACJE.md` obok punktu odniesienia.
- Każde narzędzie ma test jednostkowy.
- Akcja wymagająca zatwierdzenia nie wykonuje się bez decyzji człowieka — pokaż to na zrzucie ekranu lub w logu.
- `docs/DZIENNIK.md` ma co najmniej 5 wpisów, w tym jeden o zmianie, która nie pomogła.

## 🤖 Sprawdź się przez AI

> *„Oto specyfikacja, instrukcje systemowe, kod narzędzi i pętli agenta oraz ostatni wynik ewaluacji: [wklej]. Oceń jak recenzent przed wdrożeniem — PASS/FAIL + jedno zdanie:*
> *1. Czy każde narzędzie ma limit czasu, walidację wejścia i zwraca błędy jako dane?*
> *2. Czy istnieje ścieżka, którą model mógłby wykonać akcję wymagającą zatwierdzenia bez udziału człowieka?*
> *3. Czy pętla ma limit kroków i co się dzieje po jego przekroczeniu?*
> *4. Czy w instrukcjach systemowych są reguły, które powinny być egzekwowane kodem?*
> *5. Które scenariusze ze specyfikacji nie mają pokrycia w ewaluacjach?*
> *Nie odczytuj i nie wypisuj plików .env."*

## 🚀 Idź dalej (nieobowiązkowe)

1. **Daj agenta komuś innemu.** Jedna osoba, 15 minut, bez Twoich podpowiedzi. Wszystko, co ją zaskoczyło, zapisz jako przypadki.
2. **Obniż koszt bez utraty skuteczności.** Spróbuj przenieść proste kroki na najtańszy model i sprawdź ewaluacjami, czy wynik się utrzymał.
3. **Wydziel podagenta — jeśli pomiar to uzasadnia.** Jeżeli jeden typ zadania zalewa kontekst głównego agenta, przenieś go do osobnego wywołania, które zwraca tylko wniosek. Jak to zrobić i zmierzyć, opisuje [T4: Multi-agent](T4_MULTI_AGENT.md).
