# Moduł techniczny 1: Integracje — OAuth i MCP

> ⏱️ Ok. 5 godzin.
> 🧩 **Moduł do wyboru.** Weź go, jeśli Twój agent ma działać na cudzych systemach: poczcie, kalendarzu, repozytoriach, komunikatorze, CRM. Najlepszy moment: po [Module 2](M2_ARCHITEKTURA.md), przed albo w trakcie [Modułu 5](M5_MVP.md).

## O co chodzi

Agent z kursu głównego działa na danych, które sam przechowuje: rozmowach i dokumentach w swojej bazie. Większość agentów, które naprawdę oszczędzają komuś czas, działa na danych **w cudzych systemach** — i w imieniu konkretnej osoby. To zmienia dwie rzeczy.

Po pierwsze, potrzebujesz **zgody użytkownika** na dostęp do jego konta w innej usłudze, i musisz ten dostęp bezpiecznie przechowywać. Od tego jest OAuth.

Po drugie, potrzebujesz **narzędzi** do tej usługi. Możesz je napisać sam, wołając jej API, albo skorzystać z gotowego serwera MCP (Model Context Protocol) — standardu, w którym usługa sama wystawia narzędzia dla agentów.

## Cel

Po tym module:
- rozróżniasz **logowanie do Twojej aplikacji** od **dostępu Twojej aplikacji do cudzej usługi**,
- umiesz uzyskać, przechować i odświeżyć token dostępu użytkownika — z najwęższym możliwym zakresem,
- umiesz podłączyć serwer MCP i **udostępnić agentowi tylko wybrane narzędzia**,
- masz jedną działającą integrację, opisaną w `docs/INTEGRACJE.md`.

## Kroki

### 1. Rozdziel dwie rzeczy, które wyglądają tak samo
„Zaloguj przez Google" i „daj agentowi dostęp do mojego kalendarza Google" używają tego samego protokołu, ale to dwie różne zgody:

| | Logowanie | Dostęp do zasobów |
|---|---|---|
| Pytanie | kim jesteś? | co wolno mi zrobić w Twoim imieniu? |
| Zakres (scope) | tożsamość: e-mail, profil | konkretne dane i operacje: odczyt kalendarza, wysyłka poczty |
| Co dostajesz | sesję w swojej aplikacji | **token dostępu** do API usługi (+ zwykle token odświeżania) |
| Ryzyko wycieku | przejęcie konta w Twojej aplikacji | **działanie w cudzej usłudze w imieniu użytkownika** |

Zapisz dla swojego agenta: do jakiej usługi, **jakie dokładnie zakresy** i do którego scenariusza ze specyfikacji każdy z nich jest potrzebny. Zakres, którego nie wymaga żaden scenariusz, wykreśl. „Tylko odczyt" zamiast „pełny dostęp" to najtańsze zabezpieczenie, jakie istnieje.

### 2. Wybierz drogę do narzędzi

| Droga | Kiedy | Uwagi |
|---|---|---|
| **Własne narzędzia nad API usługi** | potrzebujesz 2–4 operacji i chcesz pełnej kontroli nad opisem, wynikiem i uprawnieniami | najwięcej pracy, najmniej niespodzianek |
| **Serwer MCP wystawiony przez samą usługę** | usługa ma oficjalny, zdalny serwer MCP | autoryzacja zwykle przez OAuth; dostajesz komplet narzędzi — wybierasz podzbiór |
| **Serwer MCP od strony trzeciej** | nie ma oficjalnego | traktuj jak każdą obcą zależność z dostępem do danych użytkownika: sprawdź, kto go utrzymuje i co robi |

Częsta rozsądna decyzja: MCP do odczytu i eksploracji, własne narzędzie do tej jednej operacji zapisu, którą chcesz ściśle kontrolować.

### 3. Uzyskaj i przechowaj token
Przebieg jest zawsze ten sam, niezależnie od usługi:

1. użytkownik klika „Połącz z …" w Twojej aplikacji,
2. przekierowujesz go do usługi z listą zakresów,
3. usługa pyta go o zgodę i odsyła do Ciebie z **kodem**,
4. Twój **serwer** wymienia kod na token dostępu i token odświeżania,
5. zapisujesz tokeny przypisane do tego użytkownika,
6. gdy token dostępu wygaśnie — serwer wymienia token odświeżania na nowy.

W stacku kursowym kroki 1–4 może wykonać Supabase Auth (`signInWithOAuth` z opcją `scopes`), ale z dwoma zastrzeżeniami, które decydują o architekturze:

- **Supabase nie przechowuje tokenów dostawcy i ich nie odświeża.** Po zalogowaniu dostajesz w sesji `provider_token` i — jeśli dostawca go wyda — `provider_refresh_token`. Zapisanie ich i odświeżanie to zadanie Twojej aplikacji.
- **Token odświeżania nie zawsze przychodzi.** Na przykład Google nie wydaje go domyślnie — trzeba poprosić o dostęp offline dodatkowymi parametrami zapytania (`access_type: 'offline'`, `prompt: 'consent'`). Sprawdź w dokumentacji swojego dostawcy.

Zasady przechowywania:

| Zasada | Dlaczego |
|---|---|
| tokeny **tylko po stronie serwera**, w tabeli niedostępnej z przeglądarki (RLS bez polityk dla zwykłego użytkownika, odczyt wyłącznie kluczem serwerowym) | token w przeglądarce to token w rękach każdego skryptu na stronie |
| szyfrowanie w spoczynku, klucz szyfrujący w zmiennych środowiskowych | wyciek kopii bazy nie oznacza wycieku dostępów |
| tokeny **nigdy** w kontekście modelu, logach ani śladach przebiegów | model nie potrzebuje tokenu — potrzebuje wyniku narzędzia |
| przycisk „Odłącz" usuwa tokeny u Ciebie **i** unieważnia je u dostawcy | zgoda musi dać się cofnąć |

Token trafia do wywołania API **wewnątrz kodu narzędzia**, wybrany na podstawie identyfikatora użytkownika z sesji — tak samo jak w zasadzie „uprawnienia użytkownika, nie agenta" z [Modułu 6](M6_BEZPIECZENSTWO_I_KOSZTY.md).

### 4. Podłącz serwer MCP
MCP to protokół, w którym serwer wystawia agentowi trzy rodzaje rzeczy: **narzędzia** (funkcje do wywołania), **zasoby** (dane do odczytu) i **prompty** (gotowe szablony). W praktyce prawie zawsze chodzi o narzędzia.

Dwa sposoby połączenia:

| Transport | Do czego |
|---|---|
| **stdio** — serwer jako lokalny proces | praca na własnym komputerze, narzędzia agenta kodującego |
| **Streamable HTTP** — serwer pod adresem URL | aplikacja wdrożona w internecie; **to jest droga dla Twojego agenta** |

Zdalne serwery MCP autoryzują się przez OAuth 2.1 albo — w prostszych przypadkach — kluczem w nagłówku.

W Vercel AI SDK klienta tworzy `createMCPClient` z pakietu `@ai-sdk/mcp`:

```ts
import { createMCPClient } from "@ai-sdk/mcp";
import { z } from "zod";

const mcp = await createMCPClient({
  transport: {
    type: "http",
    url: process.env.MCP_SERVER_URL!,
    headers: { Authorization: `Bearer ${tokenUzytkownika}` },
  },
});

try {
  // Podając `schemas`, pobierasz TYLKO wymienione narzędzia — nie cały serwer
  const tools = await mcp.tools({
    schemas: {
      search_issues: {
        inputSchema: z.object({ query: z.string(), state: z.enum(["open", "closed", "all"]) }),
      },
    },
  });
  // ...przekaż `tools` do wywołania modelu, razem z własnymi narzędziami
} finally {
  await mcp.close();
}
```

> ⚠️ Ten fragment pokazuje **kształt** rozwiązania według dokumentacji AI SDK z listopada 2026. Nazwy narzędzi i schematy zależą od serwera, a szczegóły API — od wersji pakietu. Przed użyciem sprawdź https://ai-sdk.dev/docs/ai-sdk-core/mcp-tools dla wersji, którą masz zainstalowaną. Przy odpowiedzi strumieniowanej klienta zamykasz po zakończeniu strumienia, nie w `finally`.

W innych stackach szukasz tego samego: klient MCP → lista narzędzi → **filtr** → przekazanie do modelu → zamknięcie.

### 5. Ogranicz to, co serwer MCP daje agentowi
Serwer MCP to cudzy kod, który dopisuje tekst do kontekstu Twojego modelu i wykonuje operacje na danych użytkownika. Stosujesz do niego wszystko z [Modułu 2](M2_ARCHITEKTURA.md) i [Modułu 6](M6_BEZPIECZENSTWO_I_KOSZTY.md), plus cztery rzeczy specyficzne:

| Ryzyko | Co robisz |
|---|---|
| serwer ma 40 narzędzi, agent potrzebuje 3 | pobierasz **jawną listę**; reszta nie istnieje dla modelu. Mniej narzędzi to także mniej tokenów i mniej pomyłek |
| wśród narzędzi są operacje zapisu i usuwania | nie udostępniasz ich — albo opakowujesz w zatwierdzanie przez człowieka z [Modułu 5](M5_MVP.md) |
| **opisy narzędzi to tekst od strony trzeciej, który trafia do kontekstu** | przeczytaj je; opis może zawierać polecenia dla modelu tak samo jak złośliwy dokument |
| serwer zmienia narzędzia po Twojej stronie bez uprzedzenia | przypinasz wersję tam, gdzie się da; zapisujesz listę i opisy narzędzi, na których przeszły ewaluacje, i porównujesz przy starcie |
| wynik narzędzia to niezaufana treść | traktujesz jak dane, nie polecenia — bez wyjątków |

Sprawdź też ponownie tabelę „trzech cech" z Modułu 6: integracja prawie zawsze dodaje **dane prywatne** albo **kanał wyjścia na zewnątrz**.

### 6. Obsłuż to, co w integracjach psuje się zawsze
Dopisz do ewaluacji i przetestuj ręcznie:

- **token wygasł** → narzędzie odświeża i ponawia raz; jeśli się nie da — zwraca modelowi błąd jako dane, a użytkownikowi prośbę o ponowne połączenie,
- **użytkownik cofnął zgodę u dostawcy** → to samo, bez pętli ponowień,
- **brak zakresu** → czytelny komunikat, którego zakresu brakuje,
- **limit zapytań usługi** → błąd z informacją, po jakim czasie spróbować; agent nie ponawia w kółko,
- **usługa nie odpowiada** → limit czasu, zadanie kończy się częściowym wynikiem i jasną informacją,
- **użytkownik nie połączył konta** → agent mówi o tym wprost, zamiast zmyślać dane.

### 7. Opisz integrację
`docs/INTEGRACJE.md`:

```markdown
## <usługa>
**Po co:** scenariusze S2, S4
**Droga:** serwer MCP usługi / własne narzędzia nad API
**Zakresy:** `issues:read` — S2; `comments:write` — S4 (za zatwierdzeniem)
**Narzędzia udostępnione agentowi:** search_issues, read_issue, propose_comment
**Narzędzia serwera świadomie pominięte:** wszystkie operacje delete_*, zarządzanie uprawnieniami
**Tokeny:** tabela `provider_tokens`, tylko serwer, szyfrowane; odświeżanie w <plik:linia>
**Odłączenie:** <jak działa, co jest usuwane>
**Znane ograniczenia:** limit 60 zapytań/h; brak powiadomień o zmianach
```

## ✅ Dowód „done"

- Użytkownik może połączyć i **odłączyć** konto w zewnętrznej usłudze z poziomu aplikacji.
- Tokeny nie są dostępne z przeglądarki ani obecne w logach i śladach — sprawdź to wyszukiwaniem.
- Agent wykonuje co najmniej jeden scenariusz na **prawdziwych danych** z zewnętrznej usługi.
- Agent widzi tylko jawnie wybrane narzędzia; operacje zapisu przechodzą przez zatwierdzanie.
- W ewaluacjach są przypadki: wygasły token, brak połączonego konta, błąd usługi.

## 🤖 Sprawdź się przez AI

> *„Oto kod integracji mojego agenta z zewnętrzną usługą (OAuth, przechowywanie tokenów, narzędzia lub klient MCP) i dokument integracji: [wklej]. Oceń jak audytor bezpieczeństwa — PASS/FAIL + jedno zdanie:*
> *1. Czy żądane zakresy są minimalne dla opisanych scenariuszy?*
> *2. Czy token może trafić do przeglądarki, kontekstu modelu, logów albo śladów?*
> *3. Czy token jest wybierany na podstawie sesji użytkownika, a nie argumentu podanego przez model?*
> *4. Czy agent dostaje jawnie ograniczoną listę narzędzi, a operacje zapisu wymagają zatwierdzenia?*
> *5. Co się stanie, gdy token wygaśnie w połowie zadania?*
> *Nie odczytuj i nie wypisuj plików .env ani wartości tokenów."*

## 🚀 Idź dalej (nieobowiązkowe)

1. **Wystaw własny serwer MCP.** Udostępnij narzędzia swojego agenta jako serwer MCP i podłącz go do swojego agenta kodującego — prowadzi przez to [T3](T3_WLASNY_SERWER_MCP.md).
2. **Druga usługa.** Dodaj drugą integrację i scenariusz, który łączy obie. Zmierz, jak zmieniła się liczba kroków i koszt zadania.
3. **Wykrywanie zmian.** Zapisuj skrót listy i opisów narzędzi serwera MCP i przerywaj start, gdy się zmieni — do czasu ponownego przejścia ewaluacji.
