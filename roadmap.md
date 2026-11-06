# Roadmap : projet BDA TechDistrib Europe

Option retenue : **csv → tables → contraintes**, réalisée dans **MySQL** (phpMyAdmin ou Workbench), avec une appli **Flask** pour le CRUD et la visualisation.

---

## Prérequis (à faire ensemble, environ 1 h)

- [ ] Installer MySQL 8 (ou XAMPP / WAMP / MAMP, qui inclut phpMyAdmin)
- [ ] Installer Python 3.10+, Git, VS Code
- [ ] Créer un compte GitHub et un dépôt partagé ; inviter l'autre personne (Settings → Collaborators)
- [ ] Créer la structure du dépôt :

```
projet-techdistrib/
├── README.md              # installation et lancement (base + appli)
├── .gitignore             # exclut config.py et les mots de passe
├── data/                  # les 9 CSV
├── sql/
│   ├── 01_schema.sql      # CREATE DATABASE + CREATE TABLE (clés primaires seulement)
│   ├── 02_import.sql      # LOAD DATA (ordre des imports)
│   ├── 03_contraintes.sql # FK, UNIQUE, CHECK, NOT NULL, DEFAULT
│   ├── 04_tests_contraintes.sql
│   ├── 05_vues.sql
│   ├── 06_droits.sql
│   ├── 07_concurrence.sql # scénarios à exécuter en 2 sessions
│   ├── 08_triggers.sql
│   ├── 09_procedures.sql
│   └── 10_historique.sql
├── app/                   # Flask : app.py, templates/, static/
├── rapport/
└── docs/                  # captures d'écran, schéma final
```

- [ ] Convention de travail : une personne par partie (ex. Ulysse : parties 1 à 3, Pauline : parties 4 à 6), la partie 7 (Flask) partagée.

---

## Phase 1 : base et import

1. [ ] Écrire `01_schema.sql` : une table par CSV, types exacts, clés primaires uniquement (les FK viennent après l'import).
2. [ ] Importer les CSV dans cet ordre :
   - `fournisseurs`, `employes`
   - `produits`
   - `commandes`, `demandes_reappro`
   - `receptions`
   - `lignes_commandes`, `lignes_demande`, `lignes_reception`
   Outils : `LOAD DATA LOCAL INFILE` ou l'import phpMyAdmin.
3. [ ] Noter dans le rapport les lignes rejetées ou douteuses (doublons, valeurs vides, dates mal formées).
4. [ ] Nettoyer les données si nécessaire, en gardant une trace de chaque correction.

## Phase 2 : contraintes d'intégrité

5. [ ] Ajouter les clés étrangères, `UNIQUE` (emails), `NOT NULL`, `CHECK` (prix ≥ 0, quantités > 0, stocks ≥ 0, statuts dans une liste), `DEFAULT` (date du jour).
6. [ ] Écrire `04_tests_contraintes.sql` : pour chaque contrainte, un INSERT ou UPDATE invalide qui doit échouer. Noter le message d'erreur dans le rapport.

## Phase 3 : vues

7. [ ] `V_STOCK` : produit, catégorie, stock actuel, stock minimum, `SUM(quantite_commandee)` (qte) et fournisseur. Tester la mise à jour de la catégorie, puis de `qte`, et expliquer le résultat (une vue avec agrégat n'est pas modifiable).
8. [ ] `V_REAPPRO_CRITIQUE` : `WHERE stock_actuel < stock_minimum WITH CHECK OPTION`. Tester une insertion hors critère : refusée avec l'option ; sans l'option, la ligne entre dans `PRODUIT` mais pas dans la vue.

## Phase 4 : droits d'accès

9. [ ] Table `ACCESS` : login, password VARCHAR(32), employee_id, access_level avec `CHECK` sur L, E, U, D, T.
10. [ ] Remplir depuis `employes` : login = `nom.prenom`, password = `MD5(...)`, access_level = T si la fonction est responsable ou gestionnaire, sinon L. Lister les valeurs de `fonction` avant.
11. [ ] Créer 3 utilisateurs MySQL :
    - lecture limitée sur quelques tables
    - lecture + INSERT / UPDATE sur quelques tables
    - droits complets avec `WITH GRANT OPTION`
12. [ ] Tester SELECT, INSERT, UPDATE, DELETE avec chaque compte et noter les refus.

## Phase 5 : accès concurrents

13. [ ] Scénario **mise à jour perdue** : deux sessions décrémentent le même stock ; montrer le résultat faux.
14. [ ] Scénario **lecture sale** : `READ UNCOMMITTED`, une session lit une donnée non commitée, puis rollback.
15. [ ] Démonstration de `SELECT ... FOR UPDATE` dans une transaction, avec une seconde session qui attend le COMMIT.

Note : il faut deux clients MySQL (Workbench ou terminal), phpMyAdmin n'ouvre qu'une session.

## Phase 6 : PL/SQL (triggers et procédures)

16. [ ] **Numéro séquentiel** : `AUTO_INCREMENT` sur `id_demande`, ou table compteur.
17. [ ] **Réapprovisionnement automatique** : trigger `AFTER UPDATE ON produits` qui crée une demande et sa ligne quand le stock passe sous le minimum (sans doublon).
18. [ ] **Hausse de prix par catégorie** : procédure avec paramètres catégorie et pourcentage, contrôle des bornes.
19. [ ] **Historique des statuts** : table `historique_statut_commande` + triggers `AFTER INSERT` et `AFTER UPDATE` (si le statut change).

## Phase 7 : application Flask

20. [ ] Connexion MySQL avec `mysql-connector-python` ou `PyMySQL`, configuration hors Git.
21. [ ] Pages CRUD sur une table (ex. produits) : 2 opérations par type, SQL paramétré, pas d'ORM.
22. [ ] Page de connexion utilisant la table `ACCESS` (démonstration des droits).

## Phase 8 : visualisation (5 visuels)

23. [ ] Une route Flask par visuel, chaque requête SQL en `GROUP BY`.
24. [ ] Graphiques Chart.js (via CDN) :
    - stock actuel vs stock minimum (barres)
    - répartition des produits par catégorie (anneau)
    - commandes par statut (barres empilées)
    - réceptions par mois (courbe)
    - produits en réapprovisionnement critique (tableau)

## Phase 9 : rendu

25. [ ] README : installation, ordre d'exécution des scripts, lancement de Flask.
26. [ ] Rapport : une section par partie, avec le SQL, le résultat et l'explication.
27. [ ] Relecture croisée : chacun teste le travail de l'autre sur son PC.

---

## Matériel nécessaire

- MySQL 8 + phpMyAdmin (et Workbench pour la concurrence)
- Python 3.10+, Flask, mysql-connector-python, Chart.js
- Git + GitHub, VS Code
- Les 9 CSV et le sujet
- Un jeu de données de test pour les contraintes et triggers

## Bonus possibles (à confirmer avec le sujet ou l'enseignant)

- Hash plus robuste que MD5 (SHA-256 salé ou bcrypt) pour `ACCESS.password`
- Vue ou tableau de bord de suivi des réapprovisionnements
- Export CSV depuis l'application
- Index justifiés par `EXPLAIN` sur les requêtes lentes
