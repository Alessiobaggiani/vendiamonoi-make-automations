# Changelog — SellRapido → Supabase | Aggiornamento Stati Ordini

## v1.0 — 2026-03-16
**Creazione scenario (Make ID: 8867890)**
- Architettura 2 scenari: questo è Scenario B (aggiornamento stati) — vedi Scenario A (8856551) per importazione nuovi ordini
- Trigger: daily alle 05:00
- Finestra temporale: 45 giorni (`addDays(now; -45)`)
- Logica: HTTP POST SellRapido → BasicFeeder → PATCH Supabase
- Campi aggiornati: `marketplace_status`, `tracking_number`, `carrier`, `sellrapido_modified_at`
- PATCH mirato su `marketplace_order_id` — zero rischio sovrascrittura dati anagrafici
- Nessun errore se l'ordine non esiste in Supabase (PostgREST no-op)
- Scenario creato in stato inattivo — attivare dopo test
- Risparmio crediti: ~700 op/giorno totali (A+B) vs ~9.000 op/giorno precedenti (-92%)
