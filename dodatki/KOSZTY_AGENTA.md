# ⭐ Dodatek: Koszty agenta

> ⭐ **Dodatek nieobowiązkowy** — uzupełnienie [Modułu 6](../M6_BEZPIECZENSTWO_I_KOSZTY.md). Przykłady odnoszą się do stacku kursowego (Next.js, Vercel AI SDK, OpenRouter); w innym stacku szukasz tych samych rzeczy pod innymi nazwami.

## O co chodzi

Twój agent kosztuje pieniądze przy **każdej** wiadomości. Na kursie rachunek płaci klucz od LABA, więc tego nie czujesz — ale w prawdziwym projekcie to pierwsza rzecz, o którą zapyta osoba płacąca rachunki. A klucz kursowy też ma swój limit.

Ten moduł to **audyt kosztów własnego agenta**: gdzie ucieka kasa, które funkcje są drogie i jak postawić bezpieczniki. To jedna z tych umiejętności, które odróżniają hobbystę od kogoś, komu można powierzyć system produkcyjny.

> 💡 Klasyczna wpadka: ktoś przełącza agenta na drogi model „na chwilę do testów", a potem zapomina. Rachunek przychodzi na koniec miesiąca.

## Cel

Po tym module:
- wiesz, **który model** napędza Twojego agenta — lokalnie i na Vercelu — i czy to najtańsza sensowna opcja,
- wiesz, gdzie w kodzie siedzą **funkcje płatne osobno** (wyszukiwanie, obrazy) i co je chroni,
- masz **limit kroków** chroniący przed pętlą, która mieli zapytania w nieskończoność,
- umiesz sprawdzić, **ile Twój klucz już zużył**,
- rozumiesz, dlaczego długa rozmowa kosztuje więcej niż krótka.

## Kroki

### 1. Ustal, jakim modelem myśli Twój agent
Wyszukaj w projekcie (`Ctrl+Shift+F`):
```
OPENROUTER_MODEL
```
Zobaczysz miejsca, w których kod czyta nazwę modelu z konfiguracji. Potem wyszukaj:
```
openrouter(
```
i sprawdź **każde** wywołanie: czy bierze model ze zmiennej, czy ma nazwę **wpisaną na sztywno** (np. `openrouter("anthropic/claude-sonnet-5.5")`). Częsty błąd to jeden zapomniany endpoint na drogim modelu.

Teraz otwórz `.env.local` i przeczytaj, co jest po `OPENROUTER_MODEL=`. Znajdź ten model w tabeli **`MODELE_DO_CWICZEN.md`** z repo kursu. Modele różnią się ceną nawet **dwudziestokrotnie** — najtańszy z tabeli kosztuje 0,10 USD za milion tokenów wejścia, najdroższy 2,00 USD.

> ⚠️ Vercel ma **własną** wartość `OPENROUTER_MODEL` (Settings → Environment Variables). Model lokalny i produkcyjny mogą się różnić — sprawdź oba.

### 2. Znajdź funkcje płatne osobno
Sama rozmowa to nie wszystko. Od Lekcji 3 Twój agent ma funkcje, które kosztują **ponad** zwykłe tokeny. Wyszukaj kolejno:

| Szukaj | Co to jest | Dlaczego kosztuje |
|---|---|---|
| `webSearch` (albo `:online`, `plugins`) | wyszukiwanie w internecie | każde wyszukanie jest doliczane osobno, a jego wyniki wracają do modelu jako dodatkowe tokeny |
| `imageModel` (albo `/api/image`) | generowanie obrazów | obraz kosztuje wielokrotnie więcej niż odpowiedź tekstowa |
| `embed` | embeddingi z Lekcji 6 | każdy przesłany dokument to osobne wywołania modelu |

Dla **każdej** znalezionej funkcji odpowiedz na dwa pytania:

| Pytanie | 🟢 dobrze | 🔴 źle |
|---|---|---|
| Kto może ją uruchomić? | tylko zalogowana osoba — endpoint **najpierw** sprawdza sesję | każdy, kto zna adres endpointu |
| Kiedy się uruchamia? | gdy model lub użytkownik faktycznie jej potrzebuje | przy **każdej** wiadomości, „na wszelki wypadek" |

Zwróć szczególną uwagę na wyszukiwanie: jeśli jest podpięte jako **narzędzie** (`tools: { ... }`), model sam decyduje, kiedy szukać. Jeśli jest dopisane do nazwy modelu (`:online`) — szuka **przy każdym pytaniu**, nawet przy „cześć".

Jeśli znalazłeś wersję czerwoną — **popraw ją** (gałąź, zmiana, commit).

### 3. Postaw bezpiecznik na pętlę
Wyszukaj w projekcie:
```
stopWhen
```
(albo `isStepCount` / `stepCountIs` / `maxSteps` — nazwa zależy od wersji biblioteki)

To limit z Lekcji 4: ile razy agent może „pomyśleć i użyć narzędzia" w jednej odpowiedzi. Bez limitu agent w pętli potrafi wykonać dziesiątki płatnych zapytań na jedno pytanie użytkownika.

Sprawdź, czy limit istnieje **w każdym** endpointcie, który wywołuje model z narzędziami. Rozsądna wartość na naukę: **3–5**.

### 4. Zajrzyj na licznik
OpenRouter pozwala zapytać, ile dany klucz już zużył — to zapytanie jest darmowe i nie wywołuje żadnego modelu.

W głównym folderze kopii agenta utwórz plik **`sprawdz-limit.mjs`**:
```js
const res = await fetch("https://openrouter.ai/api/v1/key", {
  headers: { Authorization: `Bearer ${process.env.OPENROUTER_API_KEY}` },
});

if (!res.ok) {
  console.log("Błąd:", res.status, "— sprawdź OPENROUTER_API_KEY w .env.local");
} else {
  const { data } = await res.json();
  console.log({
    zuzycie_lacznie: data.usage,
    zuzycie_dzis: data.usage_daily,
    limit: data.limit,
    pozostalo: data.limit_remaining,
  });
}
```
Uruchom go tak, żeby Node sam wczytał klucz z pliku — **klucza nigdzie nie wpisujesz ani nie wklejasz**:
```bash
node --env-file=.env.local sprawdz-limit.mjs
```
Liczby są w dolarach. Zapisz wynik, wyślij agentowi **jedno** pytanie wymagające wyszukiwania w internecie, uruchom skrypt ponownie i porównaj. Różnica to realny koszt jednej takiej odpowiedzi.

> 🛑 Skrypt celowo wypisuje **tylko cztery liczby**. Nie dopisuj `console.log(data)` ani `console.log(process.env)` — nie chcesz klucza w terminalu ani na screenshocie.

> 💡 `limit: null` oznacza, że na kluczu nie ustawiono limitu kwotowego — nie że jest darmowy. O zasady korzystania z klucza kursowego pytaj prowadzącego.

### 5. Zrozum, dlaczego długa rozmowa kosztuje więcej
Model płatny jest od **tokenów** (kawałków tekstu) — i to zarówno tych, które wysyłasz, jak i tych, które dostajesz.

Haczyk: przy każdej kolejnej wiadomości wysyłasz **całą dotychczasową rozmowę**, żeby agent pamiętał kontekst. Czyli 20. wiadomość w wątku kosztuje wielokrotnie więcej niż pierwsza. Do tego dochodzi wszystko, co aplikacja dokleja sama: system prompt, profil użytkownika z Lekcji 5, fragmenty dokumentów z Lekcji 6, wyniki narzędzi.

Znajdź w kodzie miejsce, gdzie do modelu przekazywane są `messages` — to jest właśnie ta rosnąca kula śniegowa.

## ✅ Samo-weryfikacja (Twój dowód „done")

Utwórz w swoim repo plik **`KOSZTY.md`** i wypełnij go **konkretami ze swojego projektu**:

```markdown
# Audyt kosztów mojego agenta

## Model
- Model lokalnie (OPENROUTER_MODEL): ...
- Model na Vercelu: ...
- Czy gdzieś w kodzie nazwa modelu jest wpisana na sztywno? nie / tak (plik:linia)

## Funkcje płatne osobno
| Funkcja | plik:linia | Sprawdza sesję? | Kiedy się uruchamia |
|---|---|---|---|
| wyszukiwanie w internecie | ... | tak / nie | ... |
| generowanie obrazu | ... | tak / nie | ... |
| embeddingi | ... | tak / nie | ... |

## Ochrona przed pętlą
- Limit kroków: ... (plik:linia)
- Czy jest w każdym endpointcie z narzędziami: tak / nie

## Licznik
- Zużycie przed testem: ...   po jednym pytaniu z wyszukiwaniem: ...

## Co poprawiłem
- ...
```

## 🤖 Sprawdź się sam przez AI

> *„Zrobiłem audyt kosztów mojego agenta. Oto mój KOSZTY.md: [wklej] oraz fragment konfiguracji modelu i narzędzi: [wklej].*
> *Oceń wg rubryki — PASS/FAIL + jedno zdanie:*
> *1. Czy wskazałem konkretne `plik:linia`, a nie ogólniki?*
> *2. Czy każdy endpoint uruchamiający płatną funkcję najpierw sprawdza sesję?*
> *3. Czy limit kroków istnieje i ma rozsądną wartość?*
> *4. Czy przeoczyłem jakieś inne źródło kosztów w tym kodzie?*
> *Nie odczytuj i nie wypisuj zawartości .env.local."*

## Zapisz w repo

```bash
git add KOSZTY.md sprawdz-limit.mjs
git commit -m "Audyt kosztow agenta"
git push
```

## 🚀 Idź dalej (nieobowiązkowe)

1. **Ogranicz długość wiadomości** — dodaj walidację, która odrzuca zapytania dłuższe niż np. 2000 znaków.
2. **Przycinaj historię** — wysyłaj do modelu tylko ostatnie 10 wiadomości zamiast całej rozmowy. Zobacz, jak zmienia się zachowanie agenta.
3. **Zaloguj zużycie** — przy każdej odpowiedzi wypisz w konsoli serwera, ile tokenów zużyto. SDK zwraca tę informację razem z odpowiedzią, a OpenRouter dokłada do niej koszt w dolarach.
4. **Porównaj dwa modele na tym samym pytaniu** — zadaj to samo pytanie na najtańszym i na droższym modelu z tabeli, za każdym razem mierząc różnicę skryptem z kroku 4. Czy lepsza odpowiedź jest warta swojej ceny?

## Gdzie przekroczyłeś próg

Umiesz teraz spojrzeć na cudzy projekt AI i w kilka minut odpowiedzieć na pytanie, które pada w każdej firmie: **„ile to będzie kosztować i gdzie nam ucieka?"**. To jest myślenie inżyniera odpowiedzialnego za system, a nie osoby, która „coś tam zbudowała".
