# Projet SQL : Menu du Berger

**Étape 1 : Modélisation Conceptuelle (Diagramme E-A) et Relationnelle**

**A. Modèle Conceptuel (Description du Diagramme Entité-Association)**

Vu que je ne peux pas dessiner directement ici, je vais décrire les entités et relations clés. Imaginez un diagramme avec des boîtes pour les entités et des lignes (ou losanges pour les relations M-N avant transformation) les reliant, avec les cardinalités (le nombre minimum et maximum d'instances liées).

- **Entités :** `Client`, `Commande`, `Plat`, `Employe`, `Reservation`.

- **Relations :**
  
  - Un `Client` peut passer (0,n) `Commande`s. Une `Commande` est passée par (1,1) `Client`. (Relation 1:N)
  - Un `Client` peut faire (0,n) `Reservation`(s). Une `Reservation` est faite par (1,1) `Client`. (Relation 1:N)
  - Une `Commande` concerne (1,n) `Plat`(s). Un `Plat` peut être concerné par (0,n) `Commande`(s). (Relation M:N). Cette relation M:N est gérée par l'entité associative `Commande_Plat`.
    - La relation devient : Une `Commande` est composée de (1,n) lignes dans `Commande_Plat`. Un `Plat` est présent dans (0,n) lignes de `Commande_Plat`. Chaque ligne de `Commande_Plat` lie (1,1) `Commande` et (1,1) `Plat`.
  - L'entité `Employe` n'a pas de relation explicite définie dans le cahier des charges avec les autres entités (par exemple, qui a pris la commande ou servi le client). Elle reste donc isolée pour le moment, basée strictement sur les informations fournies.

- **Attributs :** Listés dans le cahier des charges pour chaque entité et l'entité associative `Commande_Plat`.

**B. Modèle Relationnel (Structure des tables)**

Basé sur le modèle conceptuel, on définit les tables, leurs colonnes, et les clés primaires/étrangères.

- **Clients** (`id_client` PK, `nom`, `prenom`, `telephone`, `adresse`)
- **Employes** (`id_employe` PK, `nom`, `prenom`, `poste`, `salaire`, `date_embauche`)
- **Plats** (`id_plat` PK, `nom_plat`, `description`, `prix`, `categorie`)
- **Commandes** (`id_commande` PK, `date_commande`, `heure`, `mode_paiement`, `total`, `id_client` FK -> Clients)
- **Reservations** (`id_reservation` PK, `date_reservation`, `heure`, `nombre_personnes`, `id_client` FK -> Clients)
- **Commande_Plats** (`id_commande` FK -> Commandes, `id_plat` FK -> Plats, `quantite`)
  - *Note : La clé primaire de `Commande_Plats` est composite : (`id_commande`, `id_plat`)*

---

**Étape 2 & 3 : Modélisation Logique (SQL avec types) et Création de la Base (CREATE TABLE)**

Ici, on traduit le modèle relationnel en code SQL `CREATE TABLE`, en choisissant des types de données SQL standards et en ajoutant les contraintes.

SQL

```
-- Table Clients
CREATE TABLE Clients (
    id_client INT PRIMARY KEY AUTO_INCREMENT, -- Ou SERIAL pour PostgreSQL
    nom VARCHAR(100) NOT NULL,
    prenom VARCHAR(100),
    telephone VARCHAR(20) UNIQUE, -- Unicité du numéro de téléphone
    adresse VARCHAR(255)
);

-- Table Employes
CREATE TABLE Employes (
    id_employe INT PRIMARY KEY AUTO_INCREMENT,
    nom VARCHAR(100) NOT NULL,
    prenom VARCHAR(100),
    poste VARCHAR(100) NOT NULL,
    salaire DECIMAL(10, 2) NOT NULL CHECK (salaire >= 0), -- Salaire positif ou nul
    date_embauche DATE NOT NULL
);

-- Table Plats
CREATE TABLE Plats (
    id_plat INT PRIMARY KEY AUTO_INCREMENT,
    nom_plat VARCHAR(150) NOT NULL UNIQUE, -- Nom du plat unique
    description TEXT,
    prix DECIMAL(8, 2) NOT NULL CHECK (prix >= 0), -- Prix positif ou nul
    categorie VARCHAR(50) NOT NULL -- Ex: 'Entrée', 'Plat Principal', 'Dessert', 'Boisson'
);

-- Table Commandes
CREATE TABLE Commandes (
    id_commande INT PRIMARY KEY AUTO_INCREMENT,
    date_commande DATE NOT NULL,
    heure TIME NOT NULL,
    mode_paiement VARCHAR(50) CHECK (mode_paiement IN ('Carte', 'Espèces', 'Mobile')), -- Exemples de modes
    total DECIMAL(10, 2) NOT NULL CHECK (total >= 0), -- Total positif ou nul
    id_client INT NOT NULL,
    FOREIGN KEY (id_client) REFERENCES Clients(id_client) ON DELETE RESTRICT -- Empêche suppression client si commande existe
);

-- Table Reservations
CREATE TABLE Reservations (
    id_reservation INT PRIMARY KEY AUTO_INCREMENT,
    date_reservation DATE NOT NULL,
    heure TIME NOT NULL,
    nombre_personnes INT NOT NULL CHECK (nombre_personnes > 0), -- Au moins 1 personne
    id_client INT NOT NULL,
    FOREIGN KEY (id_client) REFERENCES Clients(id_client) ON DELETE RESTRICT
);

-- Table Associative Commande_Plats
CREATE TABLE Commande_Plats (
    id_commande INT NOT NULL,
    id_plat INT NOT NULL,
    quantite INT NOT NULL CHECK (quantite > 0), -- Quantité doit être positive
    PRIMARY KEY (id_commande, id_plat), -- Clé primaire composite
    FOREIGN KEY (id_commande) REFERENCES Commandes(id_commande) ON DELETE CASCADE, -- Si commande supprimée, lignes associées aussi
    FOREIGN KEY (id_plat) REFERENCES Plats(id_plat) ON DELETE RESTRICT -- Empêche suppression plat si dans une commande
);
```

*Remarques sur les contraintes :*

- `AUTO_INCREMENT` (MySQL) ou `SERIAL` (PostgreSQL) sont courants pour les clés primaires. Si vous n'en utilisez pas, vous devrez fournir l'ID lors de l'insertion.
- `ON DELETE RESTRICT` empêche la suppression d'une ligne parente (ex: Client) s'il existe des lignes enfants (ex: Commandes).
- `ON DELETE CASCADE` supprime automatiquement les lignes enfants (ex: Commande_Plats) si la ligne parente (ex: Commande) est supprimée. À utiliser avec prudence.

---

**Étape 4 : Insertion de Données Fictives (Exemples)**

Il faut insérer les données dans un ordre qui respecte les clés étrangères : d'abord les tables indépendantes (`Clients`, `Employes`, `Plats`), puis les tables dépendantes (`Commandes`, `Reservations`), et enfin la table associative (`Commande_Plats`).

SQL

```
-- Exemples d'insertion (à compléter pour atteindre les quantités demandées)

-- Clients (Ex: 3 sur 15)
INSERT INTO Clients (nom, prenom, telephone, adresse) VALUES
('Dupont', 'Jean', '0612345678', '1 rue de la Paix, Paris'),
('Martin', 'Alice', '0787654321', '15 avenue des Champs, Lyon'),
('Diallo', 'Moussa', '0699887766', '5 boulevard de la République, Dakar');

-- Employes (Ex: 2 sur 5)
INSERT INTO Employes (nom, prenom, poste, salaire, date_embauche) VALUES
('Ba', 'Fatou', 'Serveuse', 1800.00, '2023-01-15'),
('Ndiaye', 'Omar', 'Cuisinier', 2200.00, '2022-11-01');

-- Plats (Ex: 4 sur 20)
INSERT INTO Plats (nom_plat, description, prix, categorie) VALUES
('Salade César', 'Salade romaine, poulet grillé, croûtons, parmesan', 12.50, 'Entrée'),
('Thieboudienne', 'Riz au poisson sénégalais', 15.00, 'Plat Principal'),
('Mousse au Chocolat', 'Mousse légère au chocolat noir', 7.00, 'Dessert'),
('Jus de Bissap', 'Jus d\'hibiscus frais', 3.50, 'Boisson');-- Commandes (Ex: 2 sur 15) - Assurez-vous que les id_client existentINSERT INTO Commandes (date_commande, heure, mode_paiement, total, id_client) VALUES('2025-04-28', '12:30:00', 'Carte', 27.50, 1), -- Commande de Dupont('2025-04-29', '19:45:00', 'Espèces', 18.50, 3); -- Commande de Diallo-- Reservations (Ex: 2 sur 5) - Assurez-vous que les id_client existentINSERT INTO Reservations (date_reservation, heure, nombre_personnes, id_client) VALUES('2025-05-01', '20:00:00', 4, 2), -- Reservation de Martin('2025-05-03', '13:00:00', 2, 1); -- Reservation de Dupont-- Commande_Plats (Ex: Lier plats aux commandes créées)-- Pour Commande 1 (total 27.50)INSERT INTO Commande_Plats (id_commande, id_plat, quantite) VALUES(1, 1, 1), -- 1x Salade César (12.50)(1, 2, 1); -- 1x Thieboudienne (15.00)-- Pour Commande 2 (total 18.50)INSERT INTO Commande_Plats (id_commande, id_plat, quantite) VALUES(2, 2, 1), -- 1x Thieboudienne (15.00)(2, 4, 1); -- 1x Jus de Bissap (3.50)-- !! Répétez les INSERT avec des données variées pour atteindre les nombres requis !!
```

---

**Étape 5 : Écriture de Requêtes SQL**

**Requêtes Simples**

- **Tous les plats triés par prix (du moins cher au plus cher) :**
  
  SQL
  
  ```
  SELECT nom_plat, prix, categorie
  FROM Plats
  ORDER BY prix ASC;
  ```

- **Liste des clients avec téléphone :**
  
  SQL
  
  ```
  SELECT nom, prenom, telephone
  FROM Clients
  WHERE telephone IS NOT NULL AND telephone != ''; -- S'assurer qu'il y a un numéro
  ```

- **Toutes les réservations du jour (Aujourd'hui = 29 Avril 2025) :**
  
  SQL
  
  ```
  SELECT r.id_reservation, r.heure, r.nombre_personnes, c.nom, c.prenom
  FROM Reservations r
  JOIN Clients c ON r.id_client = c.id_client
  WHERE r.date_reservation = CURDATE(); -- Ou '2025-04-29' ou GETDATE() selon le SGBD
  ```

- **Liste des employés embauchés après le 1er Janvier 2023 (inclus) :**
  
  SQL
  
  ```
  SELECT nom, prenom, poste, date_embauche
  FROM Employes
  WHERE date_embauche >= '2023-01-01';
  ```

**Jointures**

- **Détails des plats d’une commande spécifique (ex: id_commande = 1) :**
  
  SQL
  
  ```
  SELECT p.nom_plat, cp.quantite, p.prix, (cp.quantite * p.prix) AS sous_total
  FROM Commande_Plats cp
  JOIN Plats p ON cp.id_plat = p.id_plat
  WHERE cp.id_commande = 1;
  ```

- **Liste des commandes avec nom du client :**
  
  SQL
  
  ```
  SELECT co.id_commande, co.date_commande, co.heure, co.total, cl.nom, cl.prenom
  FROM Commandes co
  JOIN Clients cl ON co.id_client = cl.id_client
  ORDER BY co.date_commande DESC, co.heure DESC;
  ```

- **Clients ayant à la fois réservé et commandé (au moins une fois chacun) :**
  
  SQL
  
  ```
  SELECT DISTINCT c.id_client, c.nom, c.prenom
  FROM Clients c
  JOIN Commandes co ON c.id_client = co.id_client
  JOIN Reservations r ON c.id_client = r.id_client;
  -- Alternative avec EXISTS pour potentiellement plus d'efficacité
  -- SELECT c.id_client, c.nom, c.prenom
  -- FROM Clients c
  -- WHERE EXISTS (SELECT 1 FROM Commandes co WHERE co.id_client = c.id_client)
  --   AND EXISTS (SELECT 1 FROM Reservations r WHERE r.id_client = c.id_client);
  ```

**Agrégations**

- **Total des ventes par jour :**
  
  SQL
  
  ```
  SELECT date_commande, SUM(total) AS ventes_totales_jour
  FROM Commandes
  GROUP BY date_commande
  ORDER BY date_commande DESC;
  ```

- **Plat le plus commandé (en quantité totale) :**
  
  SQL
  
  ```
  SELECT p.nom_plat, SUM(cp.quantite) AS quantite_totale_commandee
  FROM Commande_Plats cp
  JOIN Plats p ON cp.id_plat = p.id_plat
  GROUP BY p.id_plat, p.nom_plat -- Inclure nom_plat car il est dans SELECT
  ORDER BY quantite_totale_commandee DESC
  LIMIT 1;
  ```

- **Panier moyen par client (Total dépensé / Nombre de commandes) :**
  
  SQL
  
  ```
  SELECT
      c.id_client,
      c.nom,
      c.prenom,
      COUNT(co.id_commande) AS nombre_commandes,
      SUM(co.total) AS total_depense,
      AVG(co.total) AS panier_moyen -- SUM(co.total) / COUNT(co.id_commande)
  FROM Clients c
  JOIN Commandes co ON c.id_client = co.id_client
  GROUP BY c.id_client, c.nom, c.prenom
  HAVING COUNT(co.id_commande) > 0 -- Pour éviter division par zéro si client sans commande
  ORDER BY panier_moyen DESC;
  ```

- **Nombre de plats vendus par catégorie :**
  
  SQL
  
  ```
  SELECT p.categorie, SUM(cp.quantite) AS nombre_plats_vendus
  FROM Commande_Plats cp
  JOIN Plats p ON cp.id_plat = p.id_plat
  GROUP BY p.categorie
  ORDER BY nombre_plats_vendus DESC;
  ```

**Requêtes Complexes**

- **Clients fidèles (ayant passé plus de 5 commandes) :**
  
  SQL
  
  ```
  SELECT c.id_client, c.nom, c.prenom, COUNT(co.id_commande) AS nombre_commandes
  FROM Clients c
  JOIN Commandes co ON c.id_client = co.id_client
  GROUP BY c.id_client, c.nom, c.prenom
  HAVING COUNT(co.id_commande) > 5
  ORDER BY nombre_commandes DESC;
  ```

- **Plats jamais commandés :**
  
  SQL
  
  ```
  SELECT p.id_plat, p.nom_plat, p.prix
  FROM Plats p
  LEFT JOIN Commande_Plats cp ON p.id_plat = cp.id_plat
  WHERE cp.id_plat IS NULL; -- Si aucune correspondance dans Commande_Plats
  ```

- **Répartition des commandes par tranche horaire (ex: par heure) :**
  
  SQL
  
  ```
  SELECT
      EXTRACT(HOUR FROM heure) AS heure_commande, -- Ou HOUR(heure) selon SGBD
      COUNT(id_commande) AS nombre_commandes
  FROM Commandes
  GROUP BY heure_commande
  ORDER BY heure_commande;
  ```

- **Top 3 des clients ayant dépensé le plus :**
  
  SQL
  
  ```
  SELECT
      c.id_client,
      c.nom,
      c.prenom,
      SUM(co.total) AS total_depense_client
  FROM Clients c
  JOIN Commandes co ON c.id_client = co.id_client
  GROUP BY c.id_client, c.nom, c.prenom
  ORDER BY total_depense_client DESC
  LIMIT 3;
  ```

---

Ce plan couvre toutes les étapes demandées. N'oubliez pas d'adapter les types de données exacts (`AUTO_INCREMENT` vs `SERIAL`, `CURDATE()` vs `GETDATE()` vs `'YYYY-MM-DD'`, `HOUR()` vs `EXTRACT(HOUR FROM ...)`) en fonction du système de gestion de base de données (MySQL, PostgreSQL, SQL Server, etc.) que vous utiliserez. Bon travail de mise en place !
