# Moduł 4: Ewaluacje

> ⏱️ Ok. 4 godziny.
> 🔒 **Wejście:** działający walking skeleton z Modułu 3.

## O co chodzi

W zwykłym kodzie zmieniasz funkcję i test mówi, czy coś zepsułeś. W agencie zmieniasz jedno zdanie w instrukcji i **nie wiesz**, czy jest lepiej — bo to samo wejście może dać inny wynik, a „lepiej" dla jednego scenariusza bywa „gorzej" dla trzech innych. Bez pomiaru rozwój agenta to zgadywanie na podstawie dwóch ręcznych prób.

Ewaluacje (evals) to zestaw przypadków z kryterium oceny, który uruchamiasz po każdej istotnej zmianie. Budujesz je **teraz**, przy szkielecie — nie na końcu — bo od tej chwili to one mówią Ci, czy idziesz do przodu.

## Cel

Po tym module:
- masz `evals/` z **15–30 przypadkami** wyprowadzonymi ze scenariuszy,
- każdy przypadek ma **automatyczne kryterium oceny**,
- jedna komenda uruchamia całość i wypisuje **odsetek zaliczonych, koszt i liczbę kroków**,
- masz zapisany **punkt odniesienia** (baseline) dla szkieletu.

## Kroki

### 1. Zamień scenariusze na przypadki
Każdy scenariusz ze specyfikacji rozwiń w 2–4 przypadki: ten sam typ sytuacji, inne dane. Zapisz je w jednym pliku, po jednym na linię:

```jsonl
{"id": "s1-duplikat-oczywisty", "scenario": "S1", "input": {"title": "Logowanie nie działa po aktualizacji", "body": "..."}, "expect": {"must_reference": ["#41"], "label": "duplicate"}}
{"id": "s1-duplikat-inne-slowa", "scenario": "S1", "input": {"title": "Nie mogę wejść na konto od wczoraj", "body": "..."}, "expect": {"must_reference": ["#41"], "label": "duplicate"}}
{"id": "s6-poza-zakresem", "scenario": "S6", "input": {"title": "Jaka jest pogoda?", "body": ""}, "expect": {"refusal": true, "tools_called": []}}
```

Zadbaj o proporcje: przypadki negatywne i brzegowe mają stanowić **co najmniej jedną trzecią**. Zestaw złożony z samych łatwych przypadków pokaże 100% i niczego Cię nie nauczy.

Skąd brać dane: z przykładów zebranych w Module 1, z prawdziwych danych (zanonimizowanych, jeśli trzeba), a w ostateczności wygenerowane modelem **i przeczytane przez Ciebie** — przypadek, którego sam nie umiesz ocenić, jest bezużyteczny.

### 2. Dobierz sposób oceny
Są trzy, w kolejności od najlepszego:

| Sposób | Co sprawdza | Kiedy |
|---|---|---|
| **Sprawdzenie kodem** | wynik pasuje do schematu, zawiera wymagany identyfikator, wywołano właściwe narzędzie, nie wywołano zakazanego, liczba kroków w limicie | zawsze, gdy się da — tanie, szybkie, powtarzalne |
| **Porównanie z wzorcem** | wynik równy lub bliski oczekiwanemu (kategoria, liczba, lista) | gdy poprawna odpowiedź jest jedna |
| **Model jako sędzia** | jakość tekstu według rubryki: czy odpowiada na pytanie, czy każde twierdzenie ma źródło | tylko gdy dwóch pierwszych nie da się zastosować |

Większość tego, co chcesz wiedzieć o agencie, da się sprawdzić kodem — pod warunkiem, że w Module 3 logujesz przebieg (jakie narzędzia, w jakiej kolejności, z jakim wejściem). **Oceniaj nie tylko odpowiedź końcową, ale i drogę**: agent, który doszedł do dobrego wyniku bez przeszukania zgłoszeń, miał szczęście.

Jeśli używasz sędziego:
- rubryka ma być **zamknięta** — pytania tak/nie, nie „oceń w skali 1–10",
- sędzia dostaje **wejście, wynik i kryterium**, nic więcej,
- **sprawdź sędziego ręcznie** na 10 przypadkach, zanim mu zaufasz; jeśli nie zgadzasz się z nim w więcej niż jednym — popraw rubrykę,
- każde wywołanie sędziego to dodatkowy koszt — użyj taniego modelu.

### 3. Napisz runner
Skrypt, który dla każdego przypadku: uruchamia agenta, zbiera wynik i przebieg, stosuje kryteria, zapisuje rezultat. Wynik końcowy to tabela:

```
id                        wynik   kroki   koszt USD   powód
s1-duplikat-oczywisty     PASS    3       0.0021
s1-duplikat-inne-slowa    FAIL    2       0.0014      brak odwołania do #41
s6-poza-zakresem          PASS    1       0.0004
...
RAZEM: 17/24 (71%)   koszt: 0.041 USD   średnio kroków: 3.1
```

Wymagania:
- **jedna komenda** (`npm run eval`, `python -m evals` — jak wolisz),
- możliwość uruchomienia **jednego przypadku** po `id` — tak będziesz debugować,
- wynik zapisywany do pliku z datą i identyfikatorem commita, żeby dało się porównać przebiegi,
- koszt i tokeny brane z odpowiedzi API (OpenRouter zwraca zużycie i koszt w każdej odpowiedzi).

> 🛑 **Ewaluacje kosztują.** 24 przypadki × kilka kroków × każda zmiana w instrukcji — to szybko rośnie, a klucz kursowy ma wspólny limit. Uruchamiaj całość świadomie, w trakcie pracy używaj pojedynczych przypadków, sprawdzaj kodem zamiast sędzią, a do przebiegów roboczych bierz najtańszy model z tabeli kursowej.

### 4. Zmierz punkt odniesienia
Uruchom całość na szkielecie z Modułu 3. Wynik będzie słaby — i dobrze. Zapisz go w `docs/EWALUACJE.md`:

```markdown
## Punkt odniesienia — <data>, commit <id>
- Zaliczone: 9/24 (38%)
- Koszt całego przebiegu: 0,03 USD; średnio na przypadek: 0,0013 USD
- Średnia liczba kroków: 2,1

## Co nie przechodzi i dlaczego
- S3 (brzegowe, 0/4): narzędzie read_file jeszcze zaślepione
- S6 (negatywne, 1/4): agent próbuje odpowiadać na pytania spoza zakresu

## Próg, przy którym uznaję MVP za gotowe
- ≥ 85% ogółem, 100% w scenariuszach negatywnych dotyczących akcji zakazanych
- koszt na zadanie ≤ budżet ze specyfikacji
```

Próg ustal **teraz**, przed budową. Ustalany po fakcie zawsze wychodzi tyle, ile akurat masz.

### 5. Uwzględnij losowość
Uruchom ten sam przypadek pięć razy. Jeśli wynik się zmienia — to informacja, nie awaria runnera. Przypadki niestabilne oznacz i zdecyduj: albo kryterium jest zbyt ostre, albo zachowanie agenta jest naprawdę niepewne i trzeba je ustabilizować (precyzyjniejszy opis narzędzia, walidacja wyniku w kodzie).

### 6. Przyjmij jedną zasadę na resztę projektu
**Każdy błąd znaleziony ręcznie staje się nowym przypadkiem, zanim go naprawisz.** Zestaw rośnie razem z projektem i pilnuje, żeby naprawione rzeczy nie psuły się po cichu.

## ✅ Dowód „done"

- `evals/` zawiera co najmniej 15 przypadków, z czego co najmniej 1/3 negatywnych lub brzegowych.
- Jedna komenda uruchamia całość i wypisuje wynik, koszt i kroki.
- `docs/EWALUACJE.md` zawiera punkt odniesienia i **próg gotowości MVP**.

## 🤖 Sprawdź się przez AI

> *„Oto przypadki ewaluacyjne mojego agenta, kod runnera i wynik pierwszego przebiegu: [wklej]. Specyfikacja: [wklej]. Oceń — PASS/FAIL + jedno zdanie:*
> *1. Czy przypadki pokrywają wszystkie scenariusze ze specyfikacji, w tym negatywne?*
> *2. Czy kryteria sprawdzają także przebieg (wywołane narzędzia), a nie tylko tekst odpowiedzi?*
> *3. Czy gdzieś użyłem sędziego-modelu tam, gdzie wystarczyłoby sprawdzenie kodem?*
> *4. Czy któryś przypadek zaliczy się nawet wtedy, gdy agent zachowa się źle?*
> *5. Jakich 3 przypadków brakuje?"*

## 🚀 Idź dalej (nieobowiązkowe)

1. **Porównaj dwa modele.** Uruchom zestaw na najtańszym i na droższym modelu z tabeli. Zapisz różnicę w skuteczności i w koszcie — to najlepszy argument przy wyborze modelu.
2. **Zmierz sędziego.** Oceń 20 wyników ręcznie i policz, jak często sędzia się z Tobą zgadza.
3. **Dodaj przypadki z historii rozmów.** Jeśli agent jest konwersacyjny — przypadek to cała rozmowa, a kryterium dotyczy ostatniej odpowiedzi.
