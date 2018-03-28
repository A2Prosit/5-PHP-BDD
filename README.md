# DreamTeam :

    animateur : Émilien
    secrétaire : Fantou
    gestionnaire : Hugo
    scribe : Maxime

    
# Prosit 5 : PHP Objet et BDD


## Mots clés :

- PDO: php data object permet la co à un bdd
- BDD: représentation  structuré des données sous forme clé valeur
- SGBD: système de gestion de bases de données, logiciel permettant de stackoer et paratagr les infos dans une bdd
- Procédures stockées: requetes prédéfinie stockée dans la base dont on a juste à spécifier les paramètres d'entrées si elle en a
- Requêtes préparées : requête contenant des variables qu'on peut définir dans du code, échape automatiquement les arguments pour éviter les injections SQL 
- Transactions : opération qui ne sera complète qu'une fois le commit effectué
- SQL : structured query language, langage de prog utilisé pour manipuler des bases de données
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
-On peut lier un site à une BDD avec ASP.Net
-Des fonctions PHP sont prévues pour sécuriser l’accès aux données
-Les procédures stockées permettent d’augmenter la sécurité


## Plan d’action :

**Etudier:**

### PDO + concurrents

interface d'abstraction : DBA, dbx, ODBC, PDO permet d'interagir avec une base de donnée

PHP Data Object :connection à une BDD en PHP:

authentification:

PHP propose plusieurs moyens de se connecter à une base de données MySQL:

* L'extension mysql_ : ce sont des fonctions qui permettent d'accéder à une base de données MySQL et donc de communiquer avec MySQL. Leur nom commence toujours parmysql_. Toutefois, ces fonctions sont vieilles et on recommande de ne plus les utiliser aujourd'hui.

* L'extension mysqli_ : ce sont des fonctions améliorées d'accès à MySQL. Elles proposent plus de fonctionnalités et sont plus à jour.

* L'extension PDO : c'est un outil complet qui permet d'accéder à n'importe quel type de base de données. On peut donc l'utiliser pour se connecter aussi bien à MySQL que PostgreSQL ou Oracle.



Activer PDO
Normalement, PDO est activé par défaut. Pour le vérifier, faites un clic gauche sur l'icône de WAMP dans la barre des tâches, puis allez dans le menuPHP / Extensions PHPet vérifiez que php_pdo_mysqlest bien coché.

Pour se connecter à MySQL. Nous allons avoir besoin de quatre renseignements :

* le nom de l'hôte : c'est l'adresse de l'ordinateur où MySQL est installé (comme une adresse IP). Le plus souvent, MySQL est installé sur le même ordinateur que PHP : dans ce cas, on met la valeur localhost. Néanmoins, il est possible que notre hébergeur web vous indique une autre valeur à renseigner (qui ressemblerait à ceci :sql.hebergeur.com). Dans ce cas, il faudra modifier cette valeur lorsque nous enverons notre site sur le Web ;

* la base : c'est le nom de la base de données à laquelle vous voulez vous connecter. Dans notre cas, la base s'appelle  test . Nous l'avons créée avec phpMyAdmin dans le chapitre précédent ;

* le login

* le mot de passe 


code pour se co à une BDD via PDO
```PHP
<?php
try
{
	$bdd = new PDO('mysql:host=localhost;dbname=sandbox;charset=utf8', 'root', '');
}
catch (Exception $e)
{
        die('Erreur : ' . $e->getMessage());
}
?>
```
le premier paramètre est le DSN (Data Source Name)

on peut aussi créé une connexion persistante avec

```PHP
<?php
$dbh = new PDO('mysql:host=localhost;dbname=test', $user, $pass, array(
    PDO::ATTR_PERSISTENT => true
));
?>
```

fonctions utiles du PDO:

* beginTransaction : démare une transaction
* commit : valide une transaction
* exec : exécute une requête et retourne le nb de lignes affectées
* getAttribute : récup les attributs d'une connexion 
* inTransaction : verif si on est dans une transaction
* prepare : requête préparée
* query : exec une requete sql et retourne un jeu de résultat sous forme d'objet PDOStatement
* quote : protège une caîne pour l'utiliser dans une requête
* rollBack : anulle une transaction

fonction utiles des PDOStatement :

* execute :exécute une requête préparée
* fetch : récup la ligne suivante du jeu de donée
* fetchAll : retourne une tableau contenant tt les lignes d'un jeu d'enregistrements
* fetchObject : retourne le prochain enregistrement sous forme d'objet
* rowCount : retourne le nb de lignes affectées par le dernier appel de la fonction execute

#### query

```PHP
// On récupère tout le contenu de la table produits

$reponse = $bdd->query('SELECT * FROM produits');

// On affiche chaque entrée une à une
while ($donnees = $reponse->fetch())
{
?>
<p>
        le nom de l'utilisateur est
        <?php echo $donnees['nom']; ?> <br>
</p>

<?php
}
//on ferme 
$reponse->closeCursor();
?>
```
	

\$ reponse contenait toute la réponse de MySQL en vrac, sous forme d'objet.
\$donnees est un array renvoyé par le fetch(). Chaque fois qu'on fait une boucle fetch va chercher dans \$reponse l'entrée suivante et organise les champs dans l'array \$donnees
closeCursor() cloture la requête 

pour les requêtes préparées, procédures stockées et transactions cf la partie **type de requêtes BDD**

### SQL

SELECT : extrait des données
UPDATE : modif des valeurs
DELETE : suppr des données
INSERT INTO : ajouter des donées
CREATE DATABASE/TABLE : créé un BDD/table
ALTER DATABASE/TABLE : modif la structure d'un BDD/table
DROP DATABASE/TABLE : supp une BDD/table
CREATE INDEX créé un index
DROP INDEX : supp index
ORDER BY : trie ASC ou DESC

syntaxe:

selct

	SELECT el1,el2,el3 FROM table_name;
order

	ORDER BY col1, col2 ASC
insert into
		
	INSERT INTO table_name (col1, col2, col3) VALUES (val1,val2,val3)
	INSERT INTO table_name VALUES (val1, val2)

update

	UPDATE table_name SET col1= val1, col2=val2 WHERE condition

delete

	DELETE FROM table_name WHERE condition

count

	SELECT COUNT(column_name) FROM table_name WHERE condition;

average

	SELECT AVG(column_name) FROM table_name WHERE condition;
somme

	SELECT SUM(column_name) FROM table_name WHERE condition;

like: permet de faire des regex

	SELECT * FROM Customers WHERE CustomerName LIKE '%or%';

mots clés:

FROM : table source
WHERE : condition de selection, opérateurs : =,<> (not equal), >, <, >=, <=, BETWEEN, LIKE, IN
DISTINCT : ne retourne que les valeurs uniques
EXISTS
AS: créé un alias
AND, OR, NOT
IS (NOT) NULL
LIMIT
LIKE : permet les reg ex, % est le wildcard correspondant au . (0char ou +)  et _ 1 char ou +
IN : si la condition est dans le set donné (raccourci pour des ou multiples)

opérations :

### Types de requêtes BDD

#### gestion des erreurs

```PHP
<?php
$reponse = $bdd->query('SELECT nom FROM jeux_video') or die(print_r($bdd->errorInfo()));
?>
```
#### requête simple

```PHP
// On récupère tout le contenu de la table produits

$reponse = $bdd->query('SELECT * FROM produits');

// On affiche chaque entrée une à une
while ($donnees = $reponse->fetch())
{
?>
<p>
        le nom de l'utilisateur est
        <?php echo $donnees['nom']; ?> <br>
</p>

<?php
}
//on ferme 
$reponse->closeCursor();
?>
```

####requêtes préparées:
 permettent d'éviter les ejections SQL et plus rapide (comme ce qu'on avait fait en java)

on prépare la requête avec <code>prepare()</code> sans la partie variable qu'on représente avec un '?', puis on l'execute avec <code>execute</code>

```PHP
<?php
$req = $bdd->prepare('SELECT nom FROM jeux_video WHERE possesseur = ? AND prix <= ?');
$req->execute(array($_GET['possesseur'], $_GET['prix_max']));
?>
```

autre exemple avec un insert

```PHP
<?php
$req = $bdd->prepare('INSERT INTO jeux_video(nom, possesseur, console, prix, nbre_joueurs_max, commentaires) VALUES(:nom, :possesseur, :console, :prix, :nbre_joueurs_max, :commentaires)');
$req->execute(array(
	'nom' => $nom,
	'possesseur' => $possesseur,
	'console' => $console,
	'prix' => $prix,
	'nbre_joueurs_max' => $nbre_joueurs_max,
	'commentaires' => $commentaires
	));

echo 'Le jeu a bien été ajouté !';
?>
```


####transactions:

<code>PDO::beginTransaction()</code> désactive le mode autocommit. Lorsque l'autocommit est désactivé, les modifications faites sur la base de données via les instances des objets PDO ne sont pas appliquées tant que vous ne mettez pas fin à la transaction en appelant la fonction <code>PDO::commit()</code>. L'appel de <code>PDO::rollBack()</code> annulera toutes les modifications faites à la base de données et remettra la connexion en mode autocommit.

/!\ Sur MySQL la requête DROP TABLE valide automatiquement la transaction

```PHP
<?php
/* Démarre une transaction, désactivation de l'auto-commit */
$dbh->beginTransaction();

/* Modification du schéma de la base ainsi que des données */
$sth = $dbh->exec("DROP TABLE fruit");
$sth = $dbh->exec("UPDATE dessert
SET name = 'hamburger'");

/* On s'aperçoit d'une erreur et on annule les modifications */
$dbh->rollBack();

/* La connexion à la base de données est maintenant de retour en mode auto-commit */


/* Commence une transaction, désactivation de l'auto-commit */
$dbh->beginTransaction();

/* Insérer plusieurs enregistrements sur une base tout-ou-rien */
$sql = 'INSERT INTO fruit
    (name, colour, calories)
    VALUES (?, ?, ?)';

$sth = $dbh->prepare($sql);

foreach ($fruits as $fruit) {
    $sth->execute(array(
        $fruit->name,
        $fruit->colour,
        $fruit->calories,
    ));
}

/* Valider les modifications */
$dbh->commit();

/* La connexion à la base de données est maintenant de retour en mode auto-commit */
?>
```

###Types d’attaques et défenses possibles


injections SQL

2 types d'injection :

* injection dans les variables qui contiennent des chaines de caractères
* injection dans les variables numériques


protection strings :
mysql_real_escape_string()

protection num:
intval() retourne une val num tout le temps

**Réalisation:**   

- WS
