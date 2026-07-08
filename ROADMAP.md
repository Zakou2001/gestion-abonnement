# Roadmap CG-TECH v4 — Plan de séquencement

Ce document découpe le travail sur l'application en phases ordonnées, à partir
du fichier unique `Gestion_d_Abonnement_V4_01_Final.html` (12 405 lignes)
maintenant séparé en 3 fichiers :

- `index.html` — structure et markup (≈2 630 lignes)
- `styles.css` — feuille de styles (≈451 lignes)
- `app.js` — logique applicative (≈9 337 lignes)

Tous trois sont bit-à-bit équivalents à l'original (aucune ligne perdue ni modifiée).

---

## Phase 0 — Séparation des couches (✅ terminée)
- Extraction CSS → `styles.css`
- Extraction JS → `app.js`
- `index.html` référence les deux via `<link>` et `<script src>`
- Script anti-flash de thème conservé inline dans `<head>` (doit s'exécuter avant le premier rendu, donc **ne pas** l'externaliser)

## Phase 1 — Modularisation du JavaScript
`app.js` est un seul bloc procédural de ~9 300 lignes avec des sections marquées par des commentaires `══════`. Prochaine étape : le découper en modules thématiques (sans casser les dépendances globales `DB`, `CU`, `RENDERS`) :

| Module proposé | Contenu | Lignes approx. (dans app.js) |
|---|---|---|
| `core/data.js` | Rôles, init DB, `loadDB`/`saveDB`, solde | 1–367 |
| `core/sync.js` | Validation & synchronisation CRUD globale | 368–612 |
| `core/auth.js` | Login/logout, permissions, visibilité facturation | 573–905 |
| `ui/dashboard.js` | Dashboard + variantes par rôle | 906–1242 |
| `modules/abonnements.js` | Abonnements, sessions, requêtes, MAD, alertes | 1243–2105 |
| `modules/rh-compta.js` | RH, comptabilité, dépenses, historique | 2105–2439 |
| `modules/plans-users.js` | Plans, utilisateurs, agences | 2439–3310 |
| `modules/logistique.js` | Parc véhicules, missions, carburant | 3311–3578 |
| `modules/impression.js` | Impression, aperçu avant impression | 3578–4126, 6170–6733 |
| `modules/facturation.js` | Facturation, ventes, clients, articles | 4126–6169, 6734–7505 |
| `core/i18n.js` | Traduction FR/EN/AR/HA | 7506–8251 |
| `modules/ventes-sav.js` | Nouvelles fonctions ventes & SAV, brouillons | 8252–8975 |
| `core/params.js` | Paramètres, thème, restauration de session | 8976–9337 |

**Recommandation :** procéder module par module, en testant après chaque extraction (le couplage via variables globales `DB`/`CU`/`RENDERS` rend l'ordre d'inclusion des `<script>` important tant que les modules ne sont pas transformés en vrais modules ES avec `import`/`export`).

## Phase 2 — Modularisation ES (optionnelle, moyen terme)
- Convertir les fichiers de la Phase 1 en modules ES natifs (`type="module"`)
- Remplacer les variables globales (`DB`, `CU`) par un état centralisé exporté
- Permet le bundling futur (Vite/esbuild) et le tree-shaking

## Phase 3 — Séparation des données
- `DB` est actuellement injecté en dur dans `app.js` (données de démo). L'isoler dans `data/seed.js` ou un endpoint simulé faciliterait le passage à un vrai backend plus tard.

## Phase 4 — CSS
- `styles.css` est déjà un fichier unique cohérent ; découpage en partiels (`base.css`, `sidebar.css`, `modals.css`, `print.css`) possible si le fichier continue de grossir, mais non prioritaire vu sa taille actuelle (~450 lignes).

## Phase 5 — Qualité & outillage
- Ajouter un linter (ESLint) une fois le JS modularisé
- Ajouter des tests légers sur les fonctions pures (`fmt`, `dL`, `valider`, calculs de solde)
- Mettre en place un bundler pour la mise en prod (minification, cache-busting du `<!-- Build: ... -->`)

---

**Ordre d'exécution recommandé :** Phase 0 (fait) → Phase 1 → Phase 3 → Phase 2 → Phase 4 → Phase 5.
Chaque phase est indépendante et peut être livrée séparément sans casser l'application.
