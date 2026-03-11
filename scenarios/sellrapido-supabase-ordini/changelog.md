# Changelog — SellRapido → Supabase | Ordini Marketplace

## [v1.0] - 10/03/2026
- Creazione iniziale scenario (ID Make: 8831616)
- Moduli: HTTP SellRapido, Iterator ordini, HTTP marketplace lookup, HTTP upsert orders, SetVariable orderId, Iterator rows, HTTP upsert order_items
- Finestra temporale: -2h (produzione), -720h usato per import storico iniziale
- Testato su 29 ordini storici — import completato con successo
- Tabelle Supabase popolate: `orders` (29 record), `order_items` (30 record)
