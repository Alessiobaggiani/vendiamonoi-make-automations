# Changelog — FIC → Supabase | Fatture & Note Credito Ricevute

## [v1.0] - 10/03/2026
- Creazione iniziale scenario (ID Make: 8833326)
- Moduli: FIC listReceivedDocuments, HTTP alias lookup, HTTP upsert base, HTTP upsert matched (con filtro)
- Architettura a due step per gestire UUID null in Make IML
- Migrazione Supabase: `supplier_id` resa nullable, `matching_status` default cambiato a `'unmatched'`, indici su `fatture_in_cloud_id` e `lower(raw_supplier_name)`
- Testato con successo: 499 fatture importate, tutte `unmatched` (atteso — nessun alias configurato)
- Attivo in produzione: esecuzione giornaliera alle 06:00
