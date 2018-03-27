
# DreamTeam :

    animateur : Émilien
    secrétaire : Fantou
    gestionnaire : Hugo
    scribe : Maxime

    
# Prosit 5 : PHP Objet et BDD


## Mots clés :

- PDO : PHP Data Object
- BDD :  Base de données
- SGBD : Système de gestion de base de données, logiciel système destiné à stocker & à partager des infos dans une BDD.
- Procédures stockées : Une procédure stockée est un ensemble d'instructions SQL pré compilées, stockées dans une BDD, et exécutées sur demande par le SGBD qui manipule la BDD.
- Requêtes préparées : /
- Transactions : /
- SQL : Structured Query Language, langage normalisé servant à exploiter les BDD relationnelles.
- Failles de sécurité : /
- Recherche par mots clés : /
- Trier les produits : /
- Maquette : /
- Modélisation : /
- Mise en relation site/BDD : /


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

## 1 - Les bases de la PDO
Deux moyens :
- mysqli_ (suite de mysql_) ⇒ mysqli_connect
- PDO


**L'avantage de PDO :**
PHP Data Object (PDO) est une interface d'abstraction (même fonctions pour exécuter des requêtes ou récupérer des données) pour accéder à une BDD depuis PHP. Le principe repose sur des pilotes de BDD dans l'interface de PDO qui peut utiliser des extensions de fonctions. PDO ne fournit cependant pas une abstraction à une base de données, il ne réecrit pas le SQL. PDO requiert PHP 5 au minimum.

![](https://user.oc-static.com/files/219001_220000/219973.png)

Se connecter à la base de données :
 ``` PHP
 try{
$bdd = new PDO('mysql:host=' ';dbname=' ';charset=utf8', 'login', 'password');
}
catch (Exception $e)
{
	die('Erreur : '.$e->getMessage());
}
```
mysql s'appelle le DSN : **D**ata **S**ource **N**ame - C'est celui qui change en fonction du type de BDD on se connecte.

**NB :** Si on fais des erreurs sur la connexion de la BDD, on peut mettre cette ligne sous la connexion pour plus d'infos : 
``` PHP
array(PDO::ATTR_ERRMODE => PDO::ERRMODE_EXCEPTION));
```


## 2 - Requêtes PDO :

### **Insertion classique**
``` PHP
$bdd->exec('INSERT INTO jeux_video(nom, possesseur, console, prix, nbre_joueurs_max, commentaires) VALUES(\'Battlefield 1942\', \'Patrick\', \'PC\', 45, 50, \'2nde guerre mondiale\')');
```
``` PHP
// Faire une requête SQL
$reponse = $bdd->query('SELECT nom FROM jeux_video');

// Parcourir les données reçus si plusieurs lignes
while ($donnees = $reponse->fetch())
{
	echo $donnees['nom'];//On va afficher tous les noms
}
```
Après chaque requête, il faut fermer les recherches avec "`closeCursor()`".


**Activer les détails sur des erreurs lors de requêtes**


- Même principe sur les requêtes : 
 ``` PHP
$reponse = $bdd->query('SELECT nom FROM jeux_video') or die(print_r($bdd->errorInfo()));
```

**Requête préparée**
C'est une requête qui est **préparée qu'une seule fois**, mais qui peut-être **exécutée plusieurs** fois avec des paramètres identiques ou différents. Lorsqu'une requête préparée est exécutée, la BDD va analyser, compiler et optimiser son plan pour exécuter la requête. Ce cycle est à faire qu'une seule fois, ce qui permet de **consommer moins de ressources** pour ses prochaines exécutions. Les paramètres n'ont **pas besoin d'être des guillemets**, ce qui signifie l'impossibilité d'y mettre des injections SQL. On peut ainsi lui passer des arguments à tout moment.
``` PHP
$req = $bdd->prepare('SELECT nom FROM jeux_video WHERE possesseur = ?');
$req->execute(array($_GET['possesseur']));
```
``` PHP
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
![](http://www.experience-developpement.fr/wp-content/uploads/2010/06/requete_preparee.png)

**NB** : 
-  **PDOStatement**: Représente une requête préparée et, une fois exécutée, le jeu du résultats associé, ce sont diverses fonctions qui permettent d’interagir avec la BDD.
- **PDO Exception** : Représente une erreur émise par PDO, il ne faut pas l'utiliser dans son code, elle est déjà utilisé nativement.

**Procédure stockés**
Une procédure stockée est un ensemble d'instructions SQL pré compilées, stockées dans une BDD, et exécutées sur demande par le SGBD qui manipule la BDD.

```PHP
<?php  
$stmt = $dbh->prepare("CALL sp_takes_string_returns_string(?)");  
$value = 'hello';  
$stmt->bindParam(1, $value, PDO::PARAM_STR|PDO::PARAM_INPUT_OUTPUT, 4000);  
  
// appel de la procédure stockée  
$stmt->execute();  
  
print "La procédure a retourné : $value\n";  
?>`
````
## 3 - Les transactions :
Exemple de la banque : *Si l'on souhaite faire un transfert en validant que l'argent a bien été reçu côté client après le retrait du côté banque (qui pourraient avoir une erreur à cause d'un serveur par exemple), il est nécessaire de vérifier toutes les données qui ont transités avant de **"sauvegarder"**.*

Repose sur un mode "transaction", en désactivant le mode "Auto Commit", càd que les modifications faites sur la BDD ne seront pas appliquées tant qu'on ne marquera pas la fin à l'aide de l'instruction "commit". A l'opposition, rollback(); permet de revenir au dernier commit et d'annuler tout les changements.

**Le principe d'une transaction [SQL] :**
``` PHP
//on désactive l'autocommit, pour empêcher toutes les requêtes individuelles comme transaction.
SET autocommit = 0;
//on lance la transaction
START TRANSACTION;
//on effectue cette simple requête sur notre table compte
UPDATE compte SET montant = montant + 20000 WHERE nom = 'vendeur';
//on valide la transact
COMMIT;

//ou

//cette requête retourne une erreur
UPDATE compte SET montant = montant - 20000 WHERE nom = 'machin';
//on annule donc la transaction
ROLLBACK ;
````
**NB :** MYSQL exécute automatiquement un commit lorsqu'une requête de type DROP TABLE ou CREATE TABLE est exécutée dans une transaction. Un COMMIT empêche de revenir en arrière.

**1 ) Les moteurs de stockage de MySQL - Nécessité d'un choix**

- **Mylsam** 
	- est le moteur par défaut de MySQL
	- très simple d'utilisation & très bonnes performances
	-  index FULL-TEXT (recherches faciles) comme "LIKE"

MAIS

Souffre sur les grosses tables, sa configuration (taille des mots) n'est accéssible que un serveur dédién ne supporte NI les clefs étrangères, ni les transactions.

- **InnoDB :**
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

**2) Mettre en place son moteur**

Lors de la création de la table, il faut mettre : 
``` PHP
CREATE TABLE compte
(
    //liste des champs
)
ENGINE = InnoDB;
```

**3) Faire une transaction en PHP:**

```PHP
<?php $pdo->beginTransaction(); //Débuter la transactions

/* Requêtes */
$pdo->query('SELECT * FROM machin WHERE bidule = \'truc\'');
$pdo->query('INSERT INTO machin SET bidule = \'truc\', chose = \'moi\'');
$pdo->query('UPDATE machin SET nombre = nombre + 1');

$pdo->commit();//Commit
$pdo->rollback();//A lever si erreur
?> 
```

## 4 - Injections SQL :

Il existe deux types d'injections SQL :
- Injection dans les variables qui contiennent des chaînes de caractères.
- L'injection dans les variables numériques.

### Chaîne de caractères :

Lors de l'utilisation de $_GET, l'utilisateur peut modifier l'url pour faire ce qu'il veut. Pour se protéger, il existe 
```PHP
mysql_real_escape_string() //NULL, \x00, \n, \r, \, ', " et \x1a, banalise les caractères spéciaux.
```
### Variables numériques
Lors de l'utilisation de $_POST, pour éviter qu'une variables soit changée en injection SQL, on cast le type :
```PHP
intval() ou is_numeric() //Deux méthodes pour s'assurer du type.
```

## 5 - Rappels sur le SQL :
- Select : Récupérer des infos dans une base de données
- Insert into : Ajout d'une entrée
- Update : Modification d'une ou plusieurs entrées
- Delete : Suppression d'une ou plusieurs entrées

- Where (filtre) / order by (tri) / limit (limiations) : Affiner ses requêtes.

## 5 - Mysqli ou PDO ? :

Désavantages Mysqli :
-  Ne supporte pas les requêtes préparées côtés clients

Désavantages PDO :
- Pas d'interface procédurale
- Ne supporte pas les requêtes non-bloquantes, asynchrones avec mysqlnd ?
- Ne supporte pas TOUTES les requêtes multiples

Les deux s'équivalent sur le reste des fonctionnalités.
