# Base de Données `healthcare` - Sécurité et Intégrité

Ce projet de base de données est conçu pour gérer les informations relatives aux patients, fabricants, vaccins et carnets de vaccination, avec une attention particulière sur la sécurité et l'intégrité des données. La structure inclut des clés étrangères avec options `ON UPDATE` et `ON DELETE` pour assurer la cohérence et la gestion des dépendances.

## Sommaire
1. [Structure de la Base de Données](#structure-de-la-base-de-données)
   - [Création de la Base de Données](#création-de-la-base-de-données)
   - [Tables et Contraintes](#tables-et-contraintes)
2. [Options `ON UPDATE` et `ON DELETE`](#options-on-update-et-on-delete)
   - [Explications des Options](#explication-des-options)
3. [Scénario de Test](#scénario-de-test)
   - [Étapes de Test](#étapes-de-test)
   - [Résultats Attendus](#résultats-attendus)
---

## Structure de la Base de Données

### Création de la Base de Données

La base de données `healthcare` est créée pour stocker des informations sur les patients, les fabricants de vaccins, les vaccins eux-mêmes, ainsi que les carnets de vaccination.

# Tables et Contraintes

Voici les tables de la base de données avec leurs clés primaires, clés étrangères et contraintes d’intégrité.

- **Table `patients`** : Stocke les informations de chaque patient.
- **Table `fabricants`** : Stocke les informations de chaque fabricant de vaccins.
- **Table `vaccins`** : Stocke les informations sur chaque vaccin, avec une référence au fabricant.
- **Table `carnets`** : Stocke les informations sur les vaccins reçus par les patients, en reliant `patients` et `vaccins` par des clés étrangères.
  
```sql
CREATE DATABASE healthcare; 
USE healthcare; 
```


# Base de Données `healthcare` - Sécurité et Intégrité avec Contraintes

Cette base de données est conçue pour gérer les informations de patients, fabricants, vaccins, et carnets de vaccination, avec des contraintes de sécurité et d'intégrité. Elle inclut des clés étrangères avec options `ON UPDATE` et `ON DELETE` pour gérer les mises à jour et suppressions dans un environnement relationnel sécurisé.

## Structure de la Base de Données

### Création de la Base de Données et des Tables

```sql
CREATE DATABASE healthcare; 
USE healthcare; 

-- Table PATIENTS
CREATE TABLE patients(
    num_p INT PRIMARY KEY AUTO_INCREMENT, 
    nom VARCHAR(255) NOT NULL, 
    prenom VARCHAR(255) NOT NULL, 
    age INT NOT NULL
);

-- Table FABRICANTS
CREATE TABLE fabricants(
    num_f INT PRIMARY KEY AUTO_INCREMENT, 
    nom VARCHAR(255) NOT NULL, 
    adresse VARCHAR(255) NOT NULL  
); 

-- Table VACCINS avec options ON UPDATE et ON DELETE
CREATE TABLE vaccins(
    num_v INT AUTO_INCREMENT, 
    nom VARCHAR(255) NOT NULL, 
    num_f INT,
    PRIMARY KEY(num_v), 
    FOREIGN KEY(num_f) REFERENCES fabricants(num_f)
    ON UPDATE CASCADE
    ON DELETE SET NULL
); 

-- Table CARNETS avec options ON UPDATE et ON DELETE
CREATE TABLE carnets(
    num_p INT, 
    num_v INT, 
    date_vaccin DATE NOT NULL,
    PRIMARY KEY(num_p, num_v), 
    FOREIGN KEY(num_p) REFERENCES patients(num_p)
    ON UPDATE CASCADE
    ON DELETE CASCADE, 
    FOREIGN KEY(num_v) REFERENCES vaccins(num_v)
    ON UPDATE CASCADE
    ON DELETE CASCADE
); 
```
# Options `ON UPDATE` et `ON DELETE`

## Explication des Options

- **ON UPDATE CASCADE** : Répercute les modifications de la clé primaire dans les tables enfants. Si une clé primaire est modifiée dans la table parente, les tables enfants seront automatiquement mises à jour pour refléter cette modification.
- **ON DELETE CASCADE** : Supprime les enregistrements dépendants dans les tables enfants lorsque l'enregistrement parent est supprimé.
- **ON DELETE SET NULL** : Si un fabricant est supprimé dans la table `fabricants`, le champ `num_f` dans `vaccins` est mis à `NULL` pour indiquer que le fabricant n'existe plus sans supprimer le vaccin.


## Insertion des Données de Test
```sql
-- Insérer des patients
INSERT INTO patients (nom, prenom, age) VALUES ('Dupont', 'Jean', 45);
INSERT INTO patients (nom, prenom, age) VALUES ('Martin', 'Marie', 30);

-- Insérer des fabricants
INSERT INTO fabricants (nom, adresse) VALUES ('BioPharma', '123 Rue de la Santé');
INSERT INTO fabricants (nom, adresse) VALUES ('VacciCorp', '456 Avenue de la Science');

-- Insérer des vaccins
INSERT INTO vaccins (nom, num_f) VALUES ('Vaccin A', 1); -- Référence à BioPharma
INSERT INTO vaccins (nom, num_f) VALUES ('Vaccin B', 2); -- Référence à VacciCorp

-- Insérer des enregistrements dans la table carnets
INSERT INTO carnets (num_p, num_v, date_vaccin) VALUES (1, 1, '2023-01-15');
INSERT INTO carnets (num_p, num_v, date_vaccin) VALUES (2, 2, '2023-02-20');
```


## Scénario de Test

Ce scénario de test permet de vérifier que les contraintes `ON UPDATE` et `ON DELETE` fonctionnent correctement pour assurer la cohérence et l'intégrité des données.

### Étapes de Test

1. **Insertion des Données de Test**

   Ajouter des enregistrements dans chaque table pour configurer un ensemble de données de test.
   
```sql
-- Insérer des patients
INSERT INTO patients (nom, prenom, age) VALUES ('Dupont', 'Jean', 45);
INSERT INTO patients (nom, prenom, age) VALUES ('Martin', 'Marie', 30);

-- Insérer des fabricants
INSERT INTO fabricants (nom, adresse) VALUES ('BioPharma', '123 Rue de la Santé');
INSERT INTO fabricants (nom, adresse) VALUES ('VacciCorp', '456 Avenue de la Science');

-- Insérer des vaccins
INSERT INTO vaccins (nom, num_f) VALUES ('Vaccin A', 1); -- Référence à BioPharma
INSERT INTO vaccins (nom, num_f) VALUES ('Vaccin B', 2); -- Référence à VacciCorp

-- Insérer des enregistrements dans la table carnets
INSERT INTO carnets (num_p, num_v, date_vaccin) VALUES (1, 1, '2023-01-15');
INSERT INTO carnets (num_p, num_v, date_vaccin) VALUES (2, 2, '2023-02-20');

```

2. **Test de la Contrainte `ON DELETE CASCADE`**
   - Supprimez un patient dans la table `patients`.
   - Vérifiez que l'enregistrement associé dans `carnets` est automatiquement supprimé.

  ```sql
   -- Suppression d'un patient
   DELETE FROM patients WHERE num_p = 1;

   -- Vérification dans la table carnets
   SELECT * FROM carnets WHERE num_p = 1; -- Doit retourner zéro ligne
```
3. **Test de la Contrainte `ON DELETE SET NULL`**
  - Supprimez un fabricant dans la table `fabricants`.
  - Vérifiez que le champ `num_f` dans `vaccins` est mis à `NULL` pour les vaccins correspondants.
    
  ```sql
-- Suppression d'un fabricant
DELETE FROM fabricants WHERE num_f = 1;

-- Vérification dans la table vaccins
SELECT * FROM vaccins WHERE num_f IS NULL; -- Doit montrer les vaccins avec num_f = NULL

```
4. **Test de la Contrainte `ON UPDATE CASCADE`**
   - Mettez à jour le `num_p` d’un patient dans la table `patients` et le `num_v` dans `vaccins`.
   - Vérifiez que les changements sont propagés dans la table `carnets`.
     
```sql
-- Mise à jour d'un patient
UPDATE patients SET num_p = 3 WHERE nom = 'Martin' AND prenom = 'Marie';

-- Vérification dans la table carnets
SELECT * FROM carnets WHERE num_p = 3; -- Doit montrer l'ID mis à jour

-- Mise à jour d'un vaccin
UPDATE vaccins SET num_v = 4 WHERE nom = 'Vaccin B';

-- Vérification dans la table carnets
SELECT * FROM carnets WHERE num_v = 4; -- Doit montrer l'ID mis à jour

```

## Résultats Attendus

- **ON DELETE CASCADE** : Les enregistrements correspondants dans `carnets` sont automatiquement supprimés lorsque vous supprimez un enregistrement de `patients` ou `vaccins`.
- **ON DELETE SET NULL** : Lors de la suppression d’un fabricant dans `fabricants`, le champ `num_f` dans `vaccins` est mis à `NULL`.
- **ON UPDATE CASCADE** : Toute mise à jour de la clé primaire dans `patients` ou `vaccins` est répercutée automatiquement dans `carnets`.

