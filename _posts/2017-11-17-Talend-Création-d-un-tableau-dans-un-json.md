---
layout: post
title: Création d’un tableau dans un json avec Talend
tags: [talend, tuto, json]
image: 
---

Dans ce petit tutoriel Talend, nous allons voir comment créer un tableau dans un JSON

### Données en entrée
Les données en entrée arrivent sous la forme suivante : 

| dossier | libelle | montant |  taxe| total | numero | flag |
|:--|:--|:--|:--|:--|:--|:--|
|  134| lib1 | 325 | 24 | 349 | 2 | flg1 |
|  134| lib2|  178| 13 | 191 | 7 | flg2 |

### Données attendues en sortie

Nous voulons obtenir le JSON suivant : 

`‌[
   {
      "libelle":"lib1",
      "montant":" 325",
      "taxe":" 24",
      "total":" 349",
      "numero":" 2",
      "flag":"flag1"
   },
   {
      "libelle":"lib2",
      "montant":" 178",
      "taxe":" 13",
      "total":" 191",
      "numero":" 7",
      "flag":"flag2"
   }
]`
	

### Construction du JSON

À la suite du composant des données d’entrée, nous utilisons un [tWriteJsonFields](https://help.talend.com/reader/tM3os~7PoPzBK28jtBNgCw/50PMKM8mz4A5fAL~FQcLHw).
Il faut ensuite cliquer sur « Configurer la structure JSON ».
Dans la nouvelle fenêtre, nous ajoutons un attribut au rootTag : 

![alt](https://raw.githubusercontent.com/robinfabre/blog/gh-pages/images/posts/talend_builJSONArray_step1.png)

Nous lui donnons le libellé « class »

![alt](https://raw.githubusercontent.com/robinfabre/blog/gh-pages/images/posts/talend_builJSONArray_step2.png)

Nous ajoutons ensuite la valeur « array » à ce libellé

![alt](https://raw.githubusercontent.com/robinfabre/blog/gh-pages/images/posts/talend_builJSONArray_step3.png)

![alt](https://raw.githubusercontent.com/robinfabre/blog/gh-pages/images/posts/talend_builJSONArray_step4.png)

Nous ajoutons ensuite un nouveau sous-élément nommé « element ».\

![alt](https://raw.githubusercontent.com/robinfabre/blog/gh-pages/images/posts/talend_builJSONArray_step5.png)

![alt](https://raw.githubusercontent.com/robinfabre/blog/gh-pages/images/posts/talend_builJSONArray_step6.png)

A ce sous-élément, nous ajoutons un nouvel attribut nommé type avec comme valeur par défaut « string ».

<iframe width="560" height="315" src="https://www.youtube.com/embed/T7zlYjAm5sg?controls=0" frameborder="0" allow="accelerometer; autoplay; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>

### Finalisation

Pour terminer la création du tableau, il suffit de prendre l’ensemble des colonnes et de les rajouter comme sous-élément de « element » puis définir ce dernier comme *Element boucle*

<iframe width="560" height="315" src="https://www.youtube.com/embed/lpKn1jiGzhE?controls=0" frameborder="0" allow="accelerometer; autoplay; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>

Ici, nous ne prenons pas la colonne *dossier* car c’est elle qui va ne servir à regrouper les données.

Nous allons modifier le schéma et créer la colonne de sortie (ici *sortie*).
Ensuite dans les options du composant nous allons regrouper les données sur le numéro de dossier.

Pour ce faire, dans le tableau *Group by*, nous sélectionnons *sortie* comme **Colonne de sortie** et *dossier* comme **Colonne d’entrée**.

Enfin nous cochons **Supprimer le nœud racine** pour supprimer la balise rootTag du résultat final.

Ci-dessous, l'ensemble des étapes :

<iframe width="560" height="315" src="https://www.youtube.com/embed/1pMVvpgfrcI?controls=0" frameborder="0" allow="accelerometer; autoplay; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>

Après exécution du job, nous obtenons bien le résultat suivant : 


`‌[
   {
      "libelle":"lib1",
      "montant":" 325",
      "taxe":" 24",
      "total":" 349",
      "numero":" 2",
      "flag":"flag1"
   },
   {
      "libelle":"lib2",
      "montant":" 178",
      "taxe":" 13",
      "total":" 191",
      "numero":" 7",
      "flag":"flag2"
   }
]`

### Job d'exemple

Ci-dessous le job d'exemple développé avec la version 7.0.1 de Talend.

[![](![alt](https://raw.githubusercontent.com/robinfabre/blog/gh-pages/images/posts/zipFile.jpg)](![alt](https://raw.githubusercontent.com/robinfabre/blog/gh-pages/files/Creation_d_un_tableau_dans_un_json.zip)


[![](http://www.blog.robinfabre.fr/content/images/20191028195532-Wordmark_black_vector.jpg)](https://www.buymeacoffee.com/robinfabre)
 
