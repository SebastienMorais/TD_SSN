# TD_SSN

L'objectif de ce TD est de produire un script permettant d'interagir avec une base de donnée contenant des informations de plusieurs personnes (nom, prénom et numéro de sécurité social).
L'interaction avec la base de donnée consiste à récupérer les informations d'une personne et à réaliser des traitements dessus.
Le premier traitement que l'on souhaite réaliser consiste à itérer sur la base de donnée et tester si les numéros de sécurité social enregistrés sont valides.

# Consignes

Créez un dépôt TD_SSN sur Gitlab ou Github

Ajoutez moi à ce dépôt 

Publiez votre code et ses modifications dans le dépôt

# Exercice 1 - Créez un code Python analysant les numéros de sécurité social

Bien que nous souhaitions commencer par analyser la validité des Numéros de Sécurité Social (SSN), vous allez débuter le projet en créant :

- une classe `Personne` qui :

        - encapsule le nom, prénom et numéro de sécurité social d'une personne
        - (si besoin) possède des getters permettant d'accéder à ces valeurs
        - possède une méthode publique `has_valid_ssn` qui déclenche l'analyse d'une instance de la classe `SSN` et retourne `True` si le SSN est valide, `False` sinon

- une classe Ssn qui :

        - encapsule un numéro de sécurité social
        - possède une méthode publique `is_valid` qui retourne `True` si le SSN est valide, `False` sinon.

Un SSN est valide si :

- il a le bon nombre de caractère
- il est composé des bonnes valeurs possibles (cf image ci-dessous et disponible dans le dépôt `img/SSN.jpeg`)
- sa clef de contrôle est valide

Image wikipedia:
![SSN](https://user-images.githubusercontent.com/95085398/143722344-841fa617-5323-4c4d-a532-01c689d2d66a.jpeg)

Indication : **il serait bon d'avoir un ensemble de tests pour valider les différents cas d'utilisations**

# Exercice 2 - Utilisons Docker pour peupler une base de donnée MongoDB avec un code Python

Ici, nous allons jouer avec deux images Docker qui devrons pouvoir communiquer entre elles.
Pour cela, nous allons utiliser Docker pour créer un réseau que nous nommerons `mynet`

```
docker network create mynet
```

Commencez par lancer un premier conteneur basé sur l'image `mongo` que vous nommerez `mongodb` et que vous connecterez au réseau `mynet` (utilisez l'option `--network mynet`).
Ensuite, lancez un second conteneur basé sur l'image `python:alpine` que vous nommerez `python-mongo` et que vous connecterez au réseau `mynet`.
Afin d'éviter que le conteneur de l'image python ne se ferme automatiquement, il faudra utiliser le mode "foregroudn" (-t, -i ou -it).

Dans le conteneur `python-mongo` installez la paquet `pymongo` que nous allons utiliser pour remplir la base de donnée.
Utilisez l'interpréteur python du conteneur, pour peupler la base de données avec des informations (nom, prénom et SSNà en vous inspirant du code suivant :

```python
import pymongo
# Import de la classe MongoClient qui nous permettra de nous connecter a la base de donnees MongoDB
from pymongo import MongoClient

client = MongoClient(host="A_REMPLIR")

# Acces a la base de donnees "NOM_BASE_DE_DONNEES"
db = client["NOM_BASE_DE_DONNEES"]

# Acces a la collection "NOM_COLLECTION"
col = db["NOM_COLLECTION"]

# Ajout d'un 'fruit' dans la collection
fruit = {
        "nom": "banane",
        "couleur": "jaune"
}
res = col.insert_one(fruit)

# Verification de l'ajout
print('Le fruit {} a bien ete cree'.format(res.inserted_id))

# Localise et affiche le fruit
col.find_one()
```

En parallèle, assurez vous que la base de donnée à bien été remplie depuis le conteneur `mongo`.
Pour cela, interagissez avec celui-ci en exécutant la commande `mongo`.
Une fois réalisé, utilisez la base de donnée que vous défini depuis le conteneur python.
`use NOM_BASE_DE_DONNEES`
Ensuite, interoggez la base pour récupérer un élément qui compose la collection que vous avec définit.
`db.NOM_COLLECTION.findOne()`

# Exercice 3 - Créons un script Python peuplant la base de donnée et faisons en une image Docker

Ecrivez un script Python `seed_mongo.py` permettant de peupler la base de donnée.
Pour cela, vous allez utiliser les tests réalisés à l'Exerice 1 et ajouter des entrées valides et non valides.

Une fois le scrip écrit, il ne vous reste plus qu'à définir le Dockerfile associé.
Indication : Ce script doit conteneir :

- l'installation des dépendances nécessaires
- la copie des données nécessaires
- l'exécution du script `seed_mongo.py`

Maintenant, vous allez construire votre image en la taggant par `$DOCKERID/seed_mongo` (où DOCKERID est votre identifiant docker hub).
Puis, après vous être connecté à Docker Hub, vous allez publier votre image.

# Exercice 4 - Créons un script Python traitant la base de donnée

Ecrivez un script Python `ssn_checker.py` basé sur vos développement à l'Exercice 1.
Lorsque celui-ci est appelé, il doit se connecter à la base de données et regarder que les SSN sont valides.
Dans le cas où un SSN n'est pas valide, il doit afficher à l'écran le nom, prénom et SNN de la personne.

Note : pour itérer sur une collection obtenus via `pymongo`, vous pouvez utiliser le snippet suivant:

```python
fruits = col.find()
for fruit in fruits:
        nom = fruit["nom"]
        couleur = fruit["couleur"]
```

# Exercice 5 - Assemblons les différents conteneurs avec Docker Compose !

Votre fichier `docker-compose.yml` va se composer de trois services :

- `ssn_checker` : que vous allez construire à partir des sources de l'Exercice 4 (il vous faudra faire un nouveau Dockerfile dédié à ce conteneur)
- `mongodb` : que vous allez lancer à partir de l'image `mongo:latest` et qui dépendra de ssn_checker
- `seed_mongo` :  que vous allez lancer à partir de votre image `$DOCKERID/seed_mongo` (ou de celle d'un de vos camarades) et qui dépendra de `mongodb`

Note : dans le cas du conteneur `mongodb`, n'oubliez pas que le service `mongod` utilise le port 27017.

Vérifiez ensuite que tout cela fonctionne :
`docker-compose up --build`
et, depuis un autre terminal
`docker exec -it NOM_DU_CONTENEUR bash`
et enfin l'exécution de votre code python réalisant le traitement sur la base de donnée.

# Exercice 6 - Bonus 

Modifiez `ssn_checker` pour en faire une interface en ligne de commande (CLI).
Pour commencer vous pouvez recouvrir vos développements existant, i.e. ajouter une option "--check" permettant de regarder si des personnes ont des SSN invalides.
Ensuite vous pouvez proposer d'autres options :
- afficher les informations relatives à une personne à partir de son SSN (genre, age, ...)
- rechercher les personnes ayant une certain nom, prénom, genre, ...
- supprimer les personnes au SSN invalide dans la base de données
- ...
