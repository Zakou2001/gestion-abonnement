# CG-TECH v4 — Séquence d'exécution

## 1. Ordre de chargement (dans `index.html`)

1. **`<head>`**
   1. Polices Google Fonts
   2. `styles.css` (styles globaux, thème sombre/clair, RTL)
   3. **Script anti-flash inline** (doit rester en ligne, s'exécute avant tout rendu) :
      lit `localStorage.smv4_theme_mode` / `smv4_theme_manual` et pose l'attribut
      `data-theme` sur `<html>` **avant** que le CSS ne peigne la page — évite un flash de thème.
2. **`<body>`** : markup de l'écran de login (`#ls`) + squelette de l'app (`#app`, masqué par défaut) + toutes les pages/modales (masquées, activées via classes).
3. **`app.js`** (chargé en fin de `<body>`, donc après tout le DOM) :
   - Exécution immédiate de tout le code de premier niveau (voir §2)
   - Définition des fonctions (login, rendu des pages, CRUD, etc.) — ne s'exécutent qu'à l'appel.

## 2. Ce qui s'exécute immédiatement au chargement de `app.js`

```
loadDB()                     // ligne ~327 : charge DB depuis localStorage('smv4') ou données de démo
window.addEventListener('storage', …)   // sync multi-onglets
(function restoreSession(){ … })()      // tente de restaurer une session existante
```

`restoreSession()` :
- Lit `localStorage.smv4_session`
- Si un utilisateur correspondant existe dans `DB.users` → connexion automatique :
  masque l'écran de login, affiche `#app`, appelle `setupApp()`, restaure la dernière page visitée (`smv4_page_<id>`).
- Sinon → l'écran de login (`#ls`) reste affiché, en attente de `doLogin()`.

## 3. Parcours utilisateur (cas normal, sans session existante)

```
Utilisateur saisit identifiants → doLogin()
   ├─ vérifie les identifiants contre DB.users
   ├─ CU = utilisateur trouvé
   ├─ setupApp()
   │    ├─ configure la sidebar selon le rôle (can(), permissions)
   │    ├─ syncSidebarUser(), updateBadges(), updateSidebarSolde()
   │    ├─ démarre l'auto-refresh (startAutoRefresh())
   └─ goto('dashboard')
        └─ renderPage('dashboard') → RENDERS['dashboard'] = renderDashboard()
```

## 4. Navigation entre pages

- `goto(pg)` change la page active, met à jour l'URL/état interne, puis appelle `renderPage(pg)`.
- `renderPage(pg)` regarde dans la table `RENDERS` (fin de app.js, section HELPERS) :

  | Page | Fonction de rendu |
  |---|---|
  | dashboard | `renderDashboard` |
  | abonnements | `renderSubs` |
  | alertes | `renderAlertes` |
  | facturation | `renderFacturation` |
  | rh / comptabilite / depenses / historique | `renderRH` / `renderCompta` / `renderDepenses` / `renderHistorique` |
  | plans / users / parametres / logistique / agences / caisse | fonctions dédiées |
  | dashboard-compta / dashboard-rh | vues dashboard spécialisées |

  *(liste complète dans `app.js`, table `RENDERS`)*

## 5. Cycle CRUD standard (toute création/modification/suppression)

Décrit explicitement dans `app.js` (section "SYSTÈME DE VALIDATION & SYNCHRONISATION GLOBAL") :

```
1. valider(data, regles)     // validation des données
2. crudCreate / crudUpdate / crudDelete
3. saveDB()                  // persistance dans localStorage
4. syncApres(entite, action) // rafraîchit l'affichage concerné
5. toast(msg)                // notification utilisateur
6. journaliser(action, …)    // journal des modifications (audit trail)
```

## 6. Synchronisation multi-onglets

- Toute écriture dans `localStorage['smv4']` (la DB) déclenche l'événement `storage` dans les autres onglets ouverts → `loadDB()` y est rappelé pour se resynchroniser.
- Un canal séparé `smv4_perm_update` permet de pousser un changement de permissions vers un utilisateur précis en temps réel (déconnexion/reconnexion douce via `setupApp()`).

## 7. Impression / Aperçu

- `pvXxx()` (préfixe "print view") pilote la modale `#m-print-preview` : format papier, orientation, zoom, police, marges, interligne — génère le contenu dans `#pv-content` puis appelle `window.print()` via `pvPrint()`.

## 8. Points d'entrée à connaître pour le débogage

| Symptôme | Point de départ à inspecter |
|---|---|
| Rien ne s'affiche au chargement | Script anti-flash + `restoreSession()` |
| Connexion échoue | `doLogin()` |
| Une page ne se met pas à jour après une action | `syncApres()` / table `RENDERS` |
| Données perdues après rafraîchissement | `saveDB()` / `loadDB()` / clé `localStorage['smv4']` |
| Comportement différent entre onglets | listener `window.addEventListener('storage', …)` |
| Traduction manquante | section i18n (`app.js`, lignes ~7506–8251) |
