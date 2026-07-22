# MPM Sales — Mo Pro Max Revenue Platform

Sales-dashboard voor mopromax.com: leads inbox, niet-lineaire sales/delivery-funnel (34 fases), deals-lijst, opvolg-reminders en activiteitenlog. Eén statisch `index.html`-bestand; alle data staat in Firebase Firestore (project `mpms-e59f8`, database `mpms1`).

**Live:** https://mpmsales.onrender.com/ (Render, auto-deploy vanaf de `main`-branch van deze repo).

## Beveiliging — belangrijk

Twee lagen, beide **server-side afgedwongen door Google** (niet door de pagina zelf):

1. **Firestore security rules** beperken alle data tot het eigenaar-account.
2. **Echte tweestapsverificatie (TOTP-MFA)** via Firebase Authentication with Identity Platform: Google geeft pas een geldig token uit nádat de authenticator-code klopt. De pagina kan er niet omheen, want zonder dat token weigert Firestore elke lees- en schrijfactie.

### Eenmalige setup (in de Firebase / Google Cloud console)

**a) Firestore-regels publiceren**
1. [Firebase console](https://console.firebase.google.com/) → project **mpms-e59f8**
2. Firestore Database → database **mpms1** → tabblad **Regels**
3. Plak [`firestore.rules`](firestore.rules) en klik **Publiceren**

**b) TOTP-MFA inschakelen**
1. Firebase console → **Authentication** → **Sign-in method** → onderaan **Advanced** / **SMS multi-factor authentication** → kies **TOTP (authenticator app)** en zet aan.
   (Dit vereist **Firebase Authentication with Identity Platform**; upgraden kan de console je vragen — de gratis laag dekt ruim voldoende gebruikers.)
2. Log daarna in op de site met e-mail + wachtwoord. Omdat er nog geen tweede factor gekoppeld is, toont de pagina eenmalig een **QR-code** (lokaal getekend, het geheim verlaat de browser niet). Scan die met Google Authenticator (*"QR-code scannen"*) — of gebruik de uitklapbare **setup-sleutel** als handmatige terugval — en bevestig met de 6-cijferige code.
3. Vanaf dat moment vraagt élke login om de code, door Google gecontroleerd.

Zolang stap (b) nog niet gedaan is, meldt het inlogscherm netjes dat TOTP-MFA nog aangezet moet worden.

### Waarom dit nu écht 2FA is

De TOTP-code wordt door **Google's Identity Platform** geverifieerd, niet in de browser. Iemand met alleen het wachtwoord krijgt geen bruikbaar token en komt dus niet bij de data — ook niet via de Firestore REST API. De publieke API-key in de pagina is prima; die identificeert alleen het project en verleent op zichzelf geen toegang. Het oude, client-side TOTP-geheim (`authConfig/mfa` in Firestore) wordt niet meer gebruikt en mag verwijderd worden.

## Structuur in Firestore

| Collectie | Inhoud |
|---|---|
| `leadChunks/chunk_N` | Inbox-leads in blokken van max. 1500 |
| `deals/{id}` | Deals met fase, waarde, historie, activiteiten, opvolgdatum |
| `requests/{id}` | Log van onderzoeksaanvragen |
| `meta/status` | Sync-metadata |
| `authConfig/mfa` | *(verouderd — oud client-side TOTP-geheim, niet meer in gebruik)* |

## Deployen

Push naar `main` → Render deployt automatisch. Geen build-stap; het is één statisch HTML-bestand.
