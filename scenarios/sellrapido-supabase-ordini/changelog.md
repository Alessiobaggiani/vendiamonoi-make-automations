# Changelog — SellRapido → Supabase | Ordini Marketplace

## [v1.6] - 12/03/2026
- **Rimosso filtro status dalla chiamata API SellRapido**: il modulo 1 ora chiama l'API senza filtrare per status, recuperando TUTTI gli ordini degli ultimi 30 giorni. Root cause: eBay Italy, eBay Germany e Metro Germany non comparivano perché i loro ordini potrebbero avere stati diversi dai 4 filtrati in precedenza (`standby/accepted/sent/cancelled`).
- **Mapping status esteso — italiano + inglese**: modulo 4 ora gestisce 11 casi. Aggiunti: `accettato→confirmed`, `spedito→shipped`, `delivered→delivered`, `consegnato→delivered`, `annullato→cancelled`, `refunded→refunded`, `rimborsato→refunded`. Il frontend SellRapido mostra stati in italiano; l'API potrebbe restituire entrambe le versioni a seconda del marketplace.
- **Eliminata tabella diagnostica** `sellrapido_channel_codes_log` da Supabase.
- **Disattivato scenario diagnostico** 8845166.

## [v1.5] - 12/03/2026
- **Fix case-insensitive marketplace lookup**: modulo 3 ora usa `eq.{{lower(2.head.channel_code)}}` invece di `eq.{{2.head.channel_code}}`. Root cause: SellRapido invia i channel_code in PascalCase/UPPERCASE (`CarrefourES`, `ePRICEIT`, `IT_MAIN`, `LeroyMerlinIT`, `RueduCommerce`, `Shopify`, `Kaufland Germany`), ma Supabase `sellrapido_code` ha tutti i valori in lowercase. Con `eq.` exact match, solo `mediaworld.it` matchava per coincidenza. Con `lower()` tutti i 10 canali attivi ora matchano.
- **Fix Supabase `eprice_it` → `epriceit`**: `lower('ePRICEIT') = 'epriceit'`, ma il DB aveva `eprice_it` con underscore — rinominato.
- **Aggiunto marketplace IBS** (`sellrapido_code = 'ibs'`): IBS.it è un marketplace italiano per libri, presente tra gli ordini SellRapido.
- **Aggiunto marketplace Italy (SellRapido)** (`sellrapido_code = 'italy'`): canale generico SellRapido non ancora classificato — aggiunto per non perdere ordini, da rinominare quando identificato.
- **Diagnostico completato e risolto**: tabella `sellrapido_channel_codes_log` (creata in sessione precedente) ha confermato il problema — 9/10 channel_codes non matchavano. Può essere eliminata.

## [v1.4] - 12/03/2026
- **Fix definitivo shipping_address**: `parseJSON()` non esiste in Make IML — lo scenario crashava con `DataError: Function 'parseJSON' not found`. Fix: `shipping_address` ora costruita via string concat dentro `{{...}}` — es. `{{"{\"street\":\"" & addr & "\",...}"}}`. La `{` che apre l'oggetto JSON è DENTRO l'espressione IML come stringa letterale, non esposta al parser legacy. Verificato: `shipping_address` è ora JSONB valido in Supabase, `shipping_address_formatted` generata correttamente. Scenario esegue 426 operazioni — SUCCESS.

## [v1.3] - 12/03/2026
- **Fix NaN warning (parziale)**: `shipping_address` tentato con `{{parseJSON(...)}}` — non funzionava (funzione inesistente in Make). Superato da v1.4.
- **Fix status ordini sempre "pending"**: il mapping dello status SellRapido era errato — controllava `shipped` e `delivered` che non esistono nell'API SellRapido. Il valore reale `sent` cadeva sempre nel default `"pending"`. Fix: mapping corretto `sent→shipped`, `cancelled→cancelled`. Modulo 1 aggiornato: status richiesti ora `[standby, accepted, sent, cancelled]`.
- **Fix marketplace lookup**: modulo 3 ora usa colonna `sellrapido_code` (dedicata) invece di `slug=lower(channel_code)`.
- **Fix status ordini sempre "pending"**: il mapping dello status SellRapido era errato — controllava `shipped` e `delivered` che non esistono nell'API SellRapido. Il valore reale `sent` cadeva sempre nel default `"pending"`. Fix: mapping corretto `sent→shipped`, `cancelled→cancelled`. Modulo 1 aggiornato: status richiesti ora `[standby, accepted, sent, cancelled]`.
- **Fix marketplace lookup**: modulo 3 ora usa colonna `sellrapido_code` (dedicata) invece di `slug=lower(channel_code)`. Prerequisito: popolare `marketplaces.sellrapido_code` con i codici canale SellRapido.

## [v1.2] - 12/03/2026
- Fix critico: `startDate` cambiato da `addHours(now; -2)` a `addDays(now; -30)` — la finestra di 2h impediva aggiornamenti di stato su ordini esistenti; ora tutti gli ordini degli ultimi 30 giorni vengono ri-processati ad ogni esecuzione
- Migliorata fallback `order_date`: ora usa `ifempty(date_order; ifempty(create_date; now))` — se `date_order` è null, tenta `create_date` prima di usare `now()`
- Rimosso `raw_data` dall'upsert: la funzione `toJSON()` di Make IML non è disponibile nel contesto raw body string — campo rimosso per evitare errore `DataError: Function 'toJSON' not found`
- Rimosso `raw_data` anche dal parametro `columns`
- Verificato: scenario esegue 95 operazioni su ~30 ordini, status 1 (SUCCESS)

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
