# 🐍 Dodatek — Python 1: backend agenta w FastAPI

> ⭐ **Dodatek nieobowiązkowy.** Pierwszy z trzech przykładów w Pythonie — przydatny, jeśli w [Module 2](../M2_ARCHITEKTURA.md) rozważasz własny stack zamiast kursowego. Jeśli FastAPI znasz, przejrzyj tylko kod wywołania modelu i idź dalej.

## O co chodzi

Agent z kursu głównego to **jedna aplikacja Next.js** — interfejs i „mózg" w jednym projekcie, w jednym języku.

Teraz rozdzielasz go tak, jak działają prawdziwe systemy produkcyjne: **interfejs zostaje w Next.js, a mózg agenta przenosisz do osobnego serwisu w Pythonie (FastAPI)**. Dwa oddzielne procesy, dwa porty, rozmawiające przez HTTP.

Ten sam podział przyda Ci się, gdy agent ma żyć obok istniejącego systemu albo gdy potrzebujesz bibliotek z ekosystemu Pythona.

```
Przeglądarka  →  Next.js (frontend, :3000)  →  Python / FastAPI (mózg, :8000)  →  OpenRouter  →  model
```

## Cel

Po tym module:
- masz działający **backend w Pythonie**, który rozmawia z modelem przez OpenRouter — tym samym kluczem co na kursie,
- Twój agent w przeglądarce odpowiada **przez Pythona**, nie przez Next.js,
- rozumiesz, czym jest osobny serwis, **port**, **CORS**, **venv** i zmienne środowiskowe.

## Zanim zaczniesz

Potrzebujesz działającej aplikacji Next.js z kursu — własnego agenta po Lekcji 7 albo [startera](https://github.com/SirIdontCare/starter-agent-s2). Pracuj na **kopii**, nie na projekcie kursowym. Backend w Pythonie tworzysz w **nowym folderze obok** — nie w środku projektu Next.js.

---

## Kroki

### 1. Zainstaluj Pythona
Pobierz z [python.org/downloads](https://www.python.org/downloads) (3.11+).
> ⚠️ Na Windows przy instalacji **zaznacz „Add Python to PATH"** — bez tego terminal go nie zobaczy.

Sprawdź w terminalu VS Code:
```bash
python --version
```

### 2. Załóż folder backendu i środowisko wirtualne (venv)
```bash
mkdir agent-backend
cd agent-backend
python -m venv .venv
```
Aktywuj środowisko:
- **Windows:** `.venv\Scripts\activate`
- **Mac/Linux:** `source .venv/bin/activate`

W terminalu zobaczysz przedrostek `(.venv)` — jesteś w środowisku.

> 💡 **venv** to izolowana instalacja bibliotek tylko dla tego projektu — pythonowy odpowiednik `node_modules`.

### 3. Zainstaluj biblioteki
```bash
pip install fastapi uvicorn openai python-dotenv
```

### 4. Klucz API i ochrona sekretów
Utwórz plik **`.env`** w `agent-backend`:
```
OPENROUTER_API_KEY=TU_WKLEJ_KLUCZ_OD_LABA
OPENROUTER_MODEL=TU_WKLEJ_ID_MODELU
```
To **te same dwie zmienne**, które masz w `.env.local` swojego agenta — przepisz stamtąd obie wartości. Nazwy też zostają takie same.

> 💡 Biblioteka nazywa się `openai`, ale to nie znaczy, że korzystasz z konta OpenAI. OpenRouter mówi tym samym „językiem" co API OpenAI, więc wystarczy podać tej bibliotece **inny adres serwera** — zobaczysz to w kodzie jako `base_url`.

I plik **`.gitignore`**:
```
.venv
.env
__pycache__
```
> 🛑 **`.env` nigdy nie trafia do repo.**

### 5. Napisz backend — plik `main.py`
Utwórz `main.py` w `agent-backend`:

```python
import os
from fastapi import FastAPI
from fastapi.middleware.cors import CORSMiddleware
from pydantic import BaseModel
from dotenv import load_dotenv
from openai import OpenAI

load_dotenv()

app = FastAPI()

# Pozwól frontendowi z portu 3000 wołać ten backend (bez tego przeglądarka zablokuje)
app.add_middleware(
    CORSMiddleware,
    allow_origins=["http://localhost:3000"],
    allow_methods=["*"],
    allow_headers=["*"],
)

# Ten sam klucz i model co w kursie — tylko adres serwera wskazuje na OpenRouter
client = OpenAI(
    base_url="https://openrouter.ai/api/v1",
    api_key=os.getenv("OPENROUTER_API_KEY"),
)
MODEL = os.getenv("OPENROUTER_MODEL")

SYSTEM_PROMPT = "Jesteś pomocnym asystentem. Odpowiadaj po polsku, krótko i konkretnie."


class ChatRequest(BaseModel):
    message: str


@app.get("/")
def health():
    return {"status": "ok", "backend": "python"}


@app.post("/chat")
def chat(req: ChatRequest):
    response = client.chat.completions.create(
        model=MODEL,
        messages=[
            {"role": "system", "content": SYSTEM_PROMPT},
            {"role": "user", "content": req.message},
        ],
    )
    return {"reply": response.choices[0].message.content}
```

Przeczytaj ten plik linijka po linijce. Zwróć uwagę na cztery rzeczy: **model danych** (`ChatRequest`), **endpoint** (`@app.post("/chat")`), **CORS** oraz listę `messages` z rolami `system` i `user` — to dokładnie ta sama struktura rozmowy, którą w Next.js składało za Ciebie SDK.

> 🛑 **Ten backend nie ma hasła ani logowania.** Każdy, kto dosięgnie portu 8000, może rozmawiać z modelem na koszt klucza LABA. Na Twoim komputerze to bezpieczne — ale **nie wystawiaj go do internetu** w tej postaci.

### 6. Uruchom backend
```bash
uvicorn main:app --reload --port 8000
```

> 💡 **Windows:** jeśli w konsoli zamiast polskich znaków widzisz krzaki (`dzia�am`), to tylko kwestia kodowania terminala — odpowiedź jest poprawna. W przeglądarce wyświetli się prawidłowo.

Wejdź na **http://localhost:8000/docs** — FastAPI sam generuje interaktywną dokumentację. Rozwiń `POST /chat`, kliknij **Try it out**, wpisz wiadomość i **Execute**.

Jeśli dostajesz odpowiedź — **Twój agent myśli już w Pythonie.** 🎉

### 7. Podłącz frontend do Pythona
Teraz niech Twój agent w przeglądarce korzysta z tego backendu. W projekcie Next.js utwórz stronę **`app/python/page.tsx`**:

```tsx
"use client";
import { useState } from "react";

export default function PythonAgent() {
  const [message, setMessage] = useState("");
  const [reply, setReply] = useState("");
  const [loading, setLoading] = useState(false);

  async function send() {
    setLoading(true);
    const res = await fetch("http://localhost:8000/chat", {
      method: "POST",
      headers: { "Content-Type": "application/json" },
      body: JSON.stringify({ message }),
    });
    const data = await res.json();
    setReply(data.reply);
    setLoading(false);
  }

  return (
    <main style={{ padding: 32, maxWidth: 640 }}>
      <h1>🐍 Agent na backendzie w Pythonie</h1>
      <input
        value={message}
        onChange={(e) => setMessage(e.target.value)}
        placeholder="Zapytaj o coś..."
        style={{ width: "100%", padding: 8 }}
      />
      <button onClick={send} disabled={loading} style={{ marginTop: 8 }}>
        {loading ? "Myślę..." : "Wyślij"}
      </button>
      {reply && <p style={{ marginTop: 16, whiteSpace: "pre-wrap" }}>{reply}</p>}
    </main>
  );
}
```

### 8. Uruchom OBA serwisy naraz
Potrzebujesz **dwóch terminali** w VS Code (przycisk `+` w panelu terminala):
- Terminal 1 (folder `agent-backend`, z aktywnym venv): `uvicorn main:app --reload --port 8000`
- Terminal 2 (folder agenta Next.js): `npm run dev`

Wejdź na **http://localhost:3000/python** i zadaj pytanie. Odpowiedź przychodzi z Pythona. (Jeśli agent odeśle Cię najpierw do logowania — zaloguj się swoim kontem testowym; tak działa ochrona z Lekcji 7.)

> To jest ten moment, w którym widzisz, że aplikacja to nie jeden program, tylko **kilka serwisów, które ze sobą gadają.**

---

## ✅ Samo-weryfikacja (Twój dowód „done")

1. `http://localhost:8000/` zwraca `{"status":"ok","backend":"python"}`
2. `http://localhost:8000/docs` → `POST /chat` zwraca sensowną odpowiedź
3. `http://localhost:3000/python` → wpisujesz pytanie, dostajesz odpowiedź **z backendu w Pythonie**
4. W repo **nie ma** `.env` ani `.venv`

Szybki test z terminala:
```bash
curl -X POST http://localhost:8000/chat -H "Content-Type: application/json" -d "{\"message\":\"Czesc, kim jestes?\"}"
```

## 🤖 Sprawdź się sam przez AI

> *„Przeniosłem backend agenta do Pythona (FastAPI). Oto mój `main.py`: [wklej].*
> *Oceń wg rubryki — PASS/FAIL + jedno zdanie:*
> *1. Czy klucz API jest wczytywany ze zmiennej środowiskowej, a nie wpisany w kodzie?*
> *2. Czy CORS jest ustawiony tak, że frontend z localhost:3000 może wołać backend?*
> *3. Czy endpoint `/chat` poprawnie przyjmuje `message` i zwraca `reply`?*
> *4. Czy `.env` i `.venv` są w `.gitignore`?*
> *Nie odczytuj i nie wypisuj zawartości pliku .env."*

## Zapisz w repo

Wypchnij backend jako **osobne repo** (`agent-backend`).

## Chcesz iść jeszcze dalej? (nieobowiązkowe)

- **Dodaj narzędzie w Pythonie** — np. kalkulator albo dzielenie rachunku, tym razem po stronie backendu.
- **Podłącz bazę** — Supabase albo Neon (Postgres) i zapisuj historię rozmów po stronie Pythona.
- **Zabezpiecz, a potem wdróż backend** — np. na Railway albo Render. **Najpierw** dodaj ochronę: backend ma odrzucać żądania bez tajnego nagłówka znanego tylko Twojemu Next.js. Publiczny endpoint bez ochrony to otwarty kran z cudzym kluczem.

## Gdzie przekroczyłeś próg

Napisałeś działający serwis w **drugim języku**, rozdzieliłeś aplikację na **frontend i backend** i połączyłeś je przez HTTP. Od tej chwili możesz budować agenta w dowolnym języku, który ma klienta HTTP.

---

## ➡️ Następny dodatek

Twój backend zwraca **zwykły tekst** — ładny dla człowieka, bezużyteczny dla kodu. W kolejnym module naprawiasz to na poziomie architektury:

**[Python 2: Pydantic AI →](PYTHON_2_PYDANTIC_AI.md)** — agent, który zwraca **zwalidowany obiekt**, a nie zdanie (`pydantic` to odpowiednik `zod` w Pythonie).
