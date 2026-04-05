# AAOF — Istruzioni di Ripresa Sessione

> Questo prompt è destinato a un sub-agente che deve continuare
> un lavoro iniziato da un altro agente o sessione precedente.

---

## Istruzioni

1. Leggi `agent.md` — è il tuo manuale operativo completo
2. Leggi `session/session_state.json` — contiene lo step attuale (`VAR_SESSION_STEP`) e tutte le variabili di sessione
3. Leggi `session/requirements_checklist.json` — contiene i requisiti del progetto e il loro stato (pass/pending/fail)
4. Riprendi dal `VAR_SESSION_STEP` corrente — NON ripartire da STEP 0
5. NON rifare il lavoro già completato — i requisiti con status `"pass"` sono già soddisfatti
6. Continua fino a STEP 7 (Consolidation) o fino a esaurimento turni
7. Se raggiungi il turn limit prima di STEP 7, assicurati che `session/session_state.json` sia aggiornato con lo step corrente
