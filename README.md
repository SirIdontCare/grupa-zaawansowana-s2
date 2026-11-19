# Grupa Zaawansowana — zaprojektuj i zbuduj własnego agenta

Nieobowiązkowa ścieżka projektowa dla osób, które programują i chcą przejść pełny cykl budowy agenta AI: od wyboru problemu, przez specyfikację, architekturę, ewaluacje i zabezpieczenia, do wdrożenia i pokazu. Na kursie głównym budujesz agenta według lekcji. Tutaj budujesz **swojego** — na własny temat, według własnych decyzji, z liczbami na dowód, że działa.

> Autor: **Paweł Paruzel** · Uzupełnienie kursu [AI Agents](https://github.com/SirIdontCare/laba-agenci-ai-s2)

## Zacznij tutaj

1. Przeczytaj tę stronę do końca — 5 minut.
2. Otwórz **[Moduł 0: Problem i brief](M0_BRIEF.md)** i wybierz temat. Na tym etapie nie potrzebujesz niczego poza edytorem tekstu.
3. Załóż **nowe, puste repozytorium** na swój projekt. Tego repo nie klonujesz — to instrukcje, które czytasz w przeglądarce; cała praca powstaje u Ciebie.

Ścieżka główna to około **30 godzin** pracy plus 1–2 tygodnie budowy MVP. Nie ma terminów — idziesz własnym tempem.

## Dla kogo

Dla programistów — od juniora po seniora. Zakładam, że swobodnie piszesz kod w co najmniej jednym języku, używasz gita i pracujesz z agentem kodującym (Codex albo Claude Code). Nie zakładam doświadczenia w budowaniu systemów agentowych.

Jeśli dopiero uczysz się programować, zostań przy kursie głównym — ta ścieżka nie tłumaczy podstaw.

## Co powstanie

Jeden projekt, w jednym repozytorium, rozwijany przez całą ścieżkę:

- **działający, wdrożony agent** rozwiązujący wybrany przez Ciebie problem,
- **dokumentacja decyzji**: brief, specyfikacja, architektura, plan,
- **zestaw ewaluacji** z wynikiem przed i po,
- **model zagrożeń** i bezpieczniki kosztowe,
- **ślady przebiegów** z produkcji i runbook,
- **pięciominutowe demo** i retrospektywa.

## Mapa ścieżki

| # | Moduł | Co powstaje | Czas |
|---|---|---|---|
| **0** | [Problem i brief](M0_BRIEF.md) | `docs/BRIEF.md` | ~2 h |
| **1** | [Specyfikacja](M1_SPECYFIKACJA.md) | `docs/SPEC.md`: scenariusze, narzędzia, granice, budżety | ~3 h |
| **2** | [Architektura](M2_ARCHITEKTURA.md) | `docs/ARCHITEKTURA.md`: kształt, kontrakty narzędzi, kontekst, ADR-y | ~3 h |
| **3** | [Plan pracy i pierwszy przekrój](M3_PLAN_PRACY.md) | plik instrukcji dla agenta kodującego, `docs/PLAN.md`, działający szkielet | ~4 h |
| **4** | [Ewaluacje](M4_EWALUACJE.md) | `evals/`, punkt odniesienia, próg gotowości | ~4 h |
| **5** | [Budowa MVP](M5_MVP.md) | agent przechodzący własny próg, dziennik decyzji | 1–2 tyg. |
| **6** | [Bezpieczeństwo i koszty](M6_BEZPIECZENSTWO_I_KOSZTY.md) | przypadki ataku, limity w kodzie, `docs/BEZPIECZENSTWO.md` | ~4 h |
| **7** | [Wdrożenie i obserwowalność](M7_WDROZENIE.md) | adres produkcyjny, ślady przebiegów, `docs/RUNBOOK.md` | ~4 h |
| **8** | [Demo i retrospektywa](M8_DEMO.md) | README projektu, demo, `docs/RETRO.md` | ~3 h |

Moduły idą **po kolei** — każdy korzysta z artefaktów poprzedniego. Pierwsze trzy nie wymagają pisania kodu i to nie jest przypadek: zmiana decyzji w dokumencie kosztuje minuty, w kodzie — dni.

**Czego się spodziewać na końcu:** działającego, wdrożonego agenta dla kilku użytkowników — nie gotowego produktu komercyjnego. Płatności pojawiają się w module T5 wyłącznie w środowisku testowym; obsługa wielu firm-klientów, sprzedaż i strona prawna są poza zakresem.

### Moduły techniczne — do wyboru

Pięć umiejętności, których kurs główny nie ćwiczy — w Lekcji 12 pojawiają się jako kierunki dalszego rozwoju. Bierzesz te, których wymaga Twój projekt; żaden nie jest warunkiem ukończenia ścieżki.

| Moduł | Weź, jeśli agent… | Kiedy | Czas |
|---|---|---|---|
| **T1** [Integracje — OAuth i MCP](T1_INTEGRACJE.md) | działa na cudzych systemach: poczta, kalendarz, repozytoria, CRM | po Module 2, przed lub w trakcie Modułu 5 | ~5 h |
| **T2** [Praca w tle](T2_PRACA_W_TLE.md) | działa bez otwartej przeglądarki: o stałej porze, na zdarzenie, nad długim zadaniem | po Module 5, przed Modułem 7 | ~5 h |
| **T3** [Własny serwer MCP](T3_WLASNY_SERWER_MCP.md) | ma udostępniać swoje narzędzia innym agentom, np. Twojemu agentowi kodującemu | po Module 5 | ~5 h |
| **T4** [Multi-agent](T4_MULTI_AGENT.md) | nie mieści zadania w jednym kontekście albo ma części do zrobienia równolegle — i pokazuje to pomiar | po Module 5, najlepiej po Module 7 | ~6 h |
| **T5** [Płatności — Stripe](T5_PLATNOSCI_STRIPE.md) | ma być płatny: subskrypcja, webhook, limity zależne od planu (w środowisku testowym) | po Module 7 | ~6 h |

### Dodatki (nieobowiązkowe)

| Dodatek | Kiedy się przyda |
|---|---|
| [Python 1: backend agenta w FastAPI](dodatki/PYTHON_1_FASTAPI.md) | rozważasz stack inny niż kursowy |
| [Python 2: Pydantic AI](dodatki/PYTHON_2_PYDANTIC_AI.md) | chcesz zobaczyć wymuszanie struktury wyniku w Pythonie |
| [Python 3: LangGraph](dodatki/PYTHON_3_LANGGRAPH.md) | chcesz zobaczyć pętlę agenta jako jawny graf |
| [Koszty agenta](dodatki/KOSZTY_AGENTA.md) | audytujesz koszty agenta na stacku kursowym |

## Zasady ścieżki

1. **Własny temat.** Wybierasz problem sam. Jeśli nie masz pomysłu, w [Module 0](M0_BRIEF.md) są trzy tematy startowe.
2. **Dowolny stack.** Możesz zostać przy stacku kursowym (Next.js, Vercel AI SDK, OpenRouter, Supabase) i zacząć od [startera](https://github.com/SirIdontCare/starter-agent-s2), albo użyć własnego. Decyzję zapisujesz w Module 2.
3. **Osobne repozytorium i osobna baza.** Projekt z tej ścieżki nie dotyka Twojego projektu z kursu głównego.
4. **Najpierw pomiar, potem opinia.** „Działa lepiej" bez wyniku ewaluacji nie jest argumentem.
5. **Ty odpowiadasz za kod.** Agent kodujący pisze, Ty czytasz każdy diff. Tego, czego nie rozumiesz, nie scalasz.

## Jak pracować z modułem

Każdy moduł ma tę samą budowę i da się go przejść, wklejając gotowe prompty do asystenta (Codex albo Claude Code):

| Sekcja | Co w niej jest |
|---|---|
| **O co chodzi** | po co jest ten etap — prostym językiem, z porównaniem z życia |
| **Słowa, które tu padną** | każde nowe pojęcie wyjaśnione jednym zdaniem |
| **Przykład: Ofertownik** | jak wynik etapu wygląda na konkretnym projekcie — wypełniony, nie pusty szablon |
| **Kroki** | każdy krok: po co, co robisz, **gotowy prompt do asystenta** i warunek „gotowe, gdy" |
| **🔬 Głębiej** | zwijane bloki dla doświadczonych: kompromisy, pułapki z produkcji, rachunki |
| **Najczęstsze potknięcia** | po czym poznać, że coś poszło źle, i co wtedy |
| **✅ Sprawdź, czy masz wszystko** | lista kontrolna |
| **🤖 Poproś asystenta o recenzję** | prompt z rubryką — dostajesz PASS/FAIL z uzasadnieniem |
| **🚀 Dla chętnych** | 🔵 rozszerzenia dla każdego i 🟣 wyzwania dla doświadczonych |

### Dwa sposoby czytania

- **Budujesz pierwszy własny projekt po kursie?** Czytaj tekst główny i wklejaj prompty po kolei. Bloki „Głębiej" możesz pominąć — moduł jest kompletny bez nich.
- **Programujesz od lat?** Przeleć kroki, a zatrzymaj się na blokach „🔬 Głębiej" i wyzwaniach 🟣. Tam są rzeczy, o które warto się pospierać.

### Przykład, który prowadzi przez całą ścieżkę

We wszystkich modułach wraca ten sam projekt: **Ofertownik** — agent, który dla małej firmy szkoleniowej przygotowuje szkice ofert na podstawie cennika i wcześniejszych ofert, a właścicielka je zatwierdza. Nie budujesz go. Służy do pokazania, jak każdy etap wygląda w praktyce; swój projekt robisz obok, na własny temat.

Recenzja asystenta to pierwszy filtr, nie wyrocznia — ostateczna ocena należy do Ciebie.

## Bez prac domowych — z pokazem na koniec

Ta ścieżka **nie ma prac domowych, terminów ani oddawania modułów**. Pracujesz we własnym tempie, a o tym, czy etap jest gotowy, decydujesz sam — po liście kontrolnej i recenzji asystenta.

Jeśli zbudujesz swojego agenta, możesz go **zaprezentować grupie na 12. zajęciach** kursu. Pokaz jest dobrowolny; jak go przygotować, opisuje [Moduł 8](M8_DEMO.md).

## Klucz i koszty

Modele bierzesz z OpenRoutera **kluczem od LABA** — tym samym co na kursie głównym. Kilka konsekwencji:

- klucz jest wspólny i ma limit — ewaluacje i testy uruchamiaj świadomie ([Moduł 4](M4_EWALUACJE.md)),
- agent korzystający z tego klucza musi być **za logowaniem** i mieć limity ([Moduł 6](M6_BEZPIECZENSTWO_I_KOSZTY.md)),
- klucz nigdy nie trafia do repo, logów ani zrzutów ekranu; wyciek zgłaszasz prowadzącemu od razu,
- do pracy roboczej używaj najtańszego modelu z kursowej tabeli [`MODELE_DO_CWICZEN.md`](https://github.com/SirIdontCare/laba-agenci-ai-s2/blob/main/MODELE_DO_CWICZEN.md),
- agent, za którego ktokolwiek płaci ([T5](T5_PLATNOSCI_STRIPE.md)), nie może działać na kluczu kursowym — potrzebuje Twojego własnego.

## Gdy utkniesz

1. **Przeczytaj przebieg.** Co agent faktycznie zrobił — jakie narzędzia, z czym, co dostał. Większość odpowiedzi jest w logu.
2. **Zawęź.** Jeden przypadek, jedno narzędzie, jedna zmiana.
3. **Jeśli pytasz o pomoc — pytaj z konkretem:** co chciałeś osiągnąć, co się dzieje, identyfikator przypadku lub fragment przebiegu, co już sprawdziłeś.

## Czego potrzebujesz

Lista narzędzi i linków: **[NARZEDZIA.md](NARZEDZIA.md)**

---

© Paweł Paruzel. Materiały autorskie.
