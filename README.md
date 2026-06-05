# CSE Helper — Altays

Panneau de suivi des heures de délégation pour les élu·e·s du CSE Doctolib.

Complète l'application [Altays](https://heures-de-delegation.altays-progiciels.com) avec des rapports et visualisations que l'outil officiel ne propose pas : suivi par trimestre, répartition par type d'absence, hors temps de travail, évolution sur la période.

## Installation

**Aucun serveur, aucune installation — un seul fichier HTML.**

1. Ouvrir `index.html` dans un navigateur
2. Glisser le bouton **📊 CSE Helper** vers la barre de favoris
3. Se connecter à Altays normalement
4. Cliquer le favori — le panneau s'ouvre à droite d'Altays

> Si la barre de favoris n'est pas visible : `Ctrl+Shift+B` (Windows/Linux) ou `⌘+Shift+B` (Mac).

## Fonctionnement

Le favori est un **bookmarklet** : un script JavaScript qui s'exécute dans le contexte de la page Altays (même origine → aucun problème CORS). Il injecte un panneau latéral sans modifier Altays.

```
index.html          →  glisser le favori dans la barre
                              ↓
Altays (onglet ouvert)  →  cliquer le favori
                              ↓
Bookmarklet :
  1. Intercepte les requêtes Altays pour capturer le JWT Bearer
  2. Appelle les API Altays (même origine, credentials: include)
  3. Affiche les rapports dans le panneau latéral
```

Le token est sauvegardé dans `sessionStorage` pour éviter de le ressaisir après un rechargement de page. Il est effacé à la fermeture du panneau.

## Onglets du panneau

### Heures
Résumé de la période sélectionnée (trimestre en cours par défaut).

- Total d'heures effectuées avec barre de répartition colorée (% par type)
- Décomposition en 3 catégories :
  - 🤝 **Réunions avec la direction** — réunions ordinaires, extraordinaires, commissions, négociation
  - 🔍 **Heures pour enquête** — accidents du travail, incidents répétés
  - 📋 **Heures de délégation** — toutes les autres absences
- **Hors temps de travail** — total des heures déclarées hors temps de travail, à récupérer ou à payer en heures supplémentaires (ETAM). Chargement à la demande via le bouton "Calculer".

### Détails
Analyse mensuelle (mois en cours par défaut), avec deux sections repliables.

**Heures de délégations**
- Histogramme empilé "Toutes commissions" (jour/semaine/mois selon la période)
- Une carte par mandat avec crédit, report, consommé, solde et histogramme individuel
- Les mandats sans événement sur la période sont repliés par défaut

**Tâches**
- Camembert de répartition des motifs d'événements

## Token Altays

Le JWT est capturé automatiquement via l'interception des requêtes réseau d'Altays. Si la capture automatique échoue :

1. Ouvrir Altays → `F12` → onglet **Réseau**
2. Naviguer dans Altays pour déclencher une requête
3. Cliquer une requête vers `heures-de-delegation.altays-progiciels.com`
4. Copier la valeur du header `Authorization` (le `Bearer eyJ…` complet)
5. Coller dans le champ du panneau CSE Helper

Le token est valide environ 16h. Altays en génère un nouveau à chaque connexion.

## API Altays utilisée

Base : `https://heures-de-delegation.altays-progiciels.com`

| Endpoint | Description |
|---|---|
| `GET /api/doctolib/mandats-courants` | Mandats actifs avec solde courant |
| `GET /api/doctolib/mandat/{uuid}?date=YYYY-M-01` | Détail mensuel d'un mandat (crédit, report, consommé, transferts) |
| `GET /api/doctolib/evenements` | Historique complet des bons de délégation |
| `GET /api/doctolib/bons-delegation/{uuid}` | Détail d'un bon (dont hors temps de travail) |

### Formule du solde
```
balance = nbHours + nbPostponedHours + nbCreditedTransferHours
        − nbDebitedHours − nbDebitedTransferHours
```

## Architecture technique

Tout le code tient dans un seul fichier `index.html` :

```
<style>          CSS de la page d'installation
<body>           UI d'installation + lien drag-to-install
<script type="text/plain" id="bm-src">
                 Source du bookmarklet (JS lisible)
<script>         Lit bm-src, minifie, génère le href javascript:
```

Le bookmarklet source (`#bm-src`) est minifié et URL-encodé par le script de la page. Modifier le source, recharger `index.html`, réinstaller le favori.

Aucune dépendance externe. Aucun build. Aucun serveur.

## Confidentialité

- Les données restent dans le navigateur (aucun tiers)
- Le token JWT est stocké uniquement en `sessionStorage` (effacé à la fermeture de l'onglet ou du panneau)
- Ce projet est indépendant et non affilié à Altays ni à Doctolib
