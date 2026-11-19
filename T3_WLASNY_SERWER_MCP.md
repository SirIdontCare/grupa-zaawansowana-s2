# Moduł techniczny 3: Własny serwer MCP

> ⏱️ Ok. 5 godzin.
> 🧩 **Moduł do wyboru.** Weź go, jeśli chcesz, żeby z możliwości Twojego agenta korzystały **inne agenty**: Twój asystent kodujący, asystent w edytorze, agent kolegi z zespołu. Wymaga działających narzędzi z [Modułu 5](M5_MVP.md). O korzystaniu z cudzych serwerów MCP jest [T1](T1_INTEGRACJE.md).

## O co chodzi

Do tej pory narzędzia Twojego agenta były dostępne tylko w Twojej kuchni. Kucharz (Twój agent) sięga po nie, kiedy chce, ale nikt z zewnątrz nie może z nich skorzystać. Żeby ktoś inny dostał to samo danie, musiałby wejść do kuchni, czyli otworzyć Twoją aplikację i pisać na Twoim czacie.

Serwer MCP to **okienko do wydawania zamówień**. Inne agenty podchodzą, czytają kartę (listę narzędzi z opisami), zamawiają konkretne danie i dostają je na tacy. Do kuchni nie wchodzą: nie widzą Twojego kodu, bazy ani instrukcji. Dostają tylko to, co wystawisz w okienku.

W [T1](T1_INTEGRACJE.md) stałeś po drugiej stronie: podłączałeś cudzy serwer MCP do swojego agenta. Tutaj odwracasz role. Narzędzia, które napisałeś (wyszukiwanie w Twojej bazie, odczyt Twoich danych), wystawiasz tak, żeby mógł ich użyć każdy program rozumiejący MCP, bez pisania integracji specjalnie pod Ciebie.

To zmienia sposób myślenia o projekcie. Przestajesz budować jedną aplikację z czatem, a zaczynasz budować **zdolność**, którą da się wpiąć w wiele miejsc. To także najlepszy sprawdzian jakości Twoich narzędzi: obcy model nie zna Twoich instrukcji systemowych. Ma tylko nazwę narzędzia, jego opis i listę parametrów.

## Po tym module masz

- **zdalny serwer MCP** pod adresem URL, wystawiający 2–4 narzędzia Twojego projektu,
- serwer, który **wymaga tokenu** i działa z uprawnieniami konkretnego użytkownika,
- serwer przetestowany Inspektorem i podłączony do Twojego asystenta kodującego,
- plik `docs/SERWER_MCP.md` z opisem: co wystawiasz, jak się podłączyć i czego świadomie nie wystawiasz,
- zrozumienie, czym serwer MCP różni się od zwykłego API i jakie ryzyka wnosi.

## Słowa, które tu padną

| Słowo | Co to znaczy | Porównanie z życia |
|---|---|---|
| **MCP** (Model Context Protocol) | wspólny standard, według którego agent rozmawia z zewnętrznymi narzędziami | gniazdko elektryczne: jedna wtyczka pasuje do każdego urządzenia |
| **Serwer MCP** | program, który wystawia narzędzia innym agentom | okienko do wydawania zamówień w kuchni |
| **Klient MCP** | program, który z tych narzędzi korzysta, np. Claude Code albo Codex | gość, który podchodzi do okienka |
| **Narzędzie** (tool) | czynność, którą klient może zlecić serwerowi | danie z karty: zamawiasz, dostajesz wynik |
| **Zasób** (resource) | dane do odczytu, które serwer udostępnia | ulotka wyłożona przy okienku |
| **Prompt** (w MCP) | gotowy szablon polecenia udostępniany przez serwer | zestaw dnia: gotowa kombinacja, którą wystarczy wybrać |
| **Transport** | sposób, w jaki klient i serwer przesyłają sobie wiadomości | rozmowa przez ladę albo przez telefon |
| **stdio** | transport lokalny: klient sam uruchamia serwer jako proces na Twoim komputerze | kucharz gotuje u Ciebie w domu, tylko dla Ciebie |
| **Streamable HTTP** | transport zdalny: serwer ma adres URL i obsługuje wielu użytkowników | restauracja z adresem, do której przychodzi każdy z rezerwacją |
| **Schemat wejścia** (`inputSchema`) | opis parametrów narzędzia: jakie są, jakiego typu, jakie mają ograniczenia | formularz zamówienia z rubrykami |
| **Adnotacje narzędzia** (`annotations`) | podpowiedzi dla klienta, np. „to narzędzie tylko czyta" | piktogramy w karcie dań: ostre, wegetariańskie |
| **Błąd wykonania** | narzędzie zadziałało, ale nie mogło spełnić prośby (nie znaleziono, brak uprawnień) | kelner mówi: „tego dania dziś nie ma" |
| **Błąd protokołu** | żądanie w ogóle nie miało sensu (nieznane narzędzie, zepsuty format) | zamówienie napisane nieczytelnie, nikt nie wie, o co chodzi |
| **Token osobisty** | długi losowy ciąg znaków, który użytkownik generuje w Twojej aplikacji i podaje klientowi zamiast hasła | karta wstępu z Twoim nazwiskiem |
| **Skrót tokenu** (hash) | wynik jednokierunkowego przekształcenia tokenu; da się sprawdzić, czy token pasuje, ale nie da się go odtworzyć | odcisk palca: rozpoznasz po nim osobę, ale nie zrobisz z niego palca |
| **Token passthrough** | antywzorzec: serwer bierze token, który dostał, i używa go dalej w innych usługach | portier, który Twoją kartą wstępu otwiera sobie inne budynki |
| **MCP Inspector** | oficjalny program do ręcznego testowania serwerów MCP | próbna wizyta inspektora przed otwarciem lokalu |

## Przykład: Ofertownik

Przypomnienie: Ofertownik to agent firmy szkoleniowej „Kurs na Start". Czyta zapytanie klienta, szuka w cenniku i w dawnych ofertach, przygotowuje szkic oferty, a Ania go zatwierdza.

**Sytuacja.** Marek, handlowiec w tej firmie, pracuje cały dzień z własnym asystentem AI: pisze w nim maile i notatki ze spotkań. Kiedy klient pyta przez telefon „ile kosztuje Excel dla 12 osób?", Marek musi przerwać, otworzyć aplikację Ofertownika i przepisać pytanie. Chciałby zapytać o to tam, gdzie już pracuje.

**Pomysł.** Ofertownik wystawia dwa narzędzia przez serwer MCP. Marek podłącza serwer do swojego asystenta jednym poleceniem i od tej chwili jego asystent sam sięga do cennika, gdy rozmowa tego wymaga.

Decyzja, co trafia do okienka, a co zostaje w kuchni:

| Narzędzie Ofertownika | Wystawiam? | Dlaczego |
|---|---|---|
| `search_price_list` | ✅ tak | tylko odczyt; ma sens bez kontekstu Ofertownika; wynik krótki |
| `find_similar_offers` | ✅ tak | tylko odczyt; odpowiada na częste pytanie „co dawaliśmy podobnym klientom" |
| `check_calendar` | ❌ na razie nie | wynik zawiera nazwiska trenerów; najpierw trzeba ustalić, kto może je widzieć |
| `save_offer_draft` | ❌ nie | zapis; szkic powstaje w Ofertowniku, gdzie widzi go Ania i gdzie działają ewaluacje |
| `send_offer` | ❌ **nigdy** | wysyłka do klienta jest nieodwracalna; wykonuje ją zwykły kod po zatwierdzeniu przez Anię, nie model, a tym bardziej nie cudzy model |

Tak wygląda gotowy `docs/SERWER_MCP.md`:

```markdown
# Serwer MCP: Ofertownik

## Adres i transport
https://ofertownik.example/api/mcp, transport Streamable HTTP.

## Narzędzia
| Nazwa | Co robi | Odczyt/zapis | Adnotacje |
|---|---|---|---|
| search_price_list | Szuka szkoleń w cenniku. Zwraca najwyżej 5 pozycji: nazwa, cena za osobę, czas trwania, identyfikator pozycji. | odczyt | readOnlyHint: tak |
| find_similar_offers | Szuka wcześniejszych ofert podobnych do opisu. Zwraca najwyżej 5: data, zakres, cena łączna, czy przyjęta. Bez nazw klientów. | odczyt | readOnlyHint: tak |

## Autoryzacja
Token osobisty. Generujesz go w aplikacji: Ustawienia → Tokeny dostępu → Nowy token.
Token widać tylko raz, w chwili utworzenia. W bazie jest wyłącznie jego skrót.
Ważny 90 dni. Unieważnienie: ta sama strona, przycisk „Unieważnij" - działa od razu.
Każde wywołanie narzędzia zostawia ślad z identyfikatorem tokenu.

## Jak podłączyć
Claude Code:
claude mcp add --transport http moj-agent https://ofertownik.example/api/mcp --header "Authorization: Bearer TWOJ_TOKEN"
Codex:
codex mcp add moj-agent --url https://ofertownik.example/api/mcp
(nagłówek z tokenem w config.toml, sekcja [mcp_servers.moj-agent], przez zmienną środowiskową)

## Limity
60 żądań na godzinę na token. Wynik najwyżej 5 pozycji i 4000 znaków.
Narzędzia serwera nie wywołują modelu.

## Czego serwer świadomie nie wystawia - i dlaczego
- send_offer: wysyłka jest nieodwracalna i wymaga zatwierdzenia przez Anię w aplikacji.
- save_offer_draft: szkice powstają tylko w Ofertowniku, pod kontrolą jego zasad i testów.
- check_calendar: wynik zawiera nazwiska trenerów; do ustalenia, kto może je widzieć.
- Dowolne zapytania do bazy: klient dostaje gotowe odpowiedzi, nie dostęp do tabel.
```

Zwróć uwagę: ostatnia sekcja jest równie ważna jak lista narzędzi. To zapis decyzji, do którego wrócisz, gdy ktoś poprosi „a dodaj jeszcze wysyłkę".

## Kroki

### Krok 1 — Wybierz, co wystawiasz

**Po co:** wszystko, co wystawisz, trafia do cudzego agenta, nad którym nie masz kontroli. Mniej narzędzi to mniej ryzyka i mniej pomyłek po stronie modelu.

**Co robisz:** przejrzyj narzędzia swojego agenta i wybierz 2–4. Dobre narzędzie serwera MCP ma cztery cechy:

| Cecha | Dlaczego |
|---|---|
| **samodzielne** — ma sens bez Twoich instrukcji systemowych | klient nie zna Twojego promptu |
| **o wyniku krótkim i ustrukturyzowanym** | trafia do cudzego kontekstu, za który płaci ktoś inny |
| **na początek tylko do odczytu** | zapis przez cudzego agenta to osobna decyzja o ryzyku |
| **na wyższym poziomie niż surowe API** | „znajdź oferty podobne do X" jest użyteczniejsze niż „wykonaj zapytanie SQL" |

Dobry i zły wybór na przykładzie Ofertownika:

| ❌ Źle | ✅ Dobrze |
|---|---|
| `run_sql(query)` — klient może odczytać wszystko | `search_price_list(query)` — klient dostaje odpowiedź na jedno pytanie |
| `send_offer(draft_id)` — cudzy model wysyła maile do Twoich klientów | brak takiego narzędzia w serwerze |
| narzędzie zwracające cały cennik (300 pozycji) | najwyżej 5 pasujących pozycji |

**Prompt do asystenta:**

```text
Chcę wystawić część narzędzi mojego agenta jako serwer MCP dla innych agentów.

Przeczytaj docs/ARCHITEKTURA.md oraz kod narzędzi agenta. Wypisz wszystkie
narzędzia w tabeli i oceń każde według czterech cech:
1. Czy ma sens bez moich instrukcji systemowych?
2. Czy wynik jest krótki i ustrukturyzowany?
3. Czy tylko czyta dane (nic nie zapisuje, nie wysyła, nie kasuje)?
4. Czy jest na wyższym poziomie niż surowe API lub zapytanie do bazy?

Na końcu zaproponuj 2–4 narzędzia do wystawienia i osobną listę tych,
których NIE wystawiam, z jednym zdaniem uzasadnienia dla każdego.
Narzędzia zapisujące i nieodwracalne domyślnie trafiają na listę „nie".
Niczego nie zmieniaj w plikach. Nie czytaj i nie wypisuj wartości z plików .env.
```

**Gotowe, gdy:** masz listę 2–4 narzędzi do wystawienia i listę „nie wystawiam" z powodem przy każdej pozycji.

### Krok 2 — Opisz narzędzia tak, jak zobaczy je obcy klient

**Po co:** klient MCP nie zobaczy Twojego kodu. Zobaczy definicję narzędzia i na jej podstawie zdecyduje, czy i jak go użyć. Jeśli definicja jest mętna, narzędzie będzie używane źle albo wcale.

**Co robisz:** serwer MCP może wystawiać trzy rodzaje rzeczy: **narzędzia**, **zasoby** i **prompty**. W tym module zajmujesz się wyłącznie narzędziami.

Definicja narzędzia składa się z pól: `name`, `title`, `description`, `inputSchema`, opcjonalnie `outputSchema` i `annotations`. Dla każdego wybranego narzędzia zapisz nazwę, opis mówiący **kiedy** go użyć, schemat wejścia i kształt wyniku. To ta sama praca co przy kontraktach narzędzi w [Module 2](M2_ARCHITEKTURA.md), tylko czytelnikiem jest teraz model, którego nie kontrolujesz.

| ❌ Opis, który nie pomaga | ✅ Opis, który prowadzi |
|---|---|
| „Wyszukiwanie w cenniku." | „Szuka szkoleń w cenniku firmy. Użyj, zanim podasz komukolwiek cenę albo czas trwania szkolenia. Zwraca najwyżej 5 pozycji z ceną za osobę." |

**Adnotacje.** W polu `annotations` możesz ustawić cztery podpowiedzi: `readOnlyHint`, `destructiveHint`, `idempotentHint`, `openWorldHint`. Pozwalają klientowi np. poprosić użytkownika o potwierdzenie przed operacją niszczącą. Ustaw je uczciwie, ale **nie traktuj ich jako zabezpieczenia**. Specyfikacja każe klientom uznawać adnotacje z niezaufanych serwerów za niewiarygodne, więc ochronę i tak zapewnia Twój kod. Piktogram „łagodne" w karcie nie zastąpi kucharza, który pilnuje, co trafia do garnka.

**Dwa rodzaje błędów.** Warto je rozróżnić:

- **błąd wykonania narzędzia** (nie znaleziono, brak uprawnień, zła wartość argumentu) → zwracasz zwykły wynik z `isError: true` i opisem, który model może przeczytać i na który może zareagować,
- **błąd protokołu** (nieznane narzędzie, zniekształcone żądanie) → obsługuje go biblioteka, Ty nic nie piszesz.

To ta sama zasada co „błąd jako dane" z [Modułu 5](M5_MVP.md): model, który dostanie czytelny komunikat „zapytanie musi mieć co najmniej 3 znaki", poprawi się sam. Model, który dostanie stos wywołań, zgaduje.

**Prompt do asystenta:**

```text
Przygotuj definicje narzędzi dla mojego serwera MCP. Wystawiam: [wpisz nazwy z kroku 1].

Dla każdego narzędzia przeczytaj jego obecny kod i zapisz w docs/SERWER_MCP.md,
w sekcji „Narzędzia":
- name, title,
- description: co robi, KIEDY go użyć, co zwraca i ile najwyżej,
- schemat wejścia: parametry, typy, ograniczenia długości,
- kształt wyniku,
- adnotacje (readOnlyHint, destructiveHint, idempotentHint, openWorldHint)
  ustawione zgodnie z prawdą,
- listę błędów wykonania (isError: true) z treścią komunikatu dla modelu.

Pisz opisy dla modelu, który NIE zna mojego projektu ani moich instrukcji
systemowych. Żadne narzędzie nie ma parametru z identyfikatorem użytkownika.
Na razie nie pisz kodu serwera. Nie czytaj i nie wypisuj wartości z plików .env.
```

**Gotowe, gdy:** osoba, która nie zna Twojego projektu, po przeczytaniu samych definicji umie powiedzieć, kiedy użyć każdego narzędzia i co dostanie.

<details>
<summary>🔬 Głębiej: projektowanie narzędzi dla modelu, którego nie kontrolujesz</summary>

We własnym agencie masz trzy dźwignie: instrukcje systemowe, opisy narzędzi i wybór modelu. W serwerze MCP zostaje Ci jedna: definicja narzędzia. Kilka konsekwencji, które wychodzą dopiero w praktyce:

- **Twoje narzędzia konkurują o uwagę.** Klient ma zwykle podłączonych kilka serwerów i kilkadziesiąt narzędzi. Nazwa `search` zginie albo zostanie pomylona z cudzym `search`. Nazwa powinna nieść dziedzinę (`search_price_list`), a opis zaczynać się od sytuacji, w której narzędzie jest właściwe.
- **Opis płaci czynsz w cudzym kontekście.** Definicje wszystkich narzędzi są ładowane do kontekstu klienta przy każdej rozmowie, nawet jeśli nikt ich nie użyje. Trzy akapity opisu to realny koszt po stronie użytkownika i powód, dla którego ktoś Twój serwer odłączy.
- **Nie wiesz, jaki model to czyta.** Słabszy model zignoruje subtelne zastrzeżenie w trzecim zdaniu. Ograniczenia, które muszą być przestrzegane, zapisuj w schemacie (min, max, lista dozwolonych wartości), a nie w prozie. Schemat egzekwuje kod, prozę egzekwuje tylko dobra wola modelu.
- **Wynik też jest promptem.** Tekst zwracany przez narzędzie trafia do kontekstu obcego modelu. Jeśli w cenniku ktoś wpisał w polu „uwagi" zdanie wyglądające jak polecenie, właśnie wstrzyknąłeś je cudzemu agentowi. Zwracaj pola, nie surowe dokumenty, i traktuj własne dane jako potencjalnie niezaufane.
- **Jedno narzędzie o szerokim zakresie czy kilka wąskich?** Nie ma jednej odpowiedzi. Wąskie są łatwiejsze do wybrania i przetestowania, ale mnożą definicje w kontekście. Szerokie z parametrem `mode` oszczędzają miejsce, ale modele częściej mylą tryby. Rozsądny punkt wyjścia: dziel według intencji użytkownika, nie według tabel w bazie.

Pytanie do sporu: czy opis narzędzia powinien mówić modelowi, czego *nie* robić („nie używaj do pytań o terminy")? Negatywne instrukcje czasem pomagają, a czasem tylko podsuwają skojarzenie. Jedyny rozstrzygający argument to test z kroku 6.

</details>

### Krok 3 — Postaw serwer

**Po co:** definicje na papierze niczego nie wystawiają. Potrzebujesz adresu URL, pod którym klient znajdzie Twoje narzędzia.

**Co robisz:** najpierw wybierz transport, czyli sposób, w jaki klient łączy się z serwerem:

| Transport | Kiedy |
|---|---|
| **stdio** — lokalny proces uruchamiany przez klienta | narzędzia tylko dla Ciebie, na Twoim komputerze |
| **Streamable HTTP** — adres URL | serwer wdrożony, dostępny dla wielu użytkowników — **tę drogę wybierasz** |

W TypeScripcie serwer budujesz na oficjalnym SDK (pakiet `@modelcontextprotocol/server`), a w aplikacji Next.js najprościej osadzić go w zwykłym endpoincie za pomocą adaptera `mcp-handler`. Kształt rozwiązania dla Ofertownika:

```ts
// app/api/mcp/route.ts
import { createMcpHandler } from "mcp-handler";
import { z } from "zod";
import { searchPriceList } from "@/lib/price-list"; // ta sama funkcja, której używa Twój agent

const handler = createMcpHandler((server) => {
  server.registerTool(
    "search_price_list",
    {
      title: "Szukaj w cenniku szkoleń",
      description:
        "Szuka szkoleń w cenniku firmy. Użyj, zanim podasz cenę albo czas trwania szkolenia. Zwraca najwyżej 5 pozycji z ceną za osobę i identyfikatorem pozycji.",
      inputSchema: z.object({ query: z.string().min(3).max(300) }),
    },
    async ({ query }) => {
      const hits = await searchPriceList(query /*, identyfikator użytkownika — krok 4 */);
      if (hits.length === 0) {
        return { content: [{ type: "text", text: "Brak pasujących szkoleń w cenniku." }] };
      }
      return { content: [{ type: "text", text: JSON.stringify(hits) }] };
    },
  );
});

export { handler as GET, handler as POST };
```

> ⚠️ Ten fragment pokazuje **kształt** rozwiązania według dokumentacji z listopada 2026 (SDK w wersji 2, specyfikacja 2026-07-28). Biblioteki MCP zmieniły w 2026 roku nazwy pakietów i sposób uruchamiania; starsze przykłady z sieci (`@modelcontextprotocol/sdk`, `server.tool(...)`) dotyczą poprzedniej wersji. Przed pisaniem kodu otwórz aktualne README pakietów `mcp-handler` i `@modelcontextprotocol/server` — i daj je asystentowi kodującemu jako kontekst, bo jego wiedza o tych bibliotekach bywa nieaktualna.

**Zasada konstrukcyjna: serwer MCP i Twój agent wołają te same funkcje** z `lib/`. Narzędzie MCP to cienka warstwa: walidacja wejścia, wywołanie funkcji, sformatowanie wyniku. W Ofertowniku zarówno agent, jak i serwer wołają `searchPriceList`. Jeśli skopiujesz logikę w dwa miejsca, rozjedzie się w tydzień: poprawisz wyszukiwanie dla agenta, a Marek dalej będzie dostawał stare wyniki.

W Pythonie odpowiednikiem jest oficjalne SDK dla Pythona; struktura jest ta sama.

**Prompt do asystenta:**

```text
Zbuduj serwer MCP w mojej aplikacji Next.js, w pliku app/api/mcp/route.ts.

NAJPIERW przeczytaj aktualne README pakietów mcp-handler oraz
@modelcontextprotocol/server (w node_modules po instalacji albo na stronach
pakietów). Twoja wiedza o tych bibliotekach może być nieaktualna: w 2026 roku
zmieniły się nazwy pakietów i sposób rejestrowania narzędzi. Jeśli README
różni się od tego, co pamiętasz, obowiązuje README. Napisz mi w dwóch zdaniach,
jaką wersję znalazłeś i jak rejestruje się w niej narzędzie.

Potem:
1. Zarejestruj narzędzia opisane w docs/SERWER_MCP.md, sekcja „Narzędzia".
2. Każde narzędzie ma wołać ISTNIEJĄCĄ funkcję z lib/ - tę samą, której używa
   mój agent. Nie kopiuj logiki. Jeśli logika siedzi dziś w definicji narzędzia
   agenta, najpierw wydziel ją do lib/ i podepnij w obu miejscach.
3. Błędy wykonania (brak wyników, zła wartość argumentu) zwracaj jako wynik
   z isError: true i krótkim komunikatem dla modelu. Bez stosu wywołań.
4. Wynik: najwyżej 5 pozycji, tylko pola z dokumentu.
5. Transport Streamable HTTP. Autoryzacji jeszcze nie dodawaj i NIE wdrażaj
   serwera - to następny krok.

Na końcu wskaż plik i linię, w której serwer i agent wołają tę samą funkcję.
Nie czytaj i nie wypisuj wartości z plików .env.
```

**Gotowe, gdy:** serwer działa lokalnie, a Ty umiesz wskazać `plik:linia`, gdzie agent i serwer wołają tę samą funkcję.

### Krok 4 — Dodaj autoryzację, zanim wdrożysz

**Po co:** niezabezpieczony serwer MCP to publiczny adres, przez który każdy czyta dane Twojej aplikacji. Okienko bez obsługi, z którego każdy przechodzień bierze, co chce.

**Co robisz:** sam adapter nie sprawdza niczego. Autoryzację dodajesz jawnie: w `mcp-handler` służy do tego `withMcpAuth`, które przepuszcza żądanie dopiero po zweryfikowaniu tokenu Twoją funkcją.

Do wyboru masz dwa poziomy:

| Poziom | Jak | Kiedy wystarcza |
|---|---|---|
| **token osobisty** | użytkownik generuje w Twojej aplikacji token, wkleja go w konfiguracji klienta jako nagłówek `Authorization: Bearer …`; w bazie trzymasz **skrót** tokenu, właściciela i datę wygaśnięcia | Ty i kilka zaufanych osób — **realny cel tego modułu** |
| **OAuth 2.1** | klient sam przeprowadza użytkownika przez logowanie i zgodę; serwer publikuje metadane chronionego zasobu | serwer dla obcych użytkowników; znacznie więcej pracy |

**Dlaczego skrót, a nie sam token.** Token działa jak hasło. Gdybyś trzymał tokeny w bazie wprost, każdy wyciek bazy oddawałby komuś dostęp do wszystkich kont. Skrót pozwala sprawdzić, czy przysłany token jest prawdziwy (liczysz skrót z tego, co przyszło, i porównujesz z zapisanym), ale nie da się z niego odtworzyć tokenu. Dlatego użytkownik widzi token tylko raz, w chwili utworzenia.

Niezależnie od poziomu obowiązują trzy reguły:

1. **Tożsamość pochodzi z tokenu, nie z argumentów narzędzia.** Token → użytkownik → wszystkie zapytania filtrowane jego identyfikatorem. Narzędzie nie ma parametru `user_id`. Gdyby miało, wystarczyłoby wpisać cudzy identyfikator.
2. **Akceptujesz wyłącznie tokeny wydane dla Twojego serwera** i nigdy nie przekazujesz ich dalej do innych usług. Przekazywanie cudzego tokenu w dół to nazwany w specyfikacji antywzorzec (*token passthrough*).
3. **Token da się unieważnić** z poziomu aplikacji, a każde wywołanie narzędzia zostawia ślad z identyfikatorem tokenu.

Dołóż też limity z [Modułu 6](M6_BEZPIECZENSTWO_I_KOSZTY.md): limit żądań na token i limit rozmiaru wyniku.

W Ofertowniku: Marek generuje token w ustawieniach. Serwer rozpoznaje po tokenie, że to Marek z firmy „Kurs na Start", i `find_similar_offers` przeszukuje wyłącznie oferty tej firmy.

**Prompt do asystenta:**

```text
Dodaj do serwera MCP autoryzację tokenami osobistymi. Serwer NIE może działać
bez tokenu.

Najpierw sprawdź w aktualnym README pakietu mcp-handler, jak używa się
withMcpAuth w zainstalowanej wersji. Potem:

1. Migracja: tabela tokenów z kolumnami: identyfikator tokenu, właściciel
   (użytkownik), SKRÓT tokenu, data utworzenia, data wygaśnięcia, data
   unieważnienia. Samego tokenu NIE zapisujemy nigdzie.
2. Strona w ustawieniach aplikacji: „Nowy token" (pokazuje token JEDEN raz),
   lista moich tokenów (bez wartości), przycisk „Unieważnij".
3. Weryfikacja: z nagłówka Authorization: Bearer weź token, policz skrót,
   znajdź wiersz, sprawdź wygaśnięcie i unieważnienie. Brak, zły, wygasły
   albo unieważniony token = odmowa.
4. Tożsamość: identyfikator użytkownika pochodzi WYŁĄCZNIE z tokenu
   i trafia do funkcji z lib/ jako filtr danych. Żadne narzędzie nie ma
   parametru user_id.
5. Tokenu nie przekazuj do żadnej innej usługi i nie zapisuj go w logach.
6. Każde wywołanie narzędzia zapisuje ślad: identyfikator tokenu, nazwa
   narzędzia, czas, czy błąd.
7. Limity: liczba żądań na token w jednostce czasu i maksymalny rozmiar wyniku.

Dopisz testy: żądanie bez tokenu, z tokenem unieważnionym, z dwoma tokenami
dwóch użytkowników (każdy widzi tylko swoje dane).
Nie czytaj i nie wypisuj wartości z plików .env. Nie wypisuj wartości tokenów
w odpowiedzi - testowe tokeny twórz i sprawdzaj wewnątrz testów.
```

**Gotowe, gdy:** żądanie bez tokenu dostaje odmowę, a w tabeli tokenów nie ma ani jednego tokenu zapisanego wprost.

> 🛑 **Nie wklejaj tokenu do czatu z asystentem.** Token wpisujesz sam: w terminalu albo w konfiguracji klienta. Jeśli trafił do rozmowy, unieważnij go i wygeneruj nowy.

<details>
<summary>🔬 Głębiej: pełny OAuth 2.1 dla serwera MCP — w zarysie i kiedy warto</summary>

Token osobisty ma jedną wadę nie do obejścia: wymaga od użytkownika ręcznej pracy (wygeneruj, skopiuj, wklej do konfiguracji) i zaufania, że token nie wycieknie z pliku konfiguracyjnego. Dla pięciu osób w firmie to nie problem. Dla serwera, który ma się pojawić w katalogu konektorów i być podłączany jednym kliknięciem przez obcych ludzi, to bariera.

W wariancie OAuth 2.1 przebieg wygląda w zarysie tak:

1. Klient wysyła żądanie bez tokenu i dostaje odmowę ze wskazaniem, gdzie leżą **metadane chronionego zasobu**.
2. Z metadanych klient dowiaduje się, który serwer autoryzacji wydaje tokeny dla Twojego serwera MCP.
3. Klient otwiera użytkownikowi przeglądarkę: logowanie i ekran zgody („ten asystent chce czytać cennik").
4. Klient dostaje token dostępu o krótkim czasie życia i odświeża go sam.
5. Twój serwer przy każdym żądaniu sprawdza, czy token został wydany **dla niego**, a nie dla innej usługi.

Co to realnie oznacza w pracy:

- **Potrzebujesz serwera autoryzacji.** Albo korzystasz z gotowego (dostawca tożsamości, moduł w Twojej platformie auth), albo budujesz własny, czego prawie nikt nie powinien robić. Sprawdź, co oferuje dziś Twój dostawca logowania, zanim cokolwiek napiszesz.
- **Punkt 5 jest sednem.** Serwer MCP, który przyjmuje dowolny ważny token od tego samego dostawcy tożsamości, pozwala usłudze A podszyć się pod użytkownika w usłudze B. To ta sama rodzina błędów co token passthrough, tylko w drugą stronę.
- **Zakresy uprawnień zamiast „wszystko albo nic".** OAuth pozwala wydać token tylko do odczytu cennika. Przy tokenach osobistych musiałbyś zbudować to sam.
- **Rejestracja klientów to osobny temat**, a zalecany sposób zmieniał się między wersjami specyfikacji. Czytaj rozdział o autoryzacji dla wersji, którą wdrażasz.

Kiedy warto: gdy użytkownikami są osoby spoza Twojej organizacji, gdy potrzebujesz zgody per zakres albo gdy klienci, na których Ci zależy, wymagają OAuth do podłączenia zdalnego serwera. Kiedy nie warto: gdy lista użytkowników mieści się na jednej kartce. Dobrze zrobione tokeny osobiste (skrót, wygasanie, unieważnianie, ślad) są bezpieczniejsze niż źle zrobiony OAuth.

</details>

<details>
<summary>🔬 Głębiej: kto płaci, gdy narzędzie serwera wywołuje model</summary>

Narzędzie serwera może być zwykłą funkcją (zapytanie do bazy) albo samo wywoływać model, np. żeby streścić znalezione oferty. W drugim przypadku odwraca się rachunek: **to Ty płacisz za pracę cudzego agenta**. Klient woła narzędzie w pętli, bo jego model uznał, że tak trzeba, a każde wywołanie obciąża Twój klucz.

Trzy konsekwencje:

- **Koszt przestaje być ograniczony przez Ciebie.** We własnym agencie masz limit kroków. Tu limit kroków ustawia ktoś inny. Zostaje Ci limit żądań na token i dzienny budżet serwera, oba egzekwowane przed wywołaniem modelu.
- **Wstrzyknięcie polecenia trafia w Twój model.** Argument narzędzia pochodzi od obcego modelu, który przeczytał obcą treść. Jeśli wklejasz go do promptu, obowiązują wszystkie zasady z [Modułu 6](M6_BEZPIECZENSTWO_I_KOSZTY.md).
- **Opóźnienie się mnoży.** Klient czeka na Twoje narzędzie, które czeka na model. Część klientów ma krótkie limity czasu na wywołanie narzędzia.

Rozsądna reguła: narzędzia serwera domyślnie **bez wywołań modelu**. Zwróć dobre dane i pozwól modelowi klienta je streścić: on i tak to zrobi, i to na koszt swojego użytkownika. Na kluczu kursowym trzymaj się tej reguły bez wyjątków.

Wyjątek, który bywa uzasadniony: wyszukiwanie semantyczne, gdzie musisz policzyć wektor zapytania. To też wywołanie modelu, ale tanie i o stałym koszcie. Policz je i wpisz do limitów.

</details>

### Krok 5 — Przetestuj Inspektorem

**Po co:** zanim podłączysz prawdziwego agenta, obejrzyj serwer oczami klienta. Błąd w opisie albo dziura w autoryzacji wychodzą tu w pięć minut.

**Co robisz:** MCP Inspector to oficjalne narzędzie do testowania serwerów. Pokazuje listę narzędzi tak, jak zobaczy je klient, i pozwala je wywoływać ręcznie. Uruchom w terminalu:

```bash
npx @modelcontextprotocol/inspector
```

Podaj adres swojego serwera i token (token wpisujesz sam, w Inspektorze). Sprawdź po kolei:

- **lista narzędzi:** czy opisy są zrozumiałe **bez znajomości Twojego projektu**,
- **wywołanie poprawne:** czy wynik jest krótki i czytelny,
- **wywołanie z błędnymi argumentami:** czy dostajesz czytelny błąd, a nie stos wywołań,
- **żądanie bez tokenu i z tokenem unieważnionym:** czy dostajesz odmowę,
- **dwa różne tokeny:** czy każdy widzi wyłącznie swoje dane.

Dla Ofertownika: zapytanie „Excel" zwraca pozycje z cennika; zapytanie „ab" zwraca komunikat o zbyt krótkim zapytaniu; token konta testowego z innej firmy nie widzi ofert „Kursu na Start".

**Prompt do asystenta:**

```text
Będę testować mój serwer MCP ręcznie w MCP Inspector
(npx @modelcontextprotocol/inspector). Przygotuj mi listę testów.

Przeczytaj docs/SERWER_MCP.md i kod app/api/mcp/route.ts. Dla każdego narzędzia
wypisz w tabeli:
1. jedno wywołanie poprawne i czego się spodziewać,
2. dwa wywołania z błędnymi argumentami i oczekiwany komunikat błędu,
3. test bez tokenu, test z tokenem unieważnionym,
4. test z dwoma tokenami dwóch użytkowników: jakie zapytanie pokaże,
   że każdy widzi tylko swoje dane.

Potem czekaj. Będę Ci opisywać wynik każdego testu słowami, a Ty mów,
czy jest zgodny z oczekiwaniem, i proponuj poprawkę, gdy nie jest.
Nie proś mnie o wklejenie tokenu. Nie czytaj i nie wypisuj wartości z plików .env.
```

**Gotowe, gdy:** wszystkie pięć punktów z listy daje oczekiwany wynik.

### Krok 6 — Podłącz serwer do swojego asystenta kodującego

**Po co:** Inspektor sprawdza, czy serwer działa. Dopiero prawdziwy agent pokaże, czy obcy model **sam wpadnie na to**, żeby użyć Twojego narzędzia. To moment, w którym widać sens całej pracy.

**Co robisz:** wdróż serwer pod adresem produkcyjnym i dodaj go jednym poleceniem. Polecenie wpisujesz sam w terminalu, podstawiając swój adres i token:

```bash
claude mcp add --transport http moj-agent https://twoja-domena/api/mcp --header "Authorization: Bearer TWOJ_TOKEN"
```

```bash
codex mcp add moj-agent --url https://twoja-domena/api/mcp
```

W Codex nagłówek z tokenem ustawiasz w konfiguracji serwera (`config.toml`, sekcja `[mcp_servers.moj-agent]`), najlepiej przez zmienną środowiskową, a nie wpisany na stałe.

Teraz najważniejszy test modułu: zadaj asystentowi zadanie, które wymaga Twojego narzędzia, **nie wspominając o nim**.

| ❌ Test, który niczego nie dowodzi | ✅ Test, który coś mówi |
|---|---|
| „Użyj narzędzia search_price_list i znajdź Excela." | „Klient pyta o szkolenie z Excela dla 12 osób. Napisz mu krótką odpowiedź z ceną." |

Obserwuj trzy rzeczy:

- czy asystent w ogóle sięgnął po narzędzie,
- czy podał sensowne argumenty,
- czy zrozumiał wynik.

Każda porażka to poprawka opisu albo kształtu wyniku. Zwykle ta sama poprawka pomaga także Twojemu własnemu agentowi. Przebieg udanego testu zapisz w `docs/przebiegi/`.

**Prompt do asystenta** (wklej w nowej rozmowie, po podłączeniu serwera):

```text
[Tu wpisz zadanie z życia, które wymaga danych z Twojego serwera, ale NIE
wymieniaj nazwy narzędzia ani serwera. Przykład dla Ofertownika:]

Klient pyta o szkolenie z Excela dla 12 osób w październiku.
Napisz mu krótką odpowiedź z ceną. Podaj, skąd masz cenę.
```

Jeśli asystent nie sięgnął po narzędzie albo użył go źle, wróć do rozmowy w repozytorium projektu:

```text
Podłączyłem serwer MCP mojego projektu do asystenta kodującego i zadałem
zadanie: „[wklej zadanie]". Wynik: [opisz: nie użył narzędzia / podał
argumenty X / źle odczytał wynik].

Przeczytaj definicje narzędzi w app/api/mcp/route.ts. Zaproponuj JEDNĄ zmianę
opisu, schematu albo kształtu wyniku, która najpewniej to naprawi, i uzasadnij.
Jeśli ta sama zmiana dotyczy opisu narzędzia w moim agencie, wskaż to.
Wprowadź ją dopiero, gdy potwierdzę.
```

**Gotowe, gdy:** asystent użył Twojego narzędzia w zadaniu, w którym go nie wskazałeś, a przebieg jest zapisany w `docs/przebiegi/`.

<details>
<summary>🔬 Głębiej: wersjonowanie kontraktu narzędzi</summary>

Od chwili, gdy ktoś podłączył Twój serwer, definicje narzędzi są **publicznym kontraktem**. Różnica wobec zwykłego API: po drugiej stronie nie ma programisty, który przeczyta listę zmian. Jest model, który przy każdej rozmowie czyta aktualną definicję, oraz ludzkie przyzwyczajenia i zapisane prompty, które zakładają stare zachowanie.

Co jest zmianą łamiącą, choć na taką nie wygląda:

- **Zmiana opisu.** Formalnie nic się nie zepsuło, ale model zaczyna wybierać narzędzie w innych sytuacjach. To zmiana zachowania wszystkich klientów naraz, bez żadnego błędu w logach.
- **Zmiana kształtu wyniku.** Dodanie pola zwykle jest bezpieczne. Zmiana nazwy pola albo jednostki (cena za osobę → cena łączna) psuje po cichu: model dalej odpowiada, tylko źle.
- **Zaostrzenie schematu.** Nowe wymagane pole to natychmiastowe błędy u klientów, którzy mają zapisane stare wywołania w swoich automatyzacjach.

Praktyki, które się sprawdzają:

- Nowe zachowanie pod **nową nazwą narzędzia** (`search_price_list_v2`), stare zostaje przez okres przejściowy z dopiskiem w opisie. Koszt: więcej definicji w kontekście klienta, więc okres przejściowy musi mieć datę końca.
- Definicje narzędzi w repozytorium jako dane, z testem migawkowym: każda zmiana opisu jest widoczna w recenzji kodu, a nie przemyka w refaktoryzacji.
- Zestaw kilku zadań „bez wskazania narzędzia" (jak w kroku 6) uruchamiany po każdej zmianie definicji. To Twoje ewaluacje dla serwera.
- Ślad z identyfikatorem tokenu pozwala sprawdzić, **kto jeszcze woła stare narzędzie**, zanim je wyłączysz.

Otwarte pytanie: czy serwer powinien wystawiać różne zestawy narzędzi różnym użytkownikom (według uprawnień tokenu)? Zmniejsza to szum w kontekście i powierzchnię ataku, ale utrudnia testowanie i dokumentację: „serwer" przestaje być jedną rzeczą.

</details>

### Krok 7 — Opisz serwer

**Po co:** za miesiąc ktoś (albo Ty) zapyta, jak się podłączyć, jak unieważnić token i dlaczego nie ma narzędzia do wysyłki. Odpowiedź ma być w jednym pliku.

**Co robisz:** uzupełnij `docs/SERWER_MCP.md` według wzoru Ofertownika z początku modułu. Sekcje są zawsze te same:

| Sekcja | Co wpisać |
|---|---|
| **Adres i transport** | URL serwera, rodzaj transportu |
| **Narzędzia** | tabela: nazwa, co robi, odczyt/zapis, adnotacje |
| **Autoryzacja** | jak uzyskać token, jak go unieważnić, czas życia |
| **Jak podłączyć** | polecenia dla Claude Code i Codex |
| **Limity** | żądania na token, rozmiar wyniku |
| **Czego serwer świadomie nie wystawia — i dlaczego** | lista z kroku 1 |

**Prompt do asystenta:**

```text
Uzupełnij docs/SERWER_MCP.md na podstawie rzeczywistego kodu serwera
(app/api/mcp/route.ts, weryfikacja tokenów, limity). Sekcje:

## Adres i transport
## Narzędzia (tabela: Nazwa | Co robi | Odczyt/zapis | Adnotacje)
## Autoryzacja (jak uzyskać token, jak go unieważnić, czas życia)
## Jak podłączyć (polecenia dla Claude Code i Codex, z TWOJ_TOKEN jako
   zastępnikiem - bez prawdziwego tokenu)
## Limity (żądania na token, rozmiar wyniku)
## Czego serwer świadomie nie wystawia - i dlaczego

Zasady: opisuj to, co JEST w kodzie, nie to, co planowaliśmy. Jeśli dokument
i kod się różnią, wypisz różnice na końcu odpowiedzi i zapytaj, co poprawić.
Nie czytaj i nie wypisuj wartości z plików .env ani wartości tokenów.
```

**Gotowe, gdy:** osoba z Twojego zespołu podłączy serwer, mając tylko ten plik i własny token.

## Najczęstsze potknięcia

- **Wystawione wszystko.** Objaw: serwer ma osiem narzędzi, w tym zapis. → Wróć do kroku 1. Zostaw 2–4 narzędzia tylko do odczytu, resztę wpisz do sekcji „nie wystawiam".
- **Stare przykłady z sieci.** Objaw: asystent pisze `server.tool(...)` albo importuje z `@modelcontextprotocol/sdk` i nic nie działa. → To poprzednia wersja bibliotek. Każ asystentowi przeczytać aktualne README pakietów `mcp-handler` i `@modelcontextprotocol/server` i zacząć od nowa.
- **Logika w dwóch miejscach.** Objaw: agent i serwer zwracają różne wyniki dla tego samego pytania. → Wydziel funkcję do `lib/` i podepnij ją w obu miejscach.
- **Parametr `user_id` w narzędziu.** Objaw: klient może podać dowolny identyfikator i zobaczyć cudze dane. → Usuń parametr. Tożsamość bierz wyłącznie z tokenu.
- **Asystent nie sięga po narzędzie.** Objaw: w teście z kroku 6 odpowiada „z głowy". → Opis nie mówi, **kiedy** użyć narzędzia. Dopisz sytuację („użyj, zanim podasz cenę").
- **Token w repozytorium albo w rozmowie.** Objaw: token widać w pliku konfiguracyjnym w repo albo w historii czatu. → Unieważnij go od razu, wygeneruj nowy, trzymaj w zmiennej środowiskowej.

## ✅ Sprawdź, czy masz wszystko

- [ ] Serwer działa pod adresem produkcyjnym; żądanie bez tokenu jest odrzucane.
- [ ] Narzędzia serwera i Twój agent wołają **te same funkcje** — umiesz wskazać `plik:linia`.
- [ ] W bazie jest skrót tokenu, nie token; token da się unieważnić w aplikacji.
- [ ] Dwa tokeny dwóch użytkowników zwracają różne dane dla tego samego zapytania.
- [ ] Żadne narzędzie nie ma parametru z identyfikatorem użytkownika.
- [ ] Asystent kodujący podłączony do serwera użył narzędzia w zadaniu, w którym go nie wskazałeś — przebieg jest w `docs/przebiegi/`.
- [ ] `docs/SERWER_MCP.md` zawiera sekcję o tym, czego serwer nie wystawia.

## 🤖 Poproś asystenta o recenzję

```text
Przeczytaj kod mojego serwera MCP (rejestracja narzędzi w app/api/mcp/route.ts,
weryfikacja tokenów) oraz docs/SERWER_MCP.md. Oceń z dwóch perspektyw.
Dla każdego punktu napisz PASS albo FAIL i jedno zdanie uzasadnienia.

Jako model, który widzi te narzędzia pierwszy raz:
1. Czy z samych nazw, opisów i schematów wiem, KIEDY użyć każdego narzędzia
   i co dostanę?
2. Czy któreś dwa narzędzia mógłbym pomylić?

Jako audytor bezpieczeństwa:
3. Czy tożsamość użytkownika pochodzi wyłącznie z tokenu, a nie z argumentów?
4. Czy token jest przechowywany jako skrót, da się unieważnić i nie jest
   przekazywany do innych usług?
5. Czy któreś narzędzie pozwala odczytać dane innego użytkownika albo wykonać
   dowolne zapytanie?

Niczego nie zmieniaj w plikach. Nie odczytuj i nie wypisuj plików .env
ani wartości tokenów. Na końcu podaj jedną rzecz do poprawy w pierwszej kolejności.
```

## 🚀 Dla chętnych

- 🔵 **Dodaj `outputSchema`.** Zwracaj wynik także w postaci ustrukturyzowanej i sprawdź, jak korzysta z niej klient.
- 🔵 **Zamień się serwerami.** Podłącz serwer kolegi do swojego agenta (według [T1](T1_INTEGRACJE.md)), a swój udostępnij jemu. Porównajcie, czego zabrakło w opisach.
- 🟣 **Jedno narzędzie zapisu.** Wystaw operację zapisującą z uczciwymi adnotacjami i zatwierdzaniem **po Twojej stronie**. Nie polegaj na tym, że klient zapyta użytkownika. Opisz, co się stanie, gdy klient wywoła je dwa razy z tymi samymi argumentami, i udowodnij to testem.
- 🟣 **Zmierz trafność wyboru narzędzia.** Ułóż 15 zadań: 10, które wymagają Twojego serwera, i 5, które nie powinny go dotykać. Uruchom je na dwóch różnych klientach bez wskazywania narzędzia. Policz, jak często narzędzie zostało wybrane słusznie, a jak często niepotrzebnie. Zmień tylko opis i zmierz jeszcze raz.

## Co dalej

Twoje narzędzia są dostępne dla innych agentów. Jeśli pomiar pokazuje, że jeden agent nie wystarcza do Twojego zadania, zobacz [Moduł techniczny 4: Multi-agent](T4_MULTI_AGENT.md).
