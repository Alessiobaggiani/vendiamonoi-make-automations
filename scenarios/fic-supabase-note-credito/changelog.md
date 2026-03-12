# Changelog — FIC → Supabase | Note di Credito Ricevute
**Scenario Make ID:** 8844531

---

## [v1.0] - 12/03/2026
- Creazione scenario nuovo (separato da fatture per efficienza)
- Tipo documento: `passive_credit_note`
- Stessa struttura di scenario 8833326 (fatture)
- `invoice_type` hardcoded a `credit_note` (sempre nota di credito)
- `q` date filter: ultimi 18 mesi (stesso approccio di v1.1 fatture)
- Scenario attivo, gira ogni giorno alle 06:10 (10 min dopo le fatture)
