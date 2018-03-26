# DreamTeam :

    animateur : Émilien
    secrétaire : Fantou
    gestionnaire : Hugo
    scribe : Maxime

    
# Prosit 5 : PHP Objet et BDD


## Mots clés :

- PDO
- BDD
- SGBD
- Procédures stockées
- Requêtes préparées
- Transactions
- SQL
- Failles de sécurité
- Recherche par mots clés
- Trier les produits
- Maquette
- Modélisation
- Mise en relation site/BDD


## Contexte :

- Quoi > Connecter le site à la BDD de manière sécurisée
- Pourquoi > Obtenir des informations sur les produits
- Comment > En utilisant le PHP et un PDO


##  Contraintes :

- Sécurisé
- Générique
- SQL

## Problématique :

Comment interagir avec une BDD depuis un site web de manière sécurisé et générique avec PHP ?

Généralisation :
- Reconception
- Implémentation


##  Hypothèses :
-PDO est une extension PHP qui permet de se connecter à une BDD
-On peut lier un site à une BDD avec SPNet
-Des fonctions PHP sont prévues pour sécuriser l’accès aux données
-Les procédures stockées permettent d’augmenter la sécurité


## Plan d’action :

**Etudier:**

- PDO + concurrents
        - SQL
        - Types de requêtes BDD
        - Types d’attaques et défenses possibles

**Réalisation:**   

- WS


# BDD en PHP :

## Lire les données
Deux moyens :
- mysqli_ (suite de mysql_) ⇒ mysqli_connect
- PDO


L'avantage de PDO :

![](https://user.oc-static.com/files/219001_220000/219973.png)

Se connecter à la base de données :
 ```
 try{
$bdd = new PDO('mysql:host=' ';dbname=' ';charset=utf8', 'login', 'password');
}
catch (Exception $e)
{
	die('Erreur : '.$e->getMessage());
}
```
mysql s'appelle le DSN : **D**ata **S**ource **N**ame - C'est celui qui change en fonction du type de BDD on se connecte.

```
// Faire une requête SQL
$reponse = $bdd->query('SELECT nom FROM jeux_video');

// Parcourir les données reçus si plusieurs lignes
while ($donnees = $reponse->fetch())
{
	echo $donnees['nom'];//On va afficher tous les noms
}
```
Après chaque requête, il faut fermer les recherches avec "`closeCursor()`".

**Les requêtes préparés**  //Permet d'éviter les injections SQL:
- Beaucoup plus sûres et plus rapides, si la requête est exécutée plusieurs fois.
- On peut lui passer des arguments à tout moment :

```
$req = $bdd->prepare('SELECT nom FROM jeux_video WHERE possesseur = ?');
$req->execute(array($_GET['possesseur']));
```

**Activer les détails sur des erreurs lors de requêtes**
- Si on fais des erreurs sur la connexion de la BDD, on peut mettre cette ligne sous la connexion pour plus d'infos : 
```
array(PDO::ATTR_ERRMODE => PDO::ERRMODE_EXCEPTION));
```
- Même principe sur les requêtes : 
 ```
$reponse = $bdd->query('SELECT nom FROM jeux_video') or die(print_r($bdd->errorInfo()));
```

## Écrire les données :

**Insetion classique**
```
$bdd->exec('INSERT INTO jeux_video(nom, possesseur, console, prix, nbre_joueurs_max, commentaires) VALUES(\'Battlefield 1942\', \'Patrick\', \'PC\', 45, 50, \'2nde guerre mondiale\')');
```

**Requête préparée**
```
$req = $bdd->prepare('INSERT INTO jeux_video(nom, possesseur, console, prix, nbre_joueurs_max, commentaires) VALUES(:nom, :possesseur, :console, :prix, :nbre_joueurs_max, :commentaires)');
$req->execute(array(
	'nom' => $nom,
	'possesseur' => $possesseur,
	'console' => $console,
	'prix' => $prix,
	'nbre_joueurs_max' => $nbre_joueurs_max,
	'commentaires' => $commentaires
	));
```

## Rappels sur le SQL :
- Select : Récupérer des infos dans une base de données
- Insert into : Ajout d'une entrée
- Update : Modification d'une ou plusieurs entrées
- Delete : Suppression d'une ou plusieurs entrées

- Where (filtre) / order by (tri) / limit (limiations) : Affiner ses requêtes.

## Les transactions :

```
public bool **PDO::beginTransaction** ( void )
```
Désactive le mode autocommit, c'est à dire que les modifications faites sur la BDD via PDO ne seront pas appliquées tant que l'on ne met pas la fin de la transaction avec commit();

MYSQL exécute automatiquement un commit lorsqu'une requête de type DROP TABLE ou CREATE TABLE est exécutée dans une transaction. Un COMMIT empêche de revenir en arrière.

Par la même occasion, la fonction rollback(); permet d'annuler toutes les modifications et remet en mode autocommit.

**Les moteurs de stockage de MySQL**

Dans une base de données, on peut avoir plusieurs moteurs de stockages.

- Mylsam 
	- est le moteur par défaut de MySQL
	- très simple d'utilisation & très bonnes performances
	-  index FULL-TEXT (recherches faciles) comme "LIKE"

MAIS

Souffre sur les grosses tables, sa configuration (taille des mots) n'est accéssible que un serveur dédién ne supporte NI les clefs étrangères, ni les transactions.

- InnoDB :
	- Supporte les transactions
	- Le plus utilisé dans les secteurs sensibles (finances, jeux en ligne, architecture complexe...)
	- Gestion des clés étrangères

MAIS

Tables plus volumineuses, ne pas proposer d'index FULL-TEXT, légèrement plus lent.

- Memory (Headp): tout dans la RAM

- Stocke toutes ses tables dans la mémoire / strucure quand fichier
- Rapide d'accès
- Utile pour tables très sollicités
- Si arrêt serveur ⇒ Tout arrêter
- Utile pour compteur de visiteurs, système de chats...

**Comment choisir son moteur ?**

Lors de la création de la table, il faut mettre : 
```
CREATE TABLE compte
(
    -- liste des champs
)
ENGINE = InnoDB;
```

**Faire une transaction en SQL:**

```
-- on désactive l'autocommit, pour empêcher toutes les requêtes individuelles comme transaction.
SET autocommit = 0;
-- on lance la transaction
START TRANSACTION;
-- on effectue cette simple requête sur notre table compte
UPDATE compte SET montant = montant + 20000 WHERE nom = 'vendeur';
-- on valide la transact
COMMIT;
ou

-- cette requête retourne une erreur
UPDATE compte SET montant = montant - 20000 WHERE nom = 'machin';
-- on annule donc la transaction
ROLLBACK ;
````


**Faire une transaction en PHP:**

https://openclassrooms.com/courses/les-transactions-avec-mysql-et-pdo




