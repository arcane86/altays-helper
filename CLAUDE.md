# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Contexte

Application web sans backend destinée aux élu·e·s du CSE. Fichier unique `index.html` — à ouvrir directement dans un navigateur, aucun serveur requis.

**Architecture** : page d'installation qui génère un bookmarklet. Le bookmarklet s'exécute sur la page Altays (même origine → pas de CORS) et injecte un panneau latéral avec les rapports.

**Langue** : français avec écriture inclusive (iel, élu·e, etc.).

## Flux

```
index.html  →  l'élu·e glisse le favori dans sa barre
                     ↓
Altays (page ouverte)  →  l'élu·e clique le favori
                     ↓
Bookmarklet s'exécute :
  1. Cherche le Bearer JWT dans localStorage/sessionStorage
  2. Sinon : intercepte window.fetch pour capturer le token au vol
  3. Appelle GET /api/doctolib/evenements (même origine, pas de CORS)
  4. Affiche les données dans le panneau
```

## Structure de index.html

```
<style>           — CSS de la page d'installation
<body>            — UI d'installation + lien drag-to-install
<script type="text/plain" id="bm-src">
                  — SOURCE du bookmarklet (lue par le script suivant)
<script>          — lit bm-src, minifie, injecte dans href du lien
```

Le bookmarklet source est dans `<script type="text/plain" id="bm-src">`. Il est minifié et URL-encodé par le script de la page avant d'être assigné à `#bm-link`.

## API Altays

Base : `https://heures-de-delegation.altays-progiciels.com`  
Auth : `Authorization: Bearer eyJ…` (JWT, ~16h de validité)

| Endpoint | Description |
|---|---|
| `GET /api/doctolib/mandats-courants` | Tous les mandats actifs avec solde courant (`balance`, `nbDebitedHours`) |
| `GET /api/doctolib/mandat/{uuid}?date=YYYY-M-01` | Détail mensuel : `nbHours` (crédit), `nbPostponedHours` (report), `nbDebitedHours` (consommé), `nbCreditedTransferHours`, `nbDebitedTransferHours`, `balance`, `vouchers[]` |
| `GET /api/doctolib/evenements` | Tous les bons de délégation (historique complet) |
| `GET /api/doctolib/bons-delegation/{uuid}` | Détail d'un bon |
| `GET /api/doctolib/regularisations` | Régularisations |
| `GET /api/doctolib/mandats` | Liste complète des mandats |
| `GET /api/doctolib/absences` | Types d'absences disponibles |

### Formule du solde
`balance = nbHours + nbPostponedHours + nbCreditedTransferHours - nbDebitedHours - nbDebitedTransferHours`

## Token

Capturé automatiquement depuis `localStorage`/`sessionStorage` (cherche toute valeur commençant par `eyJ`), ou via interception de `window.fetch`. Fallback manuel si non trouvé. Si l'utilisateur·ice colle la valeur sans `Bearer `, le préfixe est ajouté automatiquement.
