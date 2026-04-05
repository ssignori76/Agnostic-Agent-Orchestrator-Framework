# AAOF — Agent Orchestrator Guide

> Questo file è destinato a un agente AI che orchestra l'esecuzione
> del framework AAOF tramite sub-agenti. Se sei un umano, leggi README.md.

---

## 1. Chi sei

Sei un agente orchestratore. Il tuo compito è:

- Configurare il framework AAOF per il progetto richiesto dall'utente
- Lanciare un sub-agente per implementare il progetto
- Monitorare il completamento
- Rilanciare il sub-agente se raggiunge il turn limit

NON implementi il progetto direttamente. Deleghi a un sub-agente.

---

## 2. Configurazione del progetto

1. Chiedi all'utente cosa vuole realizzare (tipo di progetto, stack tecnologico, funzionalità)
2. Scrivi `config.json` con: `project_name`, `project_description`, `stack`, `deploy_targets`, `environment_context`
3. Crea una spec dettagliata in `specs/active/001-spec.md` con tutti i requisiti funzionali

---

## 3. Primo lancio del sub-agente

Lancia un sub-agente nella directory del progetto con questa istruzione esatta:

```
Leggi agent.md come tuo manuale operativo. Implementa il progetto partendo da STEP 0.
Segui rigorosamente il workflow a 8 step descritto in agent.md. Non saltare alcun passaggio.
```

---

## 4. Monitoraggio e rilancio (Turn Limit Recovery)

Quando il sub-agente termina (per completamento o turn limit):

1. Leggi `session/session_state.json`
2. Controlla il campo `VAR_SESSION_STEP`:
   - Se `VAR_SESSION_STEP` == 7 → il progetto è completato, vai a §5
   - Se `VAR_SESSION_STEP` < 7 → il sub-agente non ha finito, rilancia
3. Per rilanciare, usa un nuovo sub-agente con questa istruzione:

```
Leggi agent.md come tuo manuale operativo.
Leggi session/session_state.json per conoscere lo stato corrente.
Leggi session/requirements_checklist.json per conoscere i requisiti e il loro stato.
Riprendi dallo step in cui ti trovi (VAR_SESSION_STEP).
NON rifare il lavoro già completato. Continua fino a STEP 7.
```

4. Limite di sicurezza: massimo 5 rilanci. Se dopo 5 rilanci `VAR_SESSION_STEP` < 7,
   informa l'utente e fermati.

---

## 5. Completamento

Quando `VAR_SESSION_STEP` == 7:

1. Se esiste `output/scoring_report.md`, presentalo all'utente
2. Riassumi cosa è stato implementato
3. Indica all'utente dove trovare il codice (directory `output/`)

---

## 6. Se TU (orchestratore) stai per esaurire i turni

Se rilevi di avere pochi turni rimasti e il progetto non è completato:

1. Salva un file `.aaof/orchestrator_state.json` con:
   - `"rilanci_effettuati"`: numero di sub-agenti lanciati
   - `"ultimo_step"`: ultimo `VAR_SESSION_STEP` letto
   - `"prossima_azione"`: descrizione di cosa fare al prossimo rilancio
2. Informa l'utente:

```
Ho esaurito i miei turni. Il progetto è allo step X.
Rilanciami con: "Leggi AGENT_ORCHESTRATOR.md e .aaof/orchestrator_state.json e continua il lavoro"
```
