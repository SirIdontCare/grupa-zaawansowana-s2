# Moduł techniczny 4: Multi-agent — orkiestrator i podagenci

> ⏱️ Ok. 6 godzin.
> 🧩 **Moduł do wyboru.** Weź go, jeśli **pomiar** pokazuje, że jeden agent nie wystarcza: zadanie nie mieści się w jednym kontekście albo składa się z części, które da się robić równolegle. Wymaga działającego MVP z ewaluacjami ([Moduł 4](M4_EWALUACJE.md), [Moduł 5](M5_MVP.md)).

## O co chodzi

„Zespół agentów" brzmi jak naturalny następny krok po jednym agencie. W praktyce to najdroższa decyzja architektoniczna, jaką możesz podjąć: więcej wywołań modelu, więcej miejsc na błąd, trudniejsze debugowanie. Zespół, który zbudował i opisał publicznie produkcyjny system badawczy z wieloma agentami, podał, że zużywa on około **15 razy więcej tokenów** niż zwykły czat (pojedynczy agent — około 4 razy).

Mimo to są zadania, w których to się opłaca. Ten moduł uczy dwóch rzeczy: **rozpoznać, czy Twoje jest jednym z nich**, i zbudować najprostszą wersję, która przynosi korzyść — orkiestratora zlecającego wydzielone zadania podagentom.

## Cel

Po tym module:
- umiesz uzasadnić **liczbami**, czy Twój projekt potrzebuje więcej niż jednego agenta,
- znasz pięć wzorców pośrednich między „jeden agent" a „zespół agentów",
- masz działającego orkiestratora z co najmniej jednym podagentem,
- porównałeś wersję jedno- i wieloagentową na tych samych ewaluacjach: skuteczność, koszt, czas.

## Kroki

### 1. Sprawdź, czy naprawdę tego potrzebujesz
Otwórz ślady przebiegów z [Modułu 7](M7_WDROZENIE.md) (albo logi z Modułu 3) i odpowiedz:

| Objaw w śladach | Co oznacza | Czy to powód na podagentów |
|---|---|---|
| wyniki narzędzi zapełniają kontekst, a agent „zapomina" wcześniejsze ustalenia | jedno zadanie nie mieści się rozsądnie w jednym kontekście | **tak** — podagent przerabia dużo materiału i oddaje krótki wniosek |
| zadanie ma części niezależne (pięć źródeł, dziesięć dokumentów) wykonywane po kolei | czas rośnie liniowo bez potrzeby | **tak** — równoległość |
| agent myli narzędzia z dwóch różnych dziedzin | za dużo narzędzi w jednym kontekście | **może** — najpierw popraw opisy i spróbuj routingu |
| agent źle rozumuje na jednym, prostym kontekście | problem z instrukcjami albo modelem | **nie** — więcej agentów tego nie naprawi |
| kroki zależą jeden od drugiego i wymagają wspólnej wiedzy | zadanie jest sekwencyjne | **nie** — podział zniszczy wspólny kontekst |

Podagenci pomagają przy zadaniach **szerokich** (dużo niezależnego szukania i czytania). Szkodzą przy zadaniach, w których każda część musi wiedzieć, co robią pozostałe — dlatego większość zadań programistycznych źle się dzieli.

Jeśli nie masz ani jednego wiersza z „tak" — przeczytaj krok 2 i na nim zakończ. To też jest poprawny wynik tego modułu.

### 2. Najpierw spróbuj czegoś prostszego
Między jednym agentem a zespołem agentów jest pięć wzorców, w których **kod**, a nie model, steruje przepływem. Są tańsze, szybsze i łatwiejsze do testowania:

| Wzorzec | Jak działa | Kiedy |
|---|---|---|
| **Łańcuch** | stała sekwencja wywołań, wynik jednego jest wejściem następnego | etapy są zawsze te same |
| **Routing** | pierwsze wywołanie klasyfikuje wejście i wybiera ścieżkę, model albo zestaw instrukcji | kilka wyraźnie różnych typów zadań |
| **Równoległość** | niezależne wywołania uruchomione jednocześnie, wyniki łączone w kodzie | te same operacje na wielu elementach |
| **Orkiestrator i wykonawcy** | model planuje podział, wykonawcy realizują części, wyniki są scalane | podział zależy od zadania i nie da się go zapisać z góry |
| **Oceniający i poprawiający** | jedno wywołanie tworzy, drugie ocenia według kryteriów, pętla z limitem prób | istnieje jasne kryterium jakości |

Typowa sytuacja: „potrzebuję agenta-badacza, agenta-pisarza i agenta-recenzenta" okazuje się łańcuchem trzech wywołań z jedną pętlą oceny. Zapisz, który wzorzec rozwiązałby Twój problem — i przejdź dalej tylko wtedy, gdy potrzebujesz czwartego, w którym podział ustala model.

### 3. Zaprojektuj podział
Podagent to **osobne wywołanie agenta z czystym kontekstem**, uruchamiane przez orkiestratora jako narzędzie. Wartość bierze się z izolacji: podagent może przeczytać pięćdziesiąt stron, a do orkiestratora wraca pół strony wniosków.

Dla każdego podagenta zapisz:

| Pytanie | Przykład |
|---|---|
| **Jedno zadanie**, które wykonuje | „znajdź i streść zgłoszenia podobne do podanego" |
| **Narzędzia** — tylko te potrzebne do tego zadania | `search_issues`, `read_issue` |
| **Co dostaje** od orkiestratora | cel, zakres, format wyniku |
| **Co oddaje** | lista do 5 pozycji: numer, podobieństwo, jedno zdanie uzasadnienia |
| **Limit** kroków i model | 6 kroków; tańszy model niż orkiestrator |

Zasada uprawnień: **narzędzia zapisujące i wymagające zatwierdzenia zostają u orkiestratora**. Podagenci czytają i wnioskują. W wielu bibliotekach mechanizm zatwierdzania przez człowieka i tak nie działa wewnątrz podagenta — a nawet gdyby działał, akcja podjęta głęboko w drzewie wywołań jest najtrudniejsza do prześledzenia.

### 4. Napisz zlecenie tak, jak dla człowieka
Najczęstsza przyczyna porażki systemów wieloagentowych to nie model, tylko **mętne zlecenie**. Podagent nie widzi rozmowy z użytkownikiem ani ustaleń orkiestratora — wie tylko to, co dostanie w zleceniu. „Zbadaj temat X" kończy się tym, że trzech podagentów robi to samo, a czwartego brakuje.

W instrukcjach orkiestratora wymagaj, żeby każde zlecenie zawierało cztery elementy:

1. **cel** — co ma zostać ustalone i po co,
2. **granice** — czego nie robić, czym zajmują się inni,
3. **źródła i narzędzia** — gdzie szukać,
4. **format wyniku** — dokładnie to, co ma wrócić, i jak długie.

Dopisz też regułę **skali**: ile podagentów na jakie zadanie. Bez niej orkiestrator uruchomi pięciu do pytania, na które wystarcza jedno wyszukanie. Przykład: proste pytanie — bez podagentów; porównanie kilku rzeczy — 2–3 podagentów; szerokie rozpoznanie — do 5.

### 5. Zbuduj orkiestratora z jednym podagentem
W Vercel AI SDK agentem jest `ToolLoopAgent`, a podagent to zwykłe narzędzie, którego funkcja uruchamia innego agenta:

```ts
import { ToolLoopAgent, tool, isStepCount } from "ai";
import { z } from "zod";

const researcher = new ToolLoopAgent({
  model: workerModel,                       // tańszy model z konfiguracji
  instructions:
    "Wykonujesz jedno zlecenie badawcze. Na końcu ZAWSZE podaj zwięzłe podsumowanie w formacie ze zlecenia.",
  tools: { search_issues, read_issue },     // tylko odczyt
  stopWhen: isStepCount(6),
});

const delegateResearch = tool({
  description:
    "Zleca niezależne zadanie badawcze podagentowi z czystym kontekstem. Użyj dla części zadania, które wymagają przeczytania wielu źródeł. Zlecenie musi zawierać: cel, granice, źródła, format wyniku.",
  inputSchema: z.object({ task: z.string().min(40) }),
  execute: async ({ task }, { abortSignal }) => {
    const result = await researcher.generate({ prompt: task, abortSignal });
    return result.text;                     // do orkiestratora wraca tylko wniosek
  },
});

const orchestrator = new ToolLoopAgent({
  model: leadModel,
  instructions: orchestratorInstructions,   // z regułą skali i wymaganiami wobec zlecenia
  tools: { delegateResearch, propose_labels },
  stopWhen: isStepCount(8),
});
```

> ⚠️ Fragment pokazuje **kształt** rozwiązania według dokumentacji AI SDK z listopada 2026. Sprawdź https://ai-sdk.dev/docs/agents/subagents dla wersji, którą masz zainstalowaną — znajdziesz tam także strumieniowanie postępu podagenta do interfejsu i zawężanie tego, co wraca do modelu nadrzędnego.

Cztery rzeczy, których ten szkic pilnuje i których musisz pilnować w każdym stacku:

- **limit kroków na każdym poziomie** — domyślne limity bibliotek bywają wysokie (w AI SDK 20 kroków); orkiestrator × podagenci × kroki mnoży się szybko,
- **sygnał przerwania przekazany w dół** — anulowanie zadania zatrzymuje także podagentów,
- **wraca wniosek, nie cała praca** — inaczej izolacja kontekstu nic nie daje,
- **podagent kończy podsumowaniem** — wymuszone w jego instrukcjach, bo bez tego orkiestrator dostaje ostatnią myśl zamiast wyniku.

Jeśli model wywoła kilka narzędzi `delegateResearch` w jednym kroku, podagenci ruszają równolegle. Przy wielu równoległych wywołaniach dostawca może chwilowo odmówić części z nich (limity zależne od modelu, a w OpenRouterze także rezerwacja środków na trwające żądania) — obsłuż to ponowieniem po krótkiej przerwie, z limitem prób.

### 6. Rozszerz ślady
Przebieg wieloagentowy to drzewo, nie lista. Do śladów z Modułu 7 dodaj:

- `parent_run_id` — każdy przebieg podagenta wskazuje wywołanie orkiestratora, które go uruchomiło,
- **treść zlecenia** i **treść wyniku** podagenta — to tu szukasz przyczyn błędów,
- koszt i czas **sumowane w górę** drzewa.

Gdy wynik jest zły, czytaj w tej kolejności: zlecenie → wynik podagenta → jego kroki. W większości przypadków problem widać już w zleceniu.

### 7. Zmierz i zdecyduj
Uruchom **te same ewaluacje** na wersji jednoagentowej i wieloagentowej. Zapisz w `docs/MULTI_AGENT.md`:

```markdown
## Porównanie — <data>, commit <id>
| | Jeden agent | Orkiestrator + podagenci |
|---|---|---|
| Ewaluacje ogółem | 84% | 91% |
| Scenariusze szerokie (S4, S7) | 55% | 90% |
| Scenariusze proste (S1–S3) | 100% | 100% |
| Koszt na zadanie (średni / najwyższy) | 0,004 / 0,011 USD | 0,019 / 0,062 USD |
| Czas zadania (mediana) | 9 s | 14 s |

## Decyzja
Zostaje / wraca do jednego agenta / podagenci tylko dla scenariuszy typu S4, S7 (routing).
## Uzasadnienie
```

Trzy uczciwe wyniki tego porównania:
- **lepiej i warto** — zostaje,
- **lepiej tylko w części scenariuszy** — routing: proste zadania idą do jednego agenta, szerokie do orkiestratora,
- **nie lepiej albo za drogo** — wracasz do jednego agenta i masz to udokumentowane liczbami. To pełnoprawny rezultat.

Zaktualizuj też budżety z [Modułu 6](M6_BEZPIECZENSTWO_I_KOSZTY.md): limit kosztu **na całe drzewo**, nie na pojedyncze wywołanie, oraz maksymalną liczbę podagentów na zadanie egzekwowaną w kodzie narzędzia zlecającego.

> 🛑 Wersja wieloagentowa potrafi zużyć kilkukrotnie więcej tokenów na tych samych ewaluacjach. Na kluczu kursowym uruchom porównanie na podzbiorze przypadków i na najtańszych modelach, które dają sensowny wynik.

## ✅ Dowód „done"

- `docs/MULTI_AGENT.md` zawiera objaw ze śladów, który uzasadnił próbę, oraz tabelę porównania na tych samych przypadkach.
- Podagenci nie mają narzędzi zapisujących ani zatwierdzanych — wskaż `plik:linia`.
- Limit kroków istnieje na każdym poziomie, a limit liczby podagentów i kosztu drzewa jest w kodzie.
- Dla dowolnego przebiegu potrafisz pokazać drzewo: orkiestrator → zlecenia → wyniki, z kosztem zsumowanym.
- Decyzja (zostaje / routing / powrót) jest zapisana z uzasadnieniem.

## 🤖 Sprawdź się przez AI

> *„Oto instrukcje orkiestratora i podagentów, kod narzędzia zlecającego, przykładowy ślad przebiegu oraz porównanie ewaluacji wersji jedno- i wieloagentowej: [wklej]. Oceń sceptycznie — PASS/FAIL + jedno zdanie:*
> *1. Czy objaw, który miał uzasadnić podagentów, rzeczywiście wynika z ograniczeń jednego kontekstu, a nie ze słabych instrukcji lub opisów narzędzi?*
> *2. Czy ten sam efekt dałby łańcuch, routing albo równoległe wywołania sterowane kodem?*
> *3. Czy zlecenia w śladzie zawierają cel, granice, źródła i format wyniku?*
> *4. Czy podagent może wykonać akcję zapisującą albo ominąć zatwierdzenie?*
> *5. Czy różnica w skuteczności uzasadnia różnicę w koszcie i czasie?"*

## 🚀 Idź dalej (nieobowiązkowe)

1. **Oceniający i poprawiający.** Dodaj wywołanie, które ocenia wynik końcowy według rubryki i odsyła do poprawy, z limitem dwóch prób. Zmierz, ile to daje i ile kosztuje.
2. **Różne modele na różnych poziomach.** Mocniejszy model jako orkiestrator, najtańszy u podagentów — i odwrotnie. Porównaj na ewaluacjach.
3. **Podagent jako serwer MCP.** Wystaw podagenta przez [własny serwer MCP](T3_WLASNY_SERWER_MCP.md) i pozwól wywołać go agentowi kodującemu.
