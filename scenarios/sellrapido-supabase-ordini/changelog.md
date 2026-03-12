# Changelog — SellRapido → Supabase | Ordini Marketplace

## [v1.1] - 12/03/2026
- Fix `order_date`: ora usa `date_order` (data reale ordine cliente) invece di `create_date` (data inserimento SellRapido)
- Fix `marketplace_status`: era hardcoded `"pending"`, ora mappato dinamicamente da SellRapido (`standby→pending`, `accepted→confirmed`, `shipped→shipped`, `delivered→delivered`)
- Aggiunto `shipping_value`: importa `shipping_price` dal campo `head` di SellRapido
- Aggiunto `marketplace_fee`: importa commissione marketplace dal campo `head`
- Aggiunto `raw_data`: salva JSON completo del campo `head` per tracciabilità
- Fix modulo 3 marketplace lookup: rimosso header `Accept: vnd.pgrst.object+json` che causava errore 406 quando il marketplace non veniva trovato → ora restituisce array, ordini per marketplace sconosciuti vengono saltati senza errore
- Aggiunto filtro su modulo 4: skip ordine se marketplace non trovato in Supabase
- Aggiunto parametro `columns` su upsert ordini: protegge `internal_status` da sovrascrittura su aggiornamenti successivi
- Supabase: aggiunta colonna generata `shipping_address_formatted` (formato testo: "Via X, CAP Città, PAESE")

## [v1.0] - 10/03/2026
- Creazione iniziale scenario (ID Make: 8831616)
- Moduli: HTTP SellRapido, Iterator ordini, HTTP marketplace lookup, HTTP upsert orders, SetVariable orderId, Iterator rows, HTTP upsert order_items
- Finestra temporale: -2h (produzione), -720h usato per import storico iniziale
- Testato su 29 ordini storici — import completato con successo
- Tabelle Supabase popolate: `orders` (29 record), `order_items` (30 record)
