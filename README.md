# Vendiamonoi — Make Automations

Repository di documentazione degli scenari Make di Vendiamonoi.it S.R.L.

## Struttura

```
/scenarios/
  /[nome-scenario]/
    README.md        ← documentazione scenario
    blueprint.json   ← schema del flusso (API key sostituite con placeholder)
    changelog.md     ← storico modifiche
```

> **Nota sicurezza:** I `blueprint.json` in questo repository usano `[SUPABASE_ANON_KEY]` e `[SELLRAPIDO_API_KEY]` come placeholder. Le chiavi reali si trovano nel pannello Make (variabili di scenario) e in Supabase (Settings > API). Non salvare mai chiavi reali in questo repository.

---

## Indice scenari

| Scenario | Descrizione breve | ID Make | Cartella | Stato | Ultima modifica |
|----------|-------------------|---------|----------|-------|-----------------|
| SellRapido → Supabase \| Ordini Marketplace | Sincronizza ordini da SellRapido in Supabase ogni 15 min | 8831616 | `sellrapido-supabase-ordini` | ✅ Attivo | 10/03/2026 |
| FIC → Supabase \| Fatture & Note Credito Ricevute | Importa fatture passive da Fatture in Cloud in Supabase ogni mattina | 8833326 | `fic-supabase-fatture` | ✅ Attivo | 10/03/2026 |

---

## Convenzioni

**Commit message:** `[SCENARIO] Nome scenario — azione fatta`
Esempi:
- `[SCENARIO] Ordini Marketplace — creazione iniziale`
- `[SCENARIO] FIC Fatture — aggiunto filtro note di credito`

**Quando aggiornare:**
- Ogni nuovo scenario → nuova cartella + README + changelog
- Ogni modifica a scenario esistente → aggiorna README + aggiungi riga changelog

**Linguaggio:** italiano, semplice e diretto.
