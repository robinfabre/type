---
layout: post
title: Endpoint REST Extraction d’un message de type form dans Talend
tags: [talend, tuto, webservice, rest, form]
image: '/images/posts/Vue_globale.png'
---

## Objectif

Nous voulons pouvoir extraire les données reçues par notre webservice qui sont de type x-www-form-urlencoded

## Données en entrée

Les données en entrée arrivent sous la forme suivante : 

`nom=John&prenom=Rambo&age=55`

## Construction du point d’entrée

Vue globale du point d’entrée : 

![alt](https://raw.githubusercontent.com/robinfabre/blog/gh-pages/images/posts/Vue_globale.png)

### tRESTRequest :

Nous créons pour notre exemple un point d’entrée de type REST.
Nous utilisons le composant tRESTRequest avec les paramètres suivants :
- Endpoint REST : "/talend/test"
- REST API Mapping : 
	- Output Flow : urlForm
		- Dans les options, on configure le schéma suivant : 
		
		![alt](https://raw.githubusercontent.com/robinfabre/blog/gh-pages/images/posts/tRestRequest_schema.jpeg)
		
 
		=> Il ne faut pas oublier de spécifier « form » dans la colonne Commentaire pour l’ensemble des champs à extraire.
		
	- HTTP Verb : POST
	- URI Pattern : "/form"
	- Consomme : Form
	- Produit : <oneway> (pour notre exemple, le WS ne retourne pas de réponse)
			![alt](https://raw.githubusercontent.com/robinfabre/blog/gh-pages/images/posts/tRestRequest_mapping.jpeg)


### tRunJob

Pour pouvoir configurer complètement ce composant, nous devons dans un premier temps configurer le job associer.

Dans notre exemple, le composant fait appel au job ws001_get_form.
Ce dernier est très simpliste est contient les composants suivants : 

- *tFixedFlowInput* : pour récupérer les paramètres reçus par le point d’entrée
- *tLogRow* : pour afficher dans la console les éléments récupérés
![alt](https://raw.githubusercontent.com/robinfabre/blog/gh-pages/images/posts/Subjob.jpeg)

#### Configuration des contextes : 

Dans ws001_get_form, nous devons configurer 3 variables de contextes qui contiendront les valeurs reçues : 
![alt](https://raw.githubusercontent.com/robinfabre/blog/gh-pages/images/posts/Subjob_context.jpeg)
Ensuite nous pouvons configurer le tFixedFlowInput : 
- Schéma :
![alt](https://raw.githubusercontent.com/robinfabre/blog/gh-pages/images/posts/Subjob_tFixedFlow_schema.jpeg)
- Composant :
![alt](https://raw.githubusercontent.com/robinfabre/blog/gh-pages/images/posts/Subjob_tFixedFlow_parameter.jpeg)
=> Ici, les valeurs des différentes colonnes vont prendre celles des variables de contexte

Nous en avons terminé avec ws001_get_form, nous pouvons donc revenir au tRunJob du point d’entrée.

Dans ce dernier, nous pouvons désormais configurer les paramètres : 

![alt](https://raw.githubusercontent.com/robinfabre/blog/gh-pages/images/posts/tRunJob_parameter.jpeg)

=> On coche Transmettre tout le contexte
=> Comme nous avons créé les variables de contexte dans ws001_get_form, ces dernières sont désormais accessibles dans le tableau Paramètre de contexte. 
Il nous suffit de les rajouter avec le petit « + » en bas du tableau.
Pour leur valorisation, il nous faut prendre le nom des colonnes présentes dans le flux entrant. 
Dans notre cas, cela sera :
- urlForm.nom
- urlForm.prenom
- urlForm.age

![alt](https://raw.githubusercontent.com/robinfabre/blog/gh-pages/images/posts/Endpoint_parameter.jpeg)

Enfin dans Advanced settings, nous cochons Propagate the child result to the output schema (bonne pratique). 


## Test du WS

Nous pouvons à présent exécuter le point d’entrée.
Il sera accessible à l’adresse suivante : *http://localhost:8090/talend/test/form*

Afin de pouvoir lui envoyer notre appel POST de type x-www-form-urlencoded, nous allons utiliser POSTMAN

Dans ce dernier, nous sélectionnons POST et nous renseignons l’url à droite.


Ensuite, nous nous rendons dans l’onglet Body puis sur x-www-form-urlencoded.
Nous renseignons ensuite le tableau KEY/VALUE avec les celles gérées par notre WS : 

![alt](https://raw.githubusercontent.com/robinfabre/blog/gh-pages/images/posts/Postman_step1.jpeg)

Une fois tout configuré, nous pouvons envoyer la requête avec le bouton Send.

Si tout est OK, nous aurons Status : 202 Accepted en bas à droite.

![alt](https://raw.githubusercontent.com/robinfabre/blog/gh-pages/images/posts/Postman_detail.jpeg)

Dans Talend, nous pouvons voir que ws001_get_form a bien reçu les données grâce à l’affichage du tLogRow : 

```
Démarrage du job ws000_point_entree_test a 11:09 28/02/2019.
[statistics] connecting to socket on port 3986
[statistics] connected
[WARN ]: org.eclipse.jetty.server.handler.AbstractHandler - No Server set for org.apache.cxf.transport.http_jetty.JettyHTTPServerEngine$1@78bbaf83
.----+------+---.
|   tLogRow_1   |
|=---+------+--=|
|nom |prenom|age|
|=---+------+--=|
|John|Rambo |55 |
'----+------+---'
```
