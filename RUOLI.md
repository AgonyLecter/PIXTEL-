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

Il **primo account che fa login su `furio.html`** (quando nel database non esiste ancora `staff/owner`) diventa subito **Creatore** e Admin. Da quel momento nessun altro può prendere quel posto.

⚠️ Fai il primo login su Furio **subito dopo aver pubblicato le regole qui sotto**: finché `staff/owner` è vuoto, chiunque abbia un account potrebbe arrivare prima di te.

L'account va creato prima dal gioco (`index.html`) come un qualsiasi giocatore, poi si entra su Furio con la stessa email e password.

## Dati nel database

```
staff/owner   = uid del Creatore
roles/{uid}   = "admin" | "helper" | "vip"     (assente = Utente)
admins/{uid}  = true  per Creatore e Admin      (già usato dalle regole esistenti)
```

## Regole da aggiungere (Firebase Console → Realtime Database → Regole)

Aggiungi questi nodi dentro `"rules": { ... }` accanto a quelli che hai già. Se esiste già un blocco `"admins"`, sostituiscilo con questo.

```json
"staff": {
  ".read": "auth != null",
  "owner": {
    ".write": "auth != null && !data.exists() && newData.val() === auth.uid"
  }
},
"roles": {
  ".read": true,
  "$uid": {
    ".validate": "newData.val() === 'admin' || newData.val() === 'helper' || newData.val() === 'vip'",
    ".write": "auth != null && $uid !== root.child('staff/owner').val() && (auth.uid === root.child('staff/owner').val() || (root.child('admins').child(auth.uid).exists() && data.val() !== 'admin' && newData.val() !== 'admin'))"
  }
},
"admins": {
  ".read": "auth != null",
  "$uid": {
    ".write": "auth != null && ((auth.uid === root.child('staff/owner').val() && $uid !== auth.uid) || ($uid === auth.uid && !root.child('staff/owner').exists() && newData.parent().parent().child('staff/owner').val() === auth.uid))"
  }
}
```

### Permessi degli Helper

Nelle regole che hai già, dove compare la condizione da admin
`root.child('admins').child(auth.uid).exists()` sui nodi **`users`** (lettura), **`reports`** (lettura e cancellazione) e **`audit`** (scrittura), aggiungi in OR:

```
|| root.child('roles').child(auth.uid).val() === 'helper'
```

Esempio: `".read": "root.child('admins').child(auth.uid).exists() || root.child('roles').child(auth.uid).val() === 'helper'"`.

Non darla su `bans`, `wallets`, `rooms`, ecc.: gli Helper non devono bannare né toccare monete o stanze.
