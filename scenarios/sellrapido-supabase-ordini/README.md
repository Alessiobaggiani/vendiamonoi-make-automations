# SellRapido → Supabase | Ordini Marketplace

## Descrizione
Recupera gli ordini marketplace degli ultimi 2 ore da SellRapido e li sincronizza nella tabella `orders` di Supabase. Per ogni ordine, itera anche le righe prodotto e le salva in `order_items`. La logica è idempotente: se un ordine esiste già viene aggiornato, non duplicato.

## Trigger
- **Tipo:** Schedule ricorrente
- **Frequenza:** Ogni 15 minuti (interval: 900 secondi)
- **Stato:** Attivo in produzione

## Flusso operativo
1. **HTTP POST SellRapido** — chiama l'API SellRapido con finestra temporale -2h/adesso, status `standby/accepted/shipped/delivered`, formato JSON
2. **Iterator ordini** — itera ogni ordine nell'array `orders` restituito da SellRapido
3. **HTTP GET Supabase `marketplaces`** — cerca l'ID interno del marketplace tramite `channel_code` dell'ordine (lookup per `slug`)
4. **HTTP POST Supabase `orders`** — upsert dell'ordine su chiave `(marketplace_id, marketplace_order_id)` con `Prefer: resolution=merge-duplicates`
5. **Set Variable `orderId`** — salva l'UUID dell'ordine appena inserito/aggiornato
6. **Iterator righe prodotto** — itera ogni riga (`rows`) dell'ordine corrente
7. **HTTP POST Supabase `order_items`** — upsert della riga prodotto su chiave `(order_id, product_sku)` con `Prefer: resolution=merge-duplicates`

## Moduli Make utilizzati

| # | Modulo | App | Funzione |
|---|--------|-----|----------|
| 1 | HTTP ActionSendData | HTTP | POST API SellRapido per recupero ordini |
| 2 | BasicFeeder | Built-in | Iterator su array `orders` |
| 3 | HTTP ActionSendData | HTTP | GET marketplace lookup su Supabase |
| 4 | HTTP ActionSendData | HTTP | POST upsert ordine in `orders` |
| 5 | SetVariable2 | Util | Salva `orderId` per usarlo nel modulo 7 |
| 6 | BasicFeeder | Built-in | Iterator su `rows` (righe prodotto) |
| 7 | HTTP ActionSendData | HTTP | POST upsert riga prodotto in `order_items` |

## Connessioni e credenziali necessarie
- **SellRapido:** API key embedded nell'URL (formato `ae030eaf-...` nel path)
- **Supabase:** Chiave anon Supabase (variabile `SUPABASE_ANON_KEY`) — **non salvare la chiave nel repository, usare variabile ambiente Make**

## Input attesi
- Nessun input manuale — lo scenario si avvia da schedule
- SellRapido restituisce un JSON con struttura: `{ orders: [ { head: {...}, rows: [...] } ] }`
- Campi chiave usati: `head.channel_code`, `head.code`, `head.buyer_*`, `head.price`, `rows[].sku`, `rows[].ean`, `rows[].price`, `rows[].price_cost`, `rows[].tracking`

## Output prodotti
- **Tabella `orders`:** record creato o aggiornato per ogni ordine marketplace
- **Tabella `order_items`:** uno o più record per ogni riga prodotto dell'ordine
- I record esistenti vengono aggiornati (upsert), non duplicati

## Gestione errori
- Errori non gestiti esplicitamente: Make marca il bundle come errore e procede con il successivo (comportamento predefinito)
- Se `marketplace_id` non viene trovato (modulo 3 ritorna 406), l'ordine non viene inserito — il canale non è censito in Supabase
- Nessun resume point configurato
- Nessun filtro condizionale — tutti gli ordini vengono processati

## Note operative
- La finestra temporale `-2h` è calibrata sulla frequenza di esecuzione (ogni 15 min): c'è margine per gestire ritardi
- L'upsert è idempotente: eseguire lo scenario più volte sullo stesso ordine non crea duplicati
- Per un import storico completo, modificare temporaneamente la finestra in `-720h` (30 giorni), eseguire una volta, poi ripristinare a `-2h`
- Il campo `marketplace_order_id` corrisponde a `head.code` di SellRapido
- Il marketplace viene risolto tramite `head.channel_code` → `marketplaces.slug` (case-insensitive via `lower()`)

## Ultima modifica
Data: 10/03/2026
Modificato da: Claude Cowork
