# 🐍 Dodatek — Python 3: LangGraph (narysuj mózg swojego agenta)

> ⭐ **Dodatek nieobowiązkowy.** Wymaga środowiska z [Python 1](PYTHON_1_FASTAPI.md) i [Python 2](PYTHON_2_PYDANTIC_AI.md). Przydatny, jeśli w [Module 2](../M2_ARCHITEKTURA.md) wybierasz między pętlą schowaną w SDK a jawnym grafem.

## O co chodzi

Wróć myślami do **Lekcji 4** kursu głównego. Zbudowałeś tam agenta **ReAct**: myśl → działaj → obserwuj → myśl dalej. Ale tej pętli nigdy nie widziałeś. Była schowana w SDK pod jednym parametrem:

```js
stopWhen: isStepCount(5)
```
(w starszych wersjach biblioteki ten sam limit nazywał się `stepCountIs` albo `maxSteps` — stąd nazwa, która została w języku potocznym.)

Agent gdzieś w środku kręcił się w kółko, a Ty mogłeś tylko ustawić, ile razy. **LangGraph wyciąga tę pętlę na wierzch** i zamienia ją w graf, który sam rysujesz: węzły to kroki, krawędzie to decyzje, a Ty widzisz dokładnie, którędy przebiega rozumowanie.

Pierwszy raz **zobaczysz** to, czego używasz od miesięcy.

```
        START
          │
          ▼
      ┌───────┐   nie potrzeba narzędzia
      │ MYŚL  │────────────────────────────► END
      └───────┘
          │ potrzebuję narzędzia
          ▼
     ┌───────────┐
     │ NARZĘDZIA │
     └───────────┘
          │
          └──────► wraca do MYŚL
```

## Cel

Po tym module:
- budujesz **graf stanu** (`StateGraph`) z własnymi węzłami,
- rozumiesz **krawędź warunkową** — czyli jak agent decyduje „narzędzie czy koniec",
- widzisz w konsoli **każdy krok pętli ReAct**,
- wiesz, gdzie postawić **bezpiecznik na koszty** (odpowiednik limitu kroków z Lekcji 4).

## Kroki

### 1. Zainstaluj
```bash
pip install langgraph langchain-openrouter
```

### 2. Napisz graf — plik `graph_agent.py`

```python
import os
from dotenv import load_dotenv

load_dotenv()  # wczytuje OPENROUTER_API_KEY i OPENROUTER_MODEL z pliku .env

from typing import Annotated, TypedDict
from langchain_core.messages import HumanMessage
from langchain_core.tools import tool
from langchain_openrouter import ChatOpenRouter
from langgraph.graph import StateGraph, START, END
from langgraph.graph.message import add_messages
from langgraph.prebuilt import ToolNode, tools_condition


# 1. NARZĘDZIE — to samo co zawsze, tylko w konwencji LangChaina
@tool
def split_bill(total: float, people: int, tip_percent: float = 0) -> float:
    """Dzieli rachunek na osoby z opcjonalnym napiwkiem."""
    print(f"   [WĘZEŁ: NARZĘDZIE] total={total} people={people} tip={tip_percent}")
    if people <= 0:
        raise ValueError("people musi być > 0")
    return round((total * (1 + tip_percent / 100)) / people, 2)


# 2. STAN — to, co płynie przez graf. Tu: lista wiadomości.
class State(TypedDict):
    messages: Annotated[list, add_messages]


# Klucz biblioteka bierze sama z OPENROUTER_API_KEY; model — ten sam co w kursie
llm = ChatOpenRouter(model=os.getenv("OPENROUTER_MODEL")).bind_tools([split_bill])


# 3. WĘZEŁ „MYŚL" — zwykła funkcja: bierze stan, zwraca nowy fragment stanu
def mysl(state: State):
    print("   [WĘZEŁ: MYŚL]")
    return {"messages": [llm.invoke(state["messages"])]}


# 4. BUDOWA GRAFU
g = StateGraph(State)
g.add_node("mysl", mysl)
g.add_node("narzedzia", ToolNode([split_bill]))

g.add_edge(START, "mysl")                 # start zawsze do „myśl"
g.add_conditional_edges(                  # ↙ tu agent decyduje
    "mysl", tools_condition, {"tools": "narzedzia", END: END}
)
g.add_edge("narzedzia", "mysl")           # po narzędziu wracamy myśleć — TO JEST PĘTLA

app = g.compile()


if __name__ == "__main__":
    out = app.invoke(
        {"messages": [HumanMessage("Podziel 240 zł na 4 osoby z napiwkiem 10 procent.")]},
        {"recursion_limit": 8},           # ← bezpiecznik, odpowiednik limitu kroków
    )
    print("ODPOWIEDŹ:", out["messages"][-1].text)
    print("KROKÓW W GRAFIE:", len(out["messages"]))
```

### 3. Uruchom i **obserwuj pętlę**
```bash
python graph_agent.py
```
**Oczekiwany wynik:**
```
   [WĘZEŁ: MYŚL]
   [WĘZEŁ: NARZĘDZIE] total=240.0 people=4 tip=10.0
   [WĘZEŁ: MYŚL]
ODPOWIEDŹ: ... 66 zł na osobę.
KROKÓW W GRAFIE: 4
```

**Zatrzymaj się i popatrz na te trzy linie.** To jest pętla ReAct z Lekcji 4, którą przez cały kurs miałeś schowaną pod maską:
1. agent **pomyślał** i stwierdził, że potrzebuje narzędzia,
2. graf poszedł krawędzią warunkową do **węzła narzędzia**,
3. wynik wrócił do **„myśl"**, a agent sformułował odpowiedź.

> ⚠️ **Pułapka:** `out["messages"][-1].content` **nie zawsze jest tekstem** — przy części modeli to lista bloków. Żeby zawsze dostać zwykły napis, użyj **`.text`** (bez nawiasów), tak jak w kodzie wyżej.

### 4. Zobacz bezpiecznik w akcji
Zmień `recursion_limit` na `2` i uruchom ponownie. Graf przerwie się błędem `GraphRecursionError`, bo nie zdąży dojść do końca.

To jest dokładnie ten sam mechanizm co limit kroków z Lekcji 4 — i dokładnie to miejsce, w którym **pilnujesz kosztów**. Bez limitu zapętlony agent potrafi wykonać dziesiątki płatnych zapytań na jedno pytanie użytkownika (patrz [Koszty agenta](KOSZTY_AGENTA.md)).

## ✅ Samo-weryfikacja (Twój dowód „done")

- W konsoli widzisz sekwencję **MYŚL → NARZĘDZIE → MYŚL**,
- odpowiedź zawiera **66 zł na osobę**,
- `recursion_limit: 2` powoduje przerwanie — czyli rozumiesz, gdzie jest bezpiecznik,
- potrafisz wskazać w kodzie: gdzie jest **stan**, gdzie **węzeł**, a gdzie **krawędź warunkowa**.

## 🤖 Sprawdź się sam przez AI

> *„Robię moduł o LangGraph. Oto mój graf: [wklej graph_agent.py].*
> *Oceń wg rubryki — PASS/FAIL + jedno zdanie:*
> *1. Czy graf ma krawędź warunkową decydującą między narzędziem a końcem?*
> *2. Czy po węźle narzędzia stan wraca do węzła „myśl" (czyli pętla naprawdę istnieje)?*
> *3. Czy ustawiłem limit rekursji chroniący przed nieskończoną pętlą?*
> *4. Wytłumacz mi własnymi słowami, czym różni się `add_edge` od `add_conditional_edges`."*

## Zapisz w repo

```bash
git add graph_agent.py
git commit -m "Modul LangGraph: jawna petla ReAct jako graf"
git push
```

## 🚀 Idź dalej (nieobowiązkowe)

1. **Dorzuć drugie narzędzie** (np. kurs walut albo kalkulator VAT) i zobacz, jak agent sam wybiera, do którego węzła pójść.
2. **Dodaj własny węzeł** przed „myśl" — np. `waliduj`, który odrzuca wiadomości dłuższe niż 500 znaków. To bezpiecznik kosztowy wpięty w graf.
3. **Pamięć między rozmowami:** LangGraph ma *checkpointery* — potrafi zapisywać stan grafu do Postgresa. Podepnij bazę Postgres swojego projektu i graf zacznie pamiętać poprzednie rozmowy.
4. **Narysuj graf:** `app.get_graph().draw_ascii()` wypisze Twój graf w konsoli. Porównaj go z diagramem z początku tego modułu.

## Gdzie przekroczyłeś próg

Przeszedłeś drogę od „agent to czarna skrzynka, której ustawiam limit kroków" do „agent to graf, który sam zaprojektowałem i którego każdy krok widzę". To jest różnica między używaniem cudzej abstrakcji a **projektowaniem własnej architektury**.
