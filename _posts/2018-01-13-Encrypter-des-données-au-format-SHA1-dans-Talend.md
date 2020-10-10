---
layout: post
title: Encrypter des données au format SHA1 dans Talend
tags: [talend, tuto, encryption, sha1, DigestUtils]
image: '/images/posts/SHA1.png'
---

## Objectif

Nous voulons pouvoir encrypter un flux de données au format SHA1

## Importation des librairies JAVA

Nous avons besoin d’importer une librairie JAVA : 

-     org.apache.commons.codec_1.3.0.vxxxxxx.jar

![alt](https://raw.githubusercontent.com/robinfabre/blog/gh-pages/images/posts/Sha1_Library.jpeg)

Pour cela nous utilisons les composants Talend [tLibraryLoad](https://help.talend.com/reader/ZndcSsDNtKg8FpNIRCdjag/mP7dUX1oRPLzvp9i~YXgPw) que nous mettrons à la suite du [tPrejob](https://help.talend.com/reader/tM3os~7PoPzBK28jtBNgCw/48fHCW9p63nTgR7HFkDAlg).



## Utilisation de DigestUtils

Nous utilisons un tJava.
Dans Advanced settings, nous importons la librairie à utiliser :

`import org.apache.commons.codec.digest.DigestUtils;`

Dans les Paramètres simples, voici un exemple de code pour appliquer le hashage SHA1 : 

`String concat = « Un.petit.test »;
String transfoSHA1 = DigestUtils.shaHex(concat);
System.out.println(« Result : «  + transfoSHA1);`

## Résultat

Après execution, nous obtenons le résultat suivant : 

`Démarrage du job test_hashage a 12:07 27/02/2019.
[statistics] connecting to socket on port 4003
[statistics] connected
Result : 3493d68fa6a5143b38f84cb7e3f63e7847c4dd89
[statistics] disconnected
Job test_hashage terminé à 12:07 27/02/2019. [Code sortie=0]`



[![](https://raw.githubusercontent.com/robinfabre/blog/gh-pages/images/posts/20191028195532-Wordmark_black_vector.jpg)](https://www.buymeacoffee.com/robinfabre)