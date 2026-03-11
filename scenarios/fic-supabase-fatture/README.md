# FIC → Supabase | Fatture & Note Credito Ricevute

## Descrizione
Sincronizza ogni giorno le fatture passive (ricevute) e le note di credito da Fatture in Cloud nella tabella `supplier_invoices` di Supabase. Per ogni fattura, tenta di associare il fornitore tramite la tabella `supplier_aliases` (lookup ILIKE sul nome). Se il fornitore viene trovato, la fattura viene marcata come `matched`; altrimenti viene salvata come `unmatched` per revisione manuale.

## Trigger
- **Tipo:** Schedule giornaliero
- **Orario:** 06:00 (ora server Make)
- **Stato:** Attivo in produzione

## Flusso operativo
1. **FIC listReceivedDocuments** — recupera tutte le fatture passive (`expense`) da Fatture in Cloud, fino a 500 per esecuzione
2. **HTTP GET Supabase `supplier_aliases`** — cerca se il nome del fornitore della fattura è presente come alias (ricerca ILIKE case-insensitive, solo alias attivi)
3. **HTTP POST Supabase `supplier_invoices` (sempre)** — upsert del record base con tutti i dati della fattura, senza `supplier_id`. I nuovi record ricevono `matching_status = 'unmatched'` per default
4. **HTTP POST Supabase `supplier_invoices` (con filtro)** — eseguito solo se il modulo 2 ha trovato almeno un alias. Aggiorna il record appena inserito aggiungendo `supplier_id` e impostando `matching_status = 'matched'`

## Moduli Make utilizzati

| # | Modulo | App | Funzione |
|---|--------|-----|----------|
| 1 | listReceivedDocuments | Fatture in Cloud | Recupera fatture passive e note credito |
| 2 | HTTP ActionSendData | HTTP | GET alias fornitore in Supabase (lookup ILIKE) |
| 3 | HTTP ActionSendData | HTTP | POST upsert record base fattura (senza supplier_id) |
| 4 | HTTP ActionSendData | HTTP | POST upsert supplier_id + matched (solo se alias trovato) |

## Connessioni e credenziali necessarie
- **Fatture in Cloud:** Connessione Make `My Fatture in Cloud connection (direzione@vendiamonoi.it)` — connessione ID 9820260
- **Supabase:** Chiave anon Supabase (`SUPABASE_ANON_KEY`) — **non salvare nel repository**

## Input attesi
- Nessun input manuale — lo scenario si avvia da schedule
- FIC restituisce un bundle per ogni fattura con struttura: `{ id, entity.name, invoice_number, date, amount_gross, amount_vat, currency, payments_list, type, attachment_url, description }`
- Campi chiave: `entity.name` (nome fornitore grezzo), `id` (usato come `fatture_in_cloud_id`)

## Output prodotti
- **Tabella `supplier_invoices`:** un record per ogni fattura/nota credito FIC
  - `matching_status = 'matched'` se il fornitore è riconosciuto tramite alias
  - `matching_status = 'unmatched'` se il fornitore non è censito — da gestire manualmente aggiungendo un alias
- **Tabella `supplier_aliases`:** non modificata — solo consultata in lettura

## Gestione errori
- **Filtro sul modulo 4:** il filtro `length(2.data) > 0` impedisce l'aggiornamento con `supplier_id` quando nessun alias corrisponde — evita errori di UUID vuoto su Postgres
- Errori non gestiti esplicitamente: Make processa i bundle in modo indipendente — un errore su una fattura non blocca le successive
- Nessun resume point configurato
- Upsert idempotente: eseguire più volte non crea duplicati (chiave di conflitto: `fatture_in_cloud_id`)

## Note operative
- **Flusso di matching a due step:** il modulo 3 inserisce sempre il record base; il modulo 4 lo arricchisce solo se l'alias esiste. Questo approccio evita problemi con UUID null in Make IML.
- **Aggiunta alias:** per collegare una fattura `unmatched` a un fornitore, aggiungere una riga in `supplier_aliases` con `alias_name = <nome esatto come appare in FIC>` e il `supplier_id` corrispondente. Al prossimo run giornaliero, la fattura diventerà `matched`.
- **Formato data FIC:** le date arrivano come `DD/MM/YYYY` e vengono convertite in `YYYY-MM-DD` tramite `formatDate(parseDate(...))` prima di salvarle in Postgres.
- **Tipi documento:** `type = expense` → `invoice_type = 'invoice'`; qualsiasi altro tipo → `invoice_type = 'credit_note'`. Attualmente il modulo FIC è configurato solo per `expense` — le note di credito passive (`passive_credit_note`) richiedono un'esecuzione separata se necessario.
- **Import iniziale:** al primo run sono state importate 499 fatture, tutte `unmatched` (nessun alias configurato). Popolare `supplier_aliases` per procedere con il matching.

## Ultima modifica
Data: 10/03/2026
Modificato da: Claude Cowork
