# Changelog — FIC → Supabase | Fatture Ricevute
**Scenario Make ID:** 8833326

---

## [v1.1] - 12/03/2026
### Fix critici
- **Root cause**: FIC restituisce documenti in ordine ascendente per data. Con `limit: 500`, lo scenario recuperava sempre i primi 500 documenti (2024-03-15 → 2025-01-24). I 14 mesi successivi non venivano mai raggiunti.
- **Fix `q` date filter**: Aggiunto parametro `q: "date:>=:{{addMonths(now; -18)}}"`. Se supportato dal connettore FIC Make, filtra direttamente a livello server → solo ultimi 18 mesi.
- **Fix `limit`**: Aumentato da 500 a 1000 per la fase di backfill.
- **Fix `payment_status` null-safety**: Protetto con `ifempty(first(1.payments_list).status; "")`.
- **Scenario attivato**: Era inattivo — ora gira ogni giorno alle 06:00.

### Struttura moduli
1. FIC `listReceivedDocuments` → `expense`, limit: 1000, q filter 18 mesi
2. GET `supplier_aliases` → cerca fornitore per nome (ilike)
3. POST `supplier_invoices` → upsert on_conflict `fatture_in_cloud_id`
4. POST `supplier_invoices` → update `supplier_id` + `matching_status: matched` (solo se fornitore trovato — tabella vuota per ora)

---

## [v1.0] - 10/03/2026
- Creazione scenario iniziale
- Import 499 fatture da 2024-03-15 a 2025-01-24
- Solo tipo `expense`, nessun filtro data, scenario rimasto inattivo
