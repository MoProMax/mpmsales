# MPM Sales — Mo Pro Max Revenue Platform

Sales-dashboard voor mopromax.com: leads inbox, niet-lineaire sales/delivery-funnel (34 fases), deals-lijst, opvolg-reminders en activiteitenlog. Eén statisch `index.html`-bestand; alle data staat in Firebase Firestore (project `mpms-e59f8`, database `mpms1`).

**Live:** https://mpmsales.onrender.com/ (Render, auto-deploy vanaf de `main`-branch van deze repo).

## Beveiliging — belangrijk

De data wordt beschermd door **Firestore security rules**, niet door de login-pagina zelf. Zet daarom de rules uit [`firestore.rules`](firestore.rules) in de Firebase console:

1. Ga naar [Firebase console](https://console.firebase.google.com/) → project **mpms-e59f8**
2. Firestore Database → selecteer database **mpms1** → tabblad **Regels**
3. Plak de inhoud van `firestore.rules` en klik **Publiceren**

Zonder deze rules kan iedere (eventueel anoniem) ingelogde Firebase-gebruiker bij de data — de API-key in de pagina is publiek en dat hoort zo, maar de rules moeten de toegang beperken tot het eigenaar-account.

### Wat de 2FA-stap wel en niet doet

De Google Authenticator-stap in het login-scherm is een **drempel in de browser**, geen server-side beveiliging: het TOTP-geheim staat in Firestore en de controle gebeurt client-side. Iemand die het wachtwoord kent, kan met de Firestore REST API om de 2FA-stap heen. De echte bescherming is dus: **een sterk, uniek wachtwoord** + de security rules hierboven. Wil je échte 2FA, dan is een backend (bijv. Cloud Function) nodig die de TOTP-code verifieert vóór er een token wordt uitgegeven.

## Structuur in Firestore

| Collectie | Inhoud |
|---|---|
| `leadChunks/chunk_N` | Inbox-leads in blokken van max. 1500 |
| `deals/{id}` | Deals met fase, waarde, historie, activiteiten, opvolgdatum |
| `requests/{id}` | Log van onderzoeksaanvragen |
| `meta/status` | Sync-metadata |
| `authConfig/mfa` | TOTP-geheim voor de 2FA-stap |

## Deployen

Push naar `main` → Render deployt automatisch. Geen build-stap; het is één statisch HTML-bestand.
