# 🧰 Narzędzia i linki

Co jest potrzebne na tej ścieżce. Część zależy od stacku, który wybierzesz w [Module 2](M2_ARCHITEKTURA.md).

---

## Zawsze

| Narzędzie | Po co | Link |
|---|---|---|
| **Git** + konto **GitHub** | repozytorium projektu | https://git-scm.com/downloads · https://github.com/signup |
| **Agent kodujący**: Codex albo Claude Code | główne narzędzie pracy od Modułu 3 | https://chatgpt.com/codex · https://claude.com/download |
| **Edytor** z podglądem diffów | czytanie tego, co agent kodujący zmienił | dowolny — np. https://code.visualstudio.com |
| **OpenRouter** — kluczem od LABA | dostęp do modeli; własne konto nie jest potrzebne | https://openrouter.ai/models |

## Stack kursowy

Jeśli zostajesz przy tym, co znasz z kursu głównego:

| Narzędzie | Po co | Link |
|---|---|---|
| **Node.js 22+** | uruchamianie aplikacji | https://nodejs.org |
| **Starter agenta** | gotowa baza: logowanie, Supabase, RAG, pętla z narzędziami | https://github.com/SirIdontCare/starter-agent-s2 |
| **Supabase** — nowy, osobny projekt | baza, logowanie, pgvector | https://supabase.com |
| **Vercel** — nowy, osobny projekt | wdrożenie (Moduł 7) | https://vercel.com |

## Własny stack

Dowolny język i framework. Jeśli wybierasz Pythona:

| Narzędzie | Po co | Link |
|---|---|---|
| **Python 3.11+** | język | https://www.python.org/downloads |
| **FastAPI**, **Pydantic AI**, **LangGraph** | pokazane w [dodatkach](dodatki/) | linki w tabeli dokumentacji |

> ⚠️ **Windows:** przy instalacji Pythona zaznacz **„Add Python to PATH"**.

---

## 📚 Dokumentacja

Biblioteki agentowe zmieniają się szybko. Przed użyciem konkretnej funkcji sprawdź dokumentację **dla wersji, którą masz zainstalowaną** — także wtedy, gdy kod podpowiada agent kodujący.

| Temat | Link |
|---|---|
| **OpenRouter** — API, modele, ceny | https://openrouter.ai/docs |
| **Vercel AI SDK** | https://ai-sdk.dev |
| **Next.js** | https://nextjs.org/docs |
| **Supabase** | https://supabase.com/docs |
| **Model Context Protocol (MCP)** | https://modelcontextprotocol.io |
| **AI SDK** — narzędzia z serwerów MCP | https://ai-sdk.dev/docs/ai-sdk-core/mcp-tools |
| **AI SDK** — agenci i podagenci | https://ai-sdk.dev/docs/agents/overview · https://ai-sdk.dev/docs/agents/subagents |
| **MCP** — SDK dla TypeScriptu, Inspector | https://github.com/modelcontextprotocol/typescript-sdk · https://modelcontextprotocol.io/docs/tools/inspector |
| **MCP** — serwer w aplikacji Next.js (`mcp-handler`) | https://github.com/vercel/mcp-handler |
| **Stripe** — Checkout, webhooki, testowanie | https://docs.stripe.com/payments/checkout · https://docs.stripe.com/webhooks · https://docs.stripe.com/testing |
| **Supabase** — logowanie przez dostawców (OAuth) | https://supabase.com/docs/guides/auth/social-login |
| **Supabase** — Cron i Queues | https://supabase.com/docs/guides/cron · https://supabase.com/docs/guides/queues |
| **Vercel** — limity funkcji i zadania cykliczne | https://vercel.com/docs/functions/limitations · https://vercel.com/docs/cron-jobs |
| **Next.js** — `after()` | https://nextjs.org/docs/app/api-reference/functions/after |
| **Codex** — plik `AGENTS.md` | https://agents.md |
| **Claude Code** — dokumentacja | https://code.claude.com/docs |
| **OpenTelemetry** — standard śladów | https://opentelemetry.io/docs |
| **OWASP** — zagrożenia w aplikacjach LLM | https://genai.owasp.org |
| **FastAPI** | https://fastapi.tiangolo.com |
| **Pydantic AI** | https://ai.pydantic.dev |
| **LangGraph** | https://langchain-ai.github.io/langgraph |
| **Mermaid** — diagramy w Markdownie | https://mermaid.js.org |

---

## 🗺️ Co, kiedy

- **Moduły 0–2** — edytor tekstu, git i przeglądarka. Żadnych instalacji.
- **Moduł 3** — agent kodujący i środowisko wybranego stacku.
- **Moduł 4** — nic nowego; ewaluacje to zwykły skrypt w Twoim języku.
- **Moduł 7** — konto na platformie wdrożeniowej i osobna baza produkcyjna.
- **Moduły techniczne T1–T4** — konto w usłudze, z którą integrujesz agenta; reszta jest w stacku, który już masz.
- **Moduł T5** — konto Stripe (środowisko testowe), Stripe CLI i **własny** klucz do modeli.
