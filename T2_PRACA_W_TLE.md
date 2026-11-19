# Moduł techniczny 2: Praca w tle — harmonogram, kolejka, długie zadania

> ⏱️ Ok. 5 godzin.
> 🧩 **Moduł do wyboru.** Weź go, jeśli Twój agent ma działać **bez otwartej karty przeglądarki**: o stałej porze, w reakcji na zdarzenie albo nad zadaniem dłuższym niż jedno żądanie HTTP. Najlepszy moment: po [Module 5](M5_MVP.md), przed [Modułem 7](M7_WDROZENIE.md).

## O co chodzi

Agent z kursu głównego żyje dokładnie tyle, ile trwa jedno żądanie: użytkownik pisze, agent odpowiada, połączenie się zamyka. To wystarcza do czatu. Nie wystarcza, gdy agent ma co rano przejrzeć nowe zgłoszenia, przetworzyć dwieście dokumentów albo poczekać dobę na czyjąś akceptację.

Wtedy agent przestaje być funkcją wywoływaną z przeglądarki, a staje się **zadaniem**: czymś, co ma stan, może się przerwać w połowie, może zostać uruchomione dwa razy i czego nikt nie ogląda na żywo. To zwykła inżynieria systemów rozproszonych — z jedną różnicą: każde powtórzenie kroku kosztuje pieniądze i może dać inny wynik.

## Cel

Po tym module:
- wiesz, **który z czterech wzorców** pasuje do Twojego zadania,
- Twoje zadanie w tle jest **idempotentne** i **wznawialne**,
- każdy przebieg w tle ma stan widoczny dla użytkownika i limit kosztu,
- masz jedno działające zadanie w tle, opisane w `docs/ZADANIA_W_TLE.md`.

## Kroki

### 1. Rozpoznaj swój przypadek

| Przypadek | Przykład | Wzorzec |
|---|---|---|
| zadanie trwa dłużej, niż użytkownik chce patrzeć, ale mieści się w limicie funkcji | analiza długiego dokumentu, 2–4 minuty | **odpowiedz od razu, dokończ po odpowiedzi** |
| ma się wydarzyć o stałej porze | poranny przegląd, cotygodniowy raport | **harmonogram** |
| wiele niezależnych elementów albo reakcja na zdarzenie z zewnątrz | 200 dokumentów do przetworzenia, webhook o nowym zgłoszeniu | **kolejka** |
| wiele kroków, przerwy liczone w godzinach, oczekiwanie na człowieka | proces z akceptacją następnego dnia | **trwały przepływ (durable workflow)** |

Wybierz **najprostszy**, który wystarcza. Trwały przepływ dla zadania raz dziennie to nadmiar.

Zanim pójdziesz dalej, sprawdź limity platformy, na której wdrażasz — to one wyznaczają granice między wzorcami. Dla stacku kursowego (stan na listopad 2026, sprawdź aktualne wartości w dokumentacji):

| Platforma | Limit |
|---|---|
| funkcja na Vercelu, plan Hobby | do 300 s na wywołanie; przekroczenie kończy się błędem 504 |
| zadania cykliczne na Vercelu, plan Hobby | najwyżej **raz na dobę**, z dokładnością do godziny |
| Supabase Edge Function, plan darmowy | do 150 s czasu zegarowego, 2 s czasu procesora na żądanie |
| Supabase Cron | zalecane: zadanie do 10 minut, nie więcej niż 8 jednocześnie |

### 2. Zaprojektuj stan zadania
Niezależnie od wzorca, zadanie w tle ma wiersz w bazie. To jest jego jedyne źródło prawdy:

```sql
create table jobs (
  id uuid primary key default gen_random_uuid(),
  owner_id uuid not null,
  kind text not null,                 -- 'morning_review', 'ingest_document', ...
  idempotency_key text unique,        -- np. 'morning_review:<owner>:2026-11-19'
  status text not null default 'queued',   -- queued | running | waiting_approval | done | failed | cancelled
  input jsonb not null,
  result jsonb,
  error text,
  attempts int not null default 0,
  cost_usd numeric not null default 0,
  created_at timestamptz not null default now(),
  started_at timestamptz,
  finished_at timestamptz
);
```

Powiąż go ze śladami z [Modułu 7](M7_WDROZENIE.md): każdy przebieg agenta w ramach zadania zapisuje `job_id`.

### 3. Zrób zadanie idempotentnym
Załóż, że **każde zadanie w tle kiedyś uruchomi się dwa razy** — platformy wprost to zastrzegają (dostarczenie „w miarę możliwości", możliwe pominięcia i duplikaty), a przy awarii w połowie i tak ponawiasz sam. Dwa razy uruchomiony agent to podwójny koszt, a przy narzędziach zapisujących — podwójny e-mail albo komentarz.

Trzy mechanizmy, stosowane razem:

| Mechanizm | Jak |
|---|---|
| **klucz idempotencji** | unikalny klucz na zadanie (`kind` + właściciel + okres lub identyfikator zdarzenia); druga próba wstawienia kończy się konfliktem i wychodzi bez pracy |
| **zajęcie zadania** | `update jobs set status='running' where id=$1 and status='queued' returning id` — pracuje tylko ten, kto dostał wiersz |
| **akcje zewnętrzne z własnym kluczem** | przed wysłaniem czegokolwiek na zewnątrz zapisz zamiar w bazie; po wysłaniu oznacz jako wykonany; przy wznowieniu pomiń wykonane |

### 4. Zaimplementuj wybrany wzorzec

**A. Odpowiedz od razu, dokończ po odpowiedzi.**
Endpoint tworzy wiersz w `jobs`, odsyła `job_id` i kończy pracę już po wysłaniu odpowiedzi. W Next.js służy do tego `after()` z `next/server`; na innych platformach szukaj odpowiednika (`waitUntil`). Interfejs odpytuje o status zadania.
Ograniczenie: praca po odpowiedzi **nadal mieści się w limicie czasu funkcji**. To sposób na to, żeby użytkownik nie czekał z otwartym połączeniem — nie na zadania wielogodzinne.

**B. Harmonogram.**
Platforma wywołuje Twój endpoint o ustalonej porze. Na Vercelu definiujesz to w `vercel.json`:

```json
{ "crons": [{ "path": "/api/cron/morning-review", "schedule": "0 6 * * *" }] }
```

Endpoint cykliczny **musi sprawdzać sekret**, bo jest zwykłym publicznym adresem: ustaw zmienną `CRON_SECRET`, a platforma wyśle ją w nagłówku `Authorization: Bearer …`. Bez tej kontroli każdy, kto zna adres, uruchamia Twojego agenta na Twój koszt.

```ts
export async function GET(request: Request) {
  if (request.headers.get("authorization") !== `Bearer ${process.env.CRON_SECRET}`) {
    return new Response("Unauthorized", { status: 401 });
  }
  // 1. dla każdego użytkownika z włączonym przeglądem: wstaw zadanie z kluczem idempotencji
  // 2. NIE uruchamiaj tu agenta dla wszystkich po kolei — tylko zakolejkuj
  return Response.json({ ok: true });
}
```

Rzeczy, które zaskakują: zadania cykliczne uruchamiają się **tylko na wdrożeniu produkcyjnym**, nieudane wywołanie **nie jest ponawiane**, a na darmowym planie godzina jest przybliżona. Alternatywa w tym samym stacku: Supabase Cron (oparty na `pg_cron`), który potrafi uruchomić zapytanie SQL albo wywołać adres HTTP i pozwala na częstsze harmonogramy.

> ⚠️ Sposób definiowania zadań cyklicznych na Vercelu jest w trakcie zmiany — przed wdrożeniem sprawdź aktualną dokumentację platformy.

**C. Kolejka.**
Producent wkłada wiadomości, pracownik je wyjmuje i przetwarza po jednej. W stacku kursowym kolejkę daje Supabase Queues (oparte na `pgmq`), z operacjami `send`, `read`, `delete` i `archive`.

Kluczowe pojęcie to **okno niewidoczności**: wiadomość odczytana przez pracownika znika dla innych na zadany czas. Jeśli pracownik ją w tym czasie usunie — koniec. Jeśli padnie — wiadomość wraca do kolejki i dostanie ją ktoś inny. Stąd dwie zasady:

- okno musi być **dłuższe niż najdłuższe przetwarzanie** jednej wiadomości — inaczej dwóch pracowników zrobi to samo,
- usuwasz wiadomość **po** zakończeniu pracy, nie przed.

Pracownika uruchamia harmonogram (co minutę: „weź do N wiadomości i przetwórz") albo wyzwalacz. Wiadomość, która nie udała się kilka razy, przenosisz na bok (`archive`) i oznaczasz zadanie jako `failed` — nie ponawiasz w nieskończoność.

**D. Trwały przepływ.**
Biblioteki tego typu zapisują wynik każdego kroku, więc po awarii albo po wielogodzinnej przerwie wykonanie wznawia się od ostatniego ukończonego kroku, a nie od początku. Dają też wbudowane czekanie („uśpij na 7 dni") i wznawianie po sygnale z zewnątrz — czyli dokładnie to, czego potrzebuje zatwierdzanie przez człowieka rozłożone w czasie. W ekosystemie Vercela jest to Workflow SDK (dyrektywy `"use workflow"` i `"use step"`); istnieją też rozwiązania niezależne od platformy.
Dwie rzeczy do zapamiętania: **pojedynczy krok nadal podlega limitowi czasu funkcji**, a kroki są automatycznie ponawiane — więc każdy krok wywołujący model lub narzędzie zapisujące musi być idempotentny (punkt 3).

### 5. Pokrój pracę agenta na kroki
Zadanie agentowe w tle dziel tak, żeby **wywołanie modelu nigdy nie było powtarzane bez potrzeby**:

- jeden element (dokument, zgłoszenie, użytkownik) = jedno zadanie w kolejce, nie jedna wielka pętla,
- wynik każdego kosztownego kroku zapisujesz do bazy **zanim** przejdziesz dalej,
- przy wznowieniu najpierw sprawdzasz, co już jest zapisane,
- limit kroków agenta i limit czasu narzędzi z [Modułu 5](M5_MVP.md) obowiązują bez zmian.

### 6. Daj użytkownikowi widok i hamulec
Zadania, którego nie widać, nikt nie pilnuje. Minimum:

- lista zadań użytkownika ze **statusem**, czasem i kosztem,
- **powiadomienie** o zakończeniu lub błędzie (w aplikacji wystarczy),
- przycisk **Anuluj** — zadanie sprawdza status między krokami i kończy się, gdy widzi `cancelled`,
- zatwierdzanie z Modułu 5 działa asynchronicznie: zadanie przechodzi w `waiting_approval` i **kończy wywołanie**; decyzja człowieka tworzy nowe zadanie, które wykonuje akcję.

### 7. Ogranicz koszt tego, czego nikt nie ogląda
W czacie użytkownik zauważy, że coś poszło źle. W tle agent może mielić całą noc. Do bezpieczników z [Modułu 6](M6_BEZPIECZENSTWO_I_KOSZTY.md) dodaj:

| Bezpiecznik | Chroni przed |
|---|---|
| **budżet na jedno zadanie** — sumujesz koszt kroków w `jobs.cost_usd`, po przekroczeniu `failed` | jednym zadaniem, które się zapętliło |
| **limit prób** na zadanie (np. 3) | wiecznym ponawianiem tego samego błędu |
| **limit zadań na użytkownika** na dobę i jednocześnie | zalaniem kolejki |
| **dzienny budżet aplikacji i wyłącznik awaryjny** sprawdzane przez pracownika **przed** każdym zadaniem | wszystkim powyższym naraz |
| **alert**, gdy kolejka rośnie albo odsetek `failed` skacze | awarią, o której dowiesz się z rachunku |

> 🛑 Przy kluczu kursowym to nie jest formalność. Harmonogram uruchamiający agenta dla każdego użytkownika co minutę potrafi zużyć wspólny limit w jedną noc. Zaczynaj od najrzadszego harmonogramu, który ma sens, i od najtańszego modelu.

### 8. Opisz zadanie
`docs/ZADANIA_W_TLE.md`:

```markdown
## <nazwa zadania>
**Wyzwalacz:** harmonogram codziennie 6:00 / zdarzenie / żądanie użytkownika
**Wzorzec:** harmonogram + kolejka
**Klucz idempotencji:** morning_review:<owner_id>:<data>
**Kroki i co jest zapisywane po każdym:** …
**Limity:** budżet 0,10 USD na zadanie, 3 próby, 1 zadanie na użytkownika na dobę
**Co widzi użytkownik:** status, wynik, koszt, przycisk Anuluj
**Co się dzieje przy awarii w połowie:** …
```

## ✅ Dowód „done"

- Zadanie uruchamia się bez udziału przeglądarki i zostawia wiersz w `jobs` ze statusem, kosztem i powiązanym śladem.
- **Test podwójnego uruchomienia:** wywołaj wyzwalacz dwa razy z tym samym kluczem — agent pracuje raz. Pokaż to w logu.
- **Test przerwania:** przerwij zadanie w połowie i uruchom ponownie — ukończone kroki nie są powtarzane (koszt drugiego przebiegu jest niższy).
- Endpoint wyzwalany przez platformę odrzuca żądanie bez sekretu.
- Budżet na zadanie i limit prób są w kodzie — wskaż `plik:linia`.

## 🤖 Sprawdź się przez AI

> *„Oto kod zadania w tle mojego agenta (wyzwalacz, pracownik, schemat tabeli zadań) i jego opis: [wklej]. Oceń jak inżynier, który będzie przy tym systemie dyżurował — PASS/FAIL + jedno zdanie:*
> *1. Co się stanie, jeśli wyzwalacz odpali się dwa razy w tej samej sekundzie?*
> *2. Co się stanie, jeśli proces padnie po wywołaniu modelu, a przed zapisaniem wyniku?*
> *3. Czy istnieje ścieżka, na której to samo działanie zewnętrzne wykona się dwukrotnie?*
> *4. Czy endpoint wyzwalany przez platformę da się wywołać z zewnątrz bez sekretu?*
> *5. Co ograniczy koszt, jeśli zadanie zacznie się ponawiać w pętli o trzeciej w nocy?*
> *Nie odczytuj i nie wypisuj plików .env."*

## 🚀 Idź dalej (nieobowiązkowe)

1. **Reakcja na zdarzenie.** Zamiast harmonogramu przyjmij webhook z zewnętrznej usługi: zweryfikuj podpis, wstaw zadanie z identyfikatorem zdarzenia jako kluczem idempotencji, odpowiedz natychmiast.
2. **Postęp na żywo.** Zapisuj postęp zadania do bazy i pokaż go w interfejsie bez odświeżania strony.
3. **Zatwierdzanie następnego dnia.** Przenieś zatwierdzanie do trwałego przepływu: zadanie czeka na decyzję dowolnie długo i wznawia się po niej bez ponownego wykonywania wcześniejszych kroków.
