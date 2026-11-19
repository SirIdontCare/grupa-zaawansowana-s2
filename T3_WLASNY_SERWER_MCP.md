# Moduł techniczny 3: Własny serwer MCP

> ⏱️ Ok. 5 godzin.
> 🧩 **Moduł do wyboru.** Weź go, jeśli chcesz, żeby z możliwości Twojego agenta korzystały **inne agenty**: Twój agent kodujący, asystent w edytorze, agent kolegi z zespołu. Wymaga działających narzędzi z [Modułu 5](M5_MVP.md). O korzystaniu z cudzych serwerów MCP jest [T1](T1_INTEGRACJE.md).

## O co chodzi

W T1 byłeś klientem: podłączałeś cudzy serwer MCP do swojego agenta. Tutaj odwracasz role. Narzędzia, które napisałeś dla swojego agenta — wyszukiwanie w Twojej bazie wiedzy, odczyt Twoich danych, Twoja logika — wystawiasz jako serwer MCP. Od tej chwili może ich użyć każdy klient MCP, bez pisania integracji pod Ciebie.

To zmienia sposób myślenia o projekcie: przestajesz budować jedną aplikację z czatem, a zaczynasz budować **zdolność**, którą da się wpiąć w wiele miejsc. To także najlepszy test jakości Twoich kontraktów narzędzi — obcy model nie zna Twoich instrukcji systemowych i ma do dyspozycji wyłącznie nazwę, opis i schemat.

## Cel

Po tym module:
- masz **zdalny serwer MCP** wystawiający 2–4 narzędzia Twojego projektu,
- serwer **wymaga autoryzacji** i działa z uprawnieniami konkretnego użytkownika,
- przetestowałeś go Inspektorem i podłączyłeś do swojego agenta kodującego,
- rozumiesz, czym serwer MCP różni się od zwykłego API i jakie ryzyka wnosi.

## Kroki

### 1. Wybierz, co wystawiasz
Nie wystawiaj wszystkiego, co masz. Dobre narzędzie serwera MCP:

| Cecha | Dlaczego |
|---|---|
| **samodzielne** — ma sens bez Twoich instrukcji systemowych | klient nie zna Twojego promptu |
| **o wyniku krótkim i ustrukturyzowanym** | trafia do cudzego kontekstu, za który płaci ktoś inny |
| **na początek tylko do odczytu** | zapis przez cudzego agenta to osobna decyzja o ryzyku |
| **na wyższym poziomie niż surowe API** | „znajdź zgłoszenia podobne do X" jest użyteczniejsze niż „wykonaj zapytanie SQL" |

Zapisz 2–4 narzędzia. Dla każdego: nazwa, opis mówiący **kiedy** go użyć, schemat wejścia, kształt wyniku — dokładnie jak w [Module 2](M2_ARCHITEKTURA.md), tylko czytelnikiem jest teraz model, którego nie kontrolujesz.

### 2. Zrozum, co klient zobaczy
Serwer MCP wystawia trzy rodzaje rzeczy: **narzędzia**, **zasoby** i **prompty**. W tym module zajmujesz się narzędziami.

Definicja narzędzia to: `name`, `title`, `description`, `inputSchema`, opcjonalnie `outputSchema` i `annotations`. Adnotacje (`readOnlyHint`, `destructiveHint`, `idempotentHint`, `openWorldHint`) to **podpowiedzi** dla klienta — pozwalają mu np. poprosić użytkownika o potwierdzenie przed operacją niszczącą. Ustaw je uczciwie, ale nie traktuj jako zabezpieczenia: specyfikacja każe klientom uznawać adnotacje z niezaufanych serwerów za niewiarygodne, więc ochronę i tak zapewnia Twój kod.

Błędy zwracasz na dwa sposoby i warto je rozróżnić:
- **błąd wykonania narzędzia** (nie znaleziono, brak uprawnień, zła wartość argumentu) → zwykły wynik z `isError: true` i opisem, który model może przeczytać i na który może zareagować,
- **błąd protokołu** (nieznane narzędzie, zniekształcone żądanie) → obsługuje biblioteka.

To ta sama zasada co „błąd jako dane" z Modułu 5.

### 3. Postaw serwer
Dwa sposoby uruchamiania serwera MCP:

| Transport | Kiedy |
|---|---|
| **stdio** — lokalny proces uruchamiany przez klienta | narzędzia tylko dla Ciebie, na Twoim komputerze |
| **Streamable HTTP** — adres URL | serwer wdrożony, dostępny dla wielu użytkowników — **tę drogę wybierasz** |

W TypeScripcie serwer budujesz na oficjalnym SDK (pakiet `@modelcontextprotocol/server`), a w aplikacji Next.js najprościej osadzić go w zwykłym endpoincie za pomocą adaptera `mcp-handler`. Kształt rozwiązania:

```ts
// app/api/mcp/route.ts
import { createMcpHandler } from "mcp-handler";
import { z } from "zod";
import { searchKnowledge } from "@/lib/knowledge"; // ta sama funkcja, której używa Twój agent

const handler = createMcpHandler((server) => {
  server.registerTool(
    "search_knowledge",
    {
      title: "Szukaj w bazie wiedzy",
      description:
        "Szuka fragmentów dokumentów pasujących do pytania. Użyj, zanim odpowiesz na pytanie o treść dokumentów. Zwraca najwyżej 5 fragmentów ze źródłem.",
      inputSchema: z.object({ query: z.string().min(3).max(300) }),
    },
    async ({ query }) => {
      const hits = await searchKnowledge(query /*, identyfikator użytkownika — krok 4 */);
      if (hits.length === 0) {
        return { content: [{ type: "text", text: "Brak pasujących fragmentów." }] };
      }
      return { content: [{ type: "text", text: JSON.stringify(hits) }] };
    },
  );
});

export { handler as GET, handler as POST };
```

> ⚠️ Ten fragment pokazuje **kształt** rozwiązania według dokumentacji z listopada 2026 (SDK w wersji 2, specyfikacja 2026-07-28). Biblioteki MCP zmieniły w 2026 roku nazwy pakietów i sposób uruchamiania; starsze przykłady z sieci (`@modelcontextprotocol/sdk`, `server.tool(...)`) dotyczą poprzedniej wersji. Przed pisaniem kodu otwórz aktualne README pakietów `mcp-handler` i `@modelcontextprotocol/server` — i wklej je agentowi kodującemu jako kontekst, bo jego wiedza o tych bibliotekach bywa nieaktualna.

Zasada konstrukcyjna: **serwer MCP i Twój agent wołają te same funkcje** z `lib/`. Narzędzie MCP to cienka warstwa: walidacja wejścia, wywołanie funkcji, sformatowanie wyniku. Jeśli logika jest zdublowana — rozjedzie się w tydzień.

W Pythonie odpowiednikiem jest oficjalne SDK dla Pythona; struktura jest ta sama.

### 4. Dodaj autoryzację — zanim wdrożysz
Niezabezpieczony serwer MCP to publiczny endpoint, przez który każdy czyta dane Twojej aplikacji. Sam adapter nie sprawdza niczego — autoryzację dodajesz jawnie (w `mcp-handler` służy do tego `withMcpAuth`, które przepuszcza żądanie dopiero po zweryfikowaniu tokenu Twoją funkcją).

Do wyboru masz dwa poziomy:

| Poziom | Jak | Kiedy wystarcza |
|---|---|---|
| **token osobisty** | użytkownik generuje w Twojej aplikacji token, wkleja go w konfiguracji klienta jako nagłówek `Authorization: Bearer …`; w bazie trzymasz **skrót** tokenu, właściciela i datę wygaśnięcia | Ty i kilka zaufanych osób — **realny cel tego modułu** |
| **OAuth 2.1** | klient sam przeprowadza użytkownika przez logowanie i zgodę; serwer publikuje metadane chronionego zasobu | serwer dla obcych użytkowników; znacznie więcej pracy |

Niezależnie od poziomu obowiązują trzy reguły:

1. **Tożsamość pochodzi z tokenu, nie z argumentów narzędzia.** Token → użytkownik → wszystkie zapytania filtrowane jego identyfikatorem. Narzędzie nie ma parametru `user_id`.
2. **Akceptujesz wyłącznie tokeny wydane dla Twojego serwera** i nigdy nie przekazujesz ich dalej do innych usług. Przekazywanie cudzego tokenu w dół to nazwany w specyfikacji antywzorzec (*token passthrough*).
3. **Token da się unieważnić** z poziomu aplikacji, a każde wywołanie narzędzia zostawia ślad z identyfikatorem tokenu.

Dołóż też limity z [Modułu 6](M6_BEZPIECZENSTWO_I_KOSZTY.md): limit żądań na token i limit rozmiaru wyniku. Jeśli narzędzie serwera samo wywołuje model (np. streszcza wyniki) — to **Ty** płacisz za cudzego agenta; na kluczu kursowym tym bardziej trzymaj narzędzia serwera bez wywołań modelu.

### 5. Przetestuj Inspektorem
MCP Inspector to oficjalne narzędzie do testowania serwerów — pokazuje listę narzędzi tak, jak zobaczy je klient, i pozwala je wywoływać ręcznie:

```bash
npx @modelcontextprotocol/inspector
```

Sprawdź po kolei:
- lista narzędzi: czy opisy są zrozumiałe **bez znajomości Twojego projektu**,
- wywołanie poprawne: czy wynik jest krótki i czytelny,
- wywołanie z błędnymi argumentami: czy dostajesz czytelny błąd, a nie stos wywołań,
- żądanie bez tokenu i z tokenem unieważnionym: czy dostajesz odmowę,
- dwa różne tokeny: czy każdy widzi wyłącznie swoje dane.

### 6. Podłącz do swojego agenta kodującego
To moment, w którym widać sens całej pracy. Zdalny serwer dodajesz jednym poleceniem:

```bash
claude mcp add --transport http moj-agent https://twoja-domena/api/mcp --header "Authorization: Bearer TWOJ_TOKEN"
```

```bash
codex mcp add moj-agent --url https://twoja-domena/api/mcp
```

W Codex nagłówek z tokenem ustawiasz w konfiguracji serwera (`config.toml`, sekcja `[mcp_servers.moj-agent]`), najlepiej przez zmienną środowiskową, a nie wpisany na stałe.

Teraz zadaj agentowi kodującemu zadanie, które wymaga Twojego narzędzia, **nie wspominając o nim**. Obserwuj:
- czy w ogóle po nie sięgnął,
- czy podał sensowne argumenty,
- czy zrozumiał wynik.

Każda porażka to poprawka opisu lub kształtu wyniku — i zwykle ta sama poprawka pomaga także Twojemu własnemu agentowi.

### 7. Opisz serwer
`docs/SERWER_MCP.md`:

```markdown
## Adres i transport
## Narzędzia
| Nazwa | Co robi | Odczyt/zapis | Adnotacje |
## Autoryzacja
Jak uzyskać token, jak go unieważnić, czas życia.
## Jak podłączyć
Polecenia dla Claude Code i Codex.
## Limity
Żądania na token, rozmiar wyniku.
## Czego serwer świadomie nie wystawia — i dlaczego
```

## ✅ Dowód „done"

- Serwer działa pod adresem produkcyjnym; żądanie bez tokenu jest odrzucane.
- Narzędzia serwera i Twój agent wołają **te same funkcje** — wskaż `plik:linia`.
- Dwa tokeny dwóch użytkowników zwracają różne dane dla tego samego zapytania.
- Agent kodujący podłączony do serwera użył narzędzia w zadaniu, w którym go nie wskazałeś — zapisz przebieg w `docs/przebiegi/`.
- `docs/SERWER_MCP.md` zawiera sekcję o tym, czego serwer nie wystawia.

## 🤖 Sprawdź się przez AI

> *„Oto kod mojego serwera MCP (rejestracja narzędzi, autoryzacja) i jego dokumentacja: [wklej]. Oceń z dwóch perspektyw — PASS/FAIL + jedno zdanie:*
> *Jako model, który widzi te narzędzia pierwszy raz:*
> *1. Czy z samych nazw, opisów i schematów wiem, KIEDY użyć każdego narzędzia i co dostanę?*
> *2. Czy któreś dwa narzędzia mógłbym pomylić?*
> *Jako audytor bezpieczeństwa:*
> *3. Czy tożsamość użytkownika pochodzi wyłącznie z tokenu, a nie z argumentów?*
> *4. Czy token jest przechowywany jako skrót, da się unieważnić i nie jest przekazywany do innych usług?*
> *5. Czy któreś narzędzie pozwala odczytać dane innego użytkownika albo wykonać dowolne zapytanie?*
> *Nie odczytuj i nie wypisuj plików .env ani wartości tokenów."*

## 🚀 Idź dalej (nieobowiązkowe)

1. **Dodaj `outputSchema`.** Zwracaj wynik także w postaci ustrukturyzowanej i sprawdź, jak korzysta z niej klient.
2. **Jedno narzędzie zapisu.** Wystaw operację zapisującą z uczciwymi adnotacjami i zatwierdzaniem po Twojej stronie — nie polegaj na tym, że klient zapyta użytkownika.
3. **Zamień agentów.** Podłącz serwer kolegi do swojego agenta (według T1), a swój udostępnij jemu. Porównajcie, czego zabrakło w opisach.
