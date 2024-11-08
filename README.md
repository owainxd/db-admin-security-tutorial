# Gestion des Utilisateurs, Privilèges et Sécurité dans une Base de Données MySQL

Ce projet guide sur la création et la gestion des utilisateurs, des privilèges et la sécurité des données dans une base de données MySQL. Il comprend la création de tables, l'insertion de données, la configuration des privilèges d'accès, l'utilisation de vues pour restreindre l'accès aux données sensibles et le chiffrement des informations.

## Sommaire
1. [Création de la Base de Données et des Tables](#1-création-de-la-base-de-données-et-des-tables)
2. [Insertion de Données dans les Tables](#2-insertion-de-données-dans-les-tables)
3. [Création des Utilisateurs et Attribution des Privilèges](#3-création-des-utilisateurs-et-attribution-des-privilèges)
4. [Création d’un Rôle de Lecture et Attribution à un Utilisateur](#4-création-dun-rôle-de-lecture-et-attribution-à-un-utilisateur)
5. [Création d’une Vue pour Limiter l’Accès aux Données Sensibles](#5-création-dune-vue-pour-limiter-laccès-aux-données-sensibles)
6. [Chiffrement des Données Sensibles](#6-chiffrement-des-données-sensibles)
7. [Tests des Connexions et Privilèges](#7-tests-des-connexions-et-privilèges)
8. [Révocation des Privilèges et Nettoyage](#8-révocation-des-privilèges-et-nettoyage)
9. [Explications et Conclusions](#9-explications-et-conclusions)

---

### 1. Création de la Base de Données et des Tables

On commence par créer une base de données nommée `entreprise_db` et trois tables associées pour chaque service : `comptabilite_transactions`, `support_clients`, et `admin_informations`.

```sql
-- Création de la base de données
CREATE DATABASE entreprise_db;

-- Sélection de la base de données
USE entreprise_db;

-- Création de la table comptabilite_transactions
CREATE TABLE comptabilite_transactions (
    id INT PRIMARY KEY AUTO_INCREMENT,
    montant DECIMAL(10, 2),
    description VARCHAR(255),
    date_transaction DATE
);

-- Création de la table support_clients
CREATE TABLE support_clients (
    id INT PRIMARY KEY AUTO_INCREMENT,
    nom_client VARCHAR(100),
    email_client VARCHAR(100),
    date_ticket DATE,
    description VARCHAR(255)
);

-- Création de la table admin_informations
CREATE TABLE admin_informations (
    id INT PRIMARY KEY AUTO_INCREMENT,
    info_type VARCHAR(100),
    details TEXT,
    date_enregistrement DATE
);
```
### 2. Insertion de Données dans les Tables
Insérez des données dans chaque table pour que les utilisateurs puissent tester les privilèges.

```sql
Copier le code
-- Insertion dans comptabilite_transactions
INSERT INTO comptabilite_transactions (montant, description, date_transaction) VALUES 
(1500.00, 'Achat de matériel informatique', '2024-01-10'),
(3200.50, 'Paiement de factures de fournisseurs', '2024-02-15'),
(980.75, 'Frais de déplacement', '2024-03-22');

-- Insertion dans support_clients
INSERT INTO support_clients (nom_client, email_client, date_ticket, description) VALUES 
('Alice Dupont', 'alice.dupont@example.com', '2024-01-15', 'Demande d\'assistance pour connexion'),
('Jean Martin', 'jean.martin@example.com', '2024-02-10', 'Problème d\'accès au portail'),
('Sophie Lemaitre', 'sophie.lemaitre@example.com', '2024-03-05', 'Réinitialisation de mot de passe');

-- Insertion dans admin_informations
INSERT INTO admin_informations (info_type, details, date_enregistrement) VALUES 
('Politique de sécurité', 'Nouvelle politique de sécurité pour l\'accès aux locaux', '2024-01-01'),
('Mise à jour logiciel', 'Mise à jour du logiciel interne à la version 3.2.1', '2024-02-01'),
('Politique de télétravail', 'Nouvelle politique de télétravail pour les employés', '2024-03-01');

```

#### 3. Création des Utilisateurs et Attribution des Privilèges
Créons des utilisateurs avec des droits spécifiques pour chaque table.

```sql
CREATE USER 'user_comptabilite'@'%' IDENTIFIED BY 'comptapass';
CREATE USER 'user_support'@'%' IDENTIFIED BY 'supportpass';
CREATE USER 'user_admin'@'%' IDENTIFIED BY 'adminpass';

GRANT SELECT, INSERT, UPDATE ON entreprise_db.comptabilite_transactions TO 'user_comptabilite'@'%';
GRANT SELECT, INSERT, DELETE ON entreprise_db.support_clients TO 'user_support'@'%';
GRANT ALL PRIVILEGES ON entreprise_db.* TO 'user_admin'@'%';
```

### 4. Création d’un Rôle de Lecture et Attribution à un Utilisateur
Créons un rôle lecteur avec accès en lecture pour des utilisateurs comme user_stagiaire.

```sql
CREATE ROLE 'lecteur';
GRANT SELECT ON entreprise_db.* TO 'lecteur';

CREATE USER 'user_stagiaire'@'%' IDENTIFIED BY 'stagiairepass';
GRANT 'lecteur' TO 'user_stagiaire'@'%';
```

### 5. Création d’une Vue pour Limiter l’Accès aux Données Sensibles
Créons une vue vue_comptabilite_resumee pour que certains utilisateurs puissent accéder aux transactions sans voir le montant.

```sql
CREATE VIEW vue_comptabilite_resumee AS
SELECT id, description, date_transaction FROM comptabilite_transactions;

GRANT SELECT ON entreprise_db.vue_comptabilite_resumee TO 'user_support'@'%';
GRANT SELECT ON entreprise_db.vue_comptabilite_resumee TO 'user_admin'@'%';
```

### 6. Chiffrement des Données Sensibles
Ajoutons une colonne chiffrée pour protéger les montants des transactions.

```sql
ALTER TABLE comptabilite_transactions ADD COLUMN montant_chiffre VARBINARY(255);
UPDATE comptabilite_transactions SET montant_chiffre = AES_ENCRYPT(montant, 'mot_de_passe_chiffrement');
```

### 7. Tests des Connexions et Privilèges
Effectuez des tests de connexion avec chaque utilisateur et tentez les actions suivantes :

user_comptabilite : Lecture et modification dans comptabilite_transactions.
user_support : Lecture et suppression dans support_clients, et accès à vue_comptabilite_resumee.
user_admin : Accès complet à toutes les tables et la vue.
user_stagiaire : Lecture seule sur toutes les tables via le rôle lecteur.

### 8. Révocation des Privilèges et Nettoyage
Révoquez les privilèges et supprimez les utilisateurs pour terminer l'exercice.

```sql
REVOKE ALL PRIVILEGES ON entreprise_db.* FROM 'user_comptabilite'@'%';
REVOKE ALL PRIVILEGES ON entreprise_db.* FROM 'user_support'@'%';
REVOKE ALL PRIVILEGES ON entreprise_db.* FROM 'user_admin'@'%';
REVOKE 'lecteur' FROM 'user_stagiaire'@'%';

DROP VIEW vue_comptabilite_resumee;
DROP USER 'user_comptabilite'@'%';
DROP USER 'user_support'@'%';
DROP USER 'user_admin'@'%';
DROP USER 'user_stagiaire'@'%';
```

### 9. Explications et Conclusions
Gestion des Privilèges : Limiter les privilèges réduit les risques d'accès non autorisé et protège les données sensibles.
Chiffrement des Données : Le chiffrement assure que même en cas d'accès non autorisé, les données sensibles ne seront pas lisibles sans la clé.
Utilisation des Vues : Les vues permettent d’offrir un accès restreint aux informations, masquant certaines colonnes et garantissant la sécurité.
Ce guide est conçu pour offrir une solution de gestion des utilisateurs, des privilèges et de la sécurité dans un environnement MySQL.
