# Ruoli PIXTEL — gestiti da Furio

| Ruolo | Accesso a Furio | Cosa può fare |
|---|---|---|
| **Creatore** | sì, tutto | Tutto. Nomina e rimuove gli Admin. Non può essere toccato da nessuno. |
| **Admin** | sì, tutto | Ban, monete, stanze, registro. Nomina Helper e Vip. Non può toccare altri Admin né il Creatore. |
| **Helper** | parziale | Vede Panoramica, Iscritti, Staff e Segnalazioni; può archiviare le segnalazioni. |
| **Vip** | no | Badge VIP nel gioco. |
| **Utente** | no | Giocatore normale (nessun ruolo). |

I badge (CREATORE / ADMIN / HELPER / VIP) si vedono nel gioco: nella scheda del giocatore, nel nome al passaggio del mouse e nei fumetti della chat.

## Il primo accesso diventa Creatore

Il **primo account registrato su `furio.html`** diventa subito **Creatore** e Admin. Dopo, il pulsante di registrazione sparisce e Furio mostra solo "Accedi".

⚠️ Fai il primo login su Furio **subito dopo aver pubblicato le regole qui sotto**: finché `staff/owner` è vuoto, chiunque abbia un account potrebbe arrivare prima di te.


## Dati nel database

```
staff/owner   = uid del Creatore
roles/{uid}   = "admin" | "helper" | "vip"     (assente = Utente)
admins/{uid}  = true  per Creatore e Admin      (già usato dalle regole esistenti)
```

## Regole del database

Le regole complete sono in **`database.rules.json`**. Copia tutto il contenuto del file in
Firebase Console → Realtime Database → **Regole**, sostituendo quello che c'è, e premi **Pubblica**.

Proteggono tutto il gioco:

- **Monete**: il wallet nasce a 100 monete e solo gli Admin possono cambiarlo. Nessuno legge il wallet degli altri.
- **Profili**: ognuno modifica solo il proprio look e motto. Il nome non si cambia e non si può duplicare.
- **Stanze**: massimo 3 per giocatore, una ogni 10 secondi. Si possono solo creare ed eliminare le proprie. La Hall la tocca solo lo staff.
- **Movimento**: si entra dalla porta e si avanza di una casella alla volta, non più di un passo ogni 200 ms. Niente teletrasporto e niente spostare gli altri.
- **Chat**: un messaggio al secondo, massimo 120 caratteri, sempre col proprio nome. I messaggi più vecchi di 10 minuti si possono cancellare.
- **Segnalazioni**: una ogni 10 secondi. Le leggono solo Admin e Helper.
- **Ban**: li danno solo gli Admin. Il Creatore non può essere bannato e gli Admin li banna solo il Creatore. Un bannato non può più scrivere niente.
- **Trappola** (`economy`): chi chiama `pixtel.staff.grantCoins` dalla console resta congelato finché un Admin non lo perdona. Furio imposta da solo `adminConfig/trap` al primo accesso del Creatore.
- **Registro**: lo leggono solo gli Admin e non si può cancellare.
- **Ruoli**: vedi sopra. Gli Helper leggono iscritti e segnalazioni e archiviano, niente di più.
