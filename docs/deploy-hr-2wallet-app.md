# Deploy identico su **hr.2wallet.app** (ambiente HR)

Il codice è già multi-tenant e usa `CUSTOM_DOMAIN` per URL pass, landing, callback wallet, ecc. Non serve un secondo repository: duplichi il progetto (o il servizio) su Railway e punti un nuovo hostname.

> Nota DNS: i domini sono case-insensitive; usa **`hr.2wallet.app`** nel DNS e in `CUSTOM_DOMAIN` (senza `https://`).

## 1. DNS

Nel pannello del dominio **2wallet.app** crea un record per l’istenza HR, ad esempio:

- **Tipo:** `CNAME`
- **Nome / host:** `hr`
- **Valore / target:** il hostname pubblico che Railway mostra per il **nuovo** servizio (es. `xxx.up.railway.app`), oppure il target che ti indica Railway per “Custom domain”.

Attendi propagazione (spesso pochi minuti, a volte di più).

## 2. Railway — secondo ambiente

**Opzione consigliata — nuovo progetto “clone”:**

1. Railway → **New Project** → **Deploy from GitHub** → stesso repo `ads.2wallet.app`, branch `main` (stesso codice dell’istanza Ads).
2. Aggiungi **PostgreSQL** dedicato (database **separato**: dati HR indipendenti da Ads).
3. Sul servizio **web**, collega le variabili dal Postgres (`DATABASE_URL` / riferimento).
4. **Variables** del servizio web — copia dall’istanza Ads le stesse chiavi **tranne** quelle sotto; adatta dove serve:

| Variabile | Valore HR |
|-----------|-----------|
| `CUSTOM_DOMAIN` | `hr.2wallet.app` |
| `DATABASE_URL` | Solo dal Postgres **di questo** progetto |
| `JWT_SECRET` | **Nuovo** valore casuale (non riusare quello di Ads) |
| `PORT` | Lasciare default Railway (`3000` ok se non sovrascritto) |
| Certificati Apple, `PASS_TYPE_IDENTIFIER`, `TEAM_IDENTIFIER`, Resend, Google/Samsung, ecc. | Stessi valori **solo se** HR condivide lo stesso programma tecnico; in alternativa credenziali dedicate per HR |

5. **Settings → Networking / Domains** → aggiungi **Custom Domain** `hr.2wallet.app` e completa il certificato TLS come da wizard Railway.

6. Redeploy.

**Opzione stesso progetto Railway:** due servizi web dallo stesso repo + due Postgres + due set di variabili; più facile confondere i riferimenti — per “copia identica” isolata è più chiaro un **secondo progetto**.

## 3. Primo accesso dashboard

Su DB vuoto lo seed crea (se non imposti `BOOTSTRAP_ADMIN_*`):

- `admin@ads2wallet.com` / `Ads2Wallet2026!`

Per un admin dedicato HR vedi `.env.example` (`BOOTSTRAP_ADMIN_EMAIL` / `BOOTSTRAP_ADMIN_PASSWORD`).

## 4. Apple Wallet / Google / Samsung

- **`webServiceURL`** e link pubblici useranno `https://hr.2wallet.app` se `CUSTOM_DOMAIN=hr.2wallet.app`.
- Se HR è un **prodotto Apple distinto**, usa un **`PASS_TYPE_IDENTIFIER`** diverso e certificati coerenti; se è solo un secondo front-end sullo stesso tipo pass, allinea la doc interna al team Apple.

## 5. Email (Resend)

Verifica dominio / mittente (`FROM_EMAIL`) per invii da `hr.2wallet.app` se cambi brand o dominio mittente.

---

Riepilogo: **stesso GitHub**, **nuovo Railway** (o servizio), **nuovo Postgres**, **`CUSTOM_DOMAIN=hr.2wallet.app`**, **nuovo `JWT_SECRET`**, DNS **hr** → Railway.
