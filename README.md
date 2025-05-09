# Projet SQL : Menu du Berger

**Étape 1 : Modélisation Conceptuelle (Diagramme E-A) et Relationnelle**

**A. Modèle Conceptuel (Description du Diagramme Entité-Association)**

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
- **Commandes** (`id_commande` PK, `date_commande`, `quantite`, `heure`, `mode_paiement`, `total`, `id_client`, `id_plat` FK -> Clients, Plats)
- **Reservations** (`id_reservation` PK, `date_reservation`, `heure`, `nombre_personnes`, `id_client`, `id_employe` FK -> Clients, Employes)

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
    quantite INT NOT NULL,
    heure TIME NOT NULL,
    mode_paiement VARCHAR(50) CHECK (mode_paiement IN ('Carte', 'Espèces', 'Mobile')), -- Exemples de modes
    total DECIMAL(10, 2) NOT NULL CHECK (total >= 0), -- Total positif ou nul
    id_client INT NOT NULL,
    id_plat INT NOT NULL,
    FOREIGN KEY (id_client) REFERENCES Clients(id_client) ON DELETE RESTRICT, -- Empêche suppression client si commande existe
    FOREIGN KEY (id_plat) REFERENCES Plats(id_plat) ON DELETE RESTRICT -- Empêche suppression plat si commande existe
);

-- Table Reservations
CREATE TABLE Reservations (
    id_reservation INT PRIMARY KEY AUTO_INCREMENT,
    date_reservation DATE NOT NULL,
    heure TIME NOT NULL,
    nombre_personnes INT NOT NULL CHECK (nombre_personnes > 0), -- Au moins 1 personne
    id_client INT NOT NULL,
    id_employe INT NOT NULL,
    FOREIGN KEY (id_client) REFERENCES Clients(id_client) ON DELETE RESTRICT,
    FOREIGN KEY (id_employe) REFERENCES Employes(id_employe) ON DELETE RESTRICT
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
-- Script SQL pour l'insertion de données fictives dans la base de données du restaurant
-- IMPORTANT : Assurez-vous que les tables ont été créées avec les bonnes structures et contraintes
-- (AUTO_INCREMENT pour les IDs, clés étrangères, etc.) avant d'exécuter ce script.

-- -----------------------------------------------------
-- Insertion dans la table `Clients` (15 clients)
-- -----------------------------------------------------
INSERT INTO Clients (nom, prenom, telephone, adresse) VALUES
('Diallo', 'Aïssatou', '771234567', '12 Rue Sandaga, Dakar'),
('Faye', 'Mamadou', '782345678', 'Avenue Cheikh Anta Diop, Fann'),
('Ndiaye', 'Bineta', '763456789', 'Cité Keur Gorgui, Dakar'),
('Sow', 'Ibrahim', '704567890', 'Sicap Liberté 6, Dakar'),
('Gomez', 'Fatoumata', '775678901', 'Ngor Almadies, Dakar'),
('Traoré', 'Sekou', '786789012', 'Point E, Dakar'),
('Cissé', 'Mariama', '767890123', 'Ouakam, Dakar'),
('Ba', 'Ousmane', '708901234', 'Yoff, Dakar'),
('Sylla', 'Aminata', '779012345', 'Mermoz, Dakar'),
('Kante', 'Moussa', '780123456', 'Sacré Coeur, Dakar'),
('Diouf', 'Adama', '761234560', 'Grand Dakar, Dakar'),
('Mbaye', 'Khadija', '702345671', 'Pikine, Dakar'),
('Camara', 'Lamine', '773456782', 'Guediawaye, Dakar'),
('Thiam', 'Rokhaya', '784567893', 'Rufisque, Dakar'),
('Fall', 'Cheikh', '765678904', 'Dakar Plateau, Dakar');

-- -----------------------------------------------------
-- Insertion dans la table `Employes` (5 employés)
-- -----------------------------------------------------
INSERT INTO Employes (nom, prenom, poste, salaire, date_embauche) VALUES
('Diop', 'Ousmane', 'Cuisinier Principal', 350000, '2022-08-10'),
('Sarr', 'Fatou', 'Serveuse', 180000, '2023-03-01'),
('Gueye', 'Alioune', 'Aide-Cuisinier', 220000, '2023-07-15'),
('Mbodj', 'Awa', 'Caissière', 190000, '2022-11-05'),
('Ndour', 'Pape', 'Plongeur', 150000, '2024-01-20');

-- -----------------------------------------------------
-- Insertion dans la table `Plats` (20 plats)
-- -----------------------------------------------------
INSERT INTO Plats (nom_plat, description, prix, categorie) VALUES
-- Entrées
('Salade Sénégalaise', 'Salade verte, tomates, concombres, oignons, vinaigrette locale', 2500, 'Entrée'),
('Pastels au Thon', 'Beignets farcis au thon et épices', 3000, 'Entrée'),
('Nems aux Crevettes', 'Rouleaux de printemps frits aux crevettes', 3500, 'Entrée'),
('Soupe Kandia', 'Soupe de gombos avec huile de palme', 2000, 'Entrée'),
('Accras de Niébé', 'Beignets de haricots cornilles épicés', 2800, 'Entrée'),
-- Plats Principaux
('Thieboudienne Poisson', 'Riz rouge au poisson frais et légumes variés', 5500, 'Plat Principal'),
('Yassa Poulet', 'Poulet mariné au citron et aux oignons, servi avec du riz blanc', 5000, 'Plat Principal'),
('Mafé Boeuf', 'Ragoût de boeuf à la pâte d''arachide, servi avec du riz', 6000, 'Plat Principal'),
('Thiof Grillé', 'Mérou grillé, servi avec frites ou alokos et sauce', 7500, 'Plat Principal'),
('Couscous Sénégalais (Thiéré)', 'Couscous de mil avec sauce légumes et viande', 5800, 'Plat Principal'),
('Domoda Poisson', 'Poisson mijoté dans une sauce tomate citronnée et moutardée', 5200, 'Plat Principal'),
('Poulet DG', 'Poulet sauté avec plantains, carottes, poivrons (style camerounais)', 6500, 'Plat Principal'),
-- Desserts
('Thiagry', 'Dessert à base de couscous de mil, yaourt et lait caillé sucré', 2500, 'Dessert'),
('Salade de Fruits Frais', 'Assortiment de fruits de saison', 3000, 'Dessert'),
('Mousse au Chocolat Noir', 'Mousse onctueuse au chocolat noir', 3500, 'Dessert'),
('Cinq Centimes', 'Petits beignets sucrés', 1500, 'Dessert'),
-- Boissons
('Jus de Bissap', 'Jus de fleurs d''hibiscus', 1500, 'Boisson'),
('Jus de Gingembre (Gnamakoudji)', 'Jus de gingembre frais et épicé', 1500, 'Boisson'),
('Jus de Bouye', 'Jus de fruit du baobab (pain de singe)', 2000, 'Boisson'),
('Eau Minérale (Grand format)', 'Bouteille d''eau minérale 1.5L', 1000, 'Boisson');

-- -----------------------------------------------------
-- Insertion dans la table `Commandes` (15 commandes)
-- Les totaux sont calculés ici manuellement pour l'exemple.
-- Dans un système réel, ils seraient calculés dynamiquement ou via triggers.
-- -----------------------------------------------------
-- id_client références de 1 à 15
INSERT INTO Commandes (date_commande, heure, quantite, mode_paiement, total, id_plat, id_client) VALUES
('2025-05-05', '12:30:00',3, 'Mobile', 8500, 4, 1),  -- Commande 1: Thieboudienne (5500) + Bissap (1500) + Pastels (3000) -> Erreur de calcul manuel, ajustons. Total = 10000
('2025-05-05', '13:15:00',2, 'Carte', 7000, 16, 2),   -- Commande 2: Yassa (5000) + Bouye (2000)
('2025-05-06', '19:45:00',1, 'Espèces', 11000, 4, 3), -- Commande 3: Mafé (6000) + Salade Fruits (3000) + Gingembre (1500) -> Total = 10500
('2025-05-06', '20:10:00',5, 'Carte', 7500, 18, 4),   -- Commande 4: Thiof Grillé (7500)
('2025-05-07', '12:00:00',3, 'Mobile', 9300, 2, 5),   -- Commande 5: Couscous (5800) + Nems (3500)
('2025-05-07', '14:00:00',6, 'Espèces', 10500, 19, 6), -- Commande 6: Poulet DG (6500) + Thiagry (2500) + Eau (1000)
('2025-05-08', '19:00:00',3, 'Carte', 5200, 17, 7),   -- Commande 7: Domoda (5200)
('2025-05-08', '20:30:00',5, 'Mobile', 9000, 15, 1),   -- Commande 8: Yassa (5000) + Bissap (1500) + Pastels (3000) -> Total = 9500
('2025-05-09', '12:45:00',1, 'Espèces', 8000, 13, 8),   -- Commande 9: Mafé (6000) + Bouye (2000)
('2025-05-09', '13:30:00',4, 'Carte', 12500, 11, 9),  -- Commande 10: Thiof (7500) + Salade Sen. (2500) + Mousse Choc. (3500) -> Total = 13500
('2025-05-10', '19:15:00',2, 'Mobile', 8300, 9, 10),  -- Commande 11: Couscous (5800) + Cinq Centimes (1500) + Eau (1000)
('2025-05-10', '21:00:00',2, 'Espèces', 11700, 7, 11), -- Commande 12: Poulet DG (6500) + Accras (2800) + Bissap (1500) + Eau (1000) -> Total = 11800
('2025-05-11', '12:10:00',4, 'Carte', 7700, 5, 12),  -- Commande 13: Domoda (5200) + Salade Fruits (3000) -> Total = 8200
('2025-05-11', '13:05:00',3, 'Mobile', 14000, 3, 13), -- Commande 14: Thieb (5500) + Yassa (5000) + 2xGingembre (3000)
('2025-05-12', '20:00:00',1, 'Espèces', 10000, 1, 14); -- Commande 15: Thiof (7500) + Thiagry (2500)

-- Ajustement des totaux pour les commandes où le calcul manuel initial était erroné
UPDATE Commandes SET total = 10000 WHERE id_commande = 1;
UPDATE Commandes SET total = 10500 WHERE id_commande = 3;
UPDATE Commandes SET total = 9500 WHERE id_commande = 8;
UPDATE Commandes SET total = 13500 WHERE id_commande = 10;
UPDATE Commandes SET total = 11800 WHERE id_commande = 12;
UPDATE Commandes SET total = 8200 WHERE id_commande = 13;


-- -----------------------------------------------------
-- Insertion dans la table `Reservations` (5 réservations)
-- -----------------------------------------------------
-- id_client références de 1 à 15
INSERT INTO Reservations (date_reservation, heure, nombre_personnes, id_client, id_employe) VALUES
('2025-05-15', '20:00:00', 4, 2, 1),  -- Reservation 1
('2025-05-16', '13:00:00', 2, 5, 2),  -- Reservation 2
('2025-05-17', '19:30:00', 6, 8, 3),  -- Reservation 3
('2025-05-18', '12:30:00', 3, 11, 4), -- Reservation 4
('2025-05-19', '20:30:00', 5, 15, 5); -- Reservation 5

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

Le principe consistera à adapter les types de données exacts (`AUTO_INCREMENT` vs `SERIAL`, `CURDATE()` vs `GETDATE()` vs `'YYYY-MM-DD'`, `HOUR()` vs `EXTRACT(HOUR FROM ...)`) en fonction du système de gestion de base de données (MySQL, PostgreSQL, SQL Server, etc.) utilisé.
