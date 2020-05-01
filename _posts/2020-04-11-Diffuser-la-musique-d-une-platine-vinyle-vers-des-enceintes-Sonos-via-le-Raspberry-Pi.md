---
layout: post
title: Diffuser la musique d'une platine vinyle vers des enceintes Sonos via le Raspberry Pi
tags: [sonos, raspberry, icecast, darkrice]
image: '/images/posts/8.jpg'
---

Pour éviter d'investir dans un Sonos Port à 449€, j'ai tenté de trouver comment mettre à contribution l'un des mes Raspberry Pi pour diffuser le son de ma platine vinyle vers ma Sonos Beam.

Pour cela, j'utilise la sortie USB de ma platine vinyle que je vais connecter à un port du Raspberry Pi.

## 1. Installation de Raspbian sur la carte SD

Dans mon cas, j'ai utilisé la dernière version de Raspbian pour le Raspberry Pi4.

Il est aussi possible d'utiliser la version de Raspbian Lite si on dédie un raspberry uniquement à cet usage.

Pour le tutoriel d'installation de Raspbian, je vous conseille de suivre celui du site officiel : 
https://www.raspberrypi.org/documentation/installation/installing-images/README.md



## 2. Connecter la platine vinyle au Raspberry

Une fois que votre Raspberry est opérationnel, connectez la platine à l'un des ports USB puis lancez le terminal.
Utilisez la commande  `arecord -l` pour vérifier si votre appareil est détecté : 

```
**** Liste des Périphériques Matériels CAPTURE ****
carte 1: CODEC [USB AUDIO  CODEC], périphérique 0: USB Audio [USB Audio]
  Sous-périphériques: 0/1
  Sous-périphérique #0: subdevice #0
```

Retenez bien le numéro de la carte, `1` dans mon cas. Vous aurez probablement le même mais on ne sait jamais. 

## 3. Anticiper les problèmes de volume
La plupart des platine vinyle USB n'ont pas de réglage du volume et la sortie audio est souvent "bridée".
Nous allons donc rajouter un contrôle du volume.\
Pour cela, nous créons le fichier  `/etc/asound.conf` (avec la commande `sudo nano /etc/asound.conf`) et nous lui mettons le contenu suivant :

```
pcm.dmic_hw {
    type hw
    card 1
    channels 2
    format S16_LE
}
pcm.dmic_mm {
    type mmap_emul
    slave.pcm dmic_hw
}
pcm.dmic_sv {
    type softvol
    slave.pcm dmic_hw
    control {
        name "Boost Capture Volume"
        card 1
    }
    min_dB -5.0
    max_dB 20.0
}
```

On sauvegarde le fichier avec ctrl+X suivi de O.

Ensuite on lance la commande suivante pour recharger l'état d'alsa et nous affichons le test de volume : 

```arecord -D dmic_sv -r 44100 -f S16_LE -c 2 --vumeter=stereo /dev/null```

Dans mon cas ces paramètres étaient suffisants mais il possible d'utiliser 
`alsamixer` pour affiner les réglages si le volume est trop bas.

## 4. Installation de Darkice
Exécutez les commandes suivantes :

```sh
sudo apt-get install libmp3lame0 libtwolame0

wget https://github.com/basdp/USB-Turntables-to-Sonos-with-RPi/raw/master/darkice_1.0.1-999_mp3%2B1_armhf.deb

sudo dpkg -i darkice_1.0.1-999_mp3+1_armhf.deb
sudo apt-get install -f
```

## 6. Installation de Icecast

```sh
sudo apt-get install icecast2
```

Sélectionnez `Yes` pour configurer Icecast. 
Vous pouvez si vous le souhaitez laisser tous les paramètres par défaut mais je vous conseille quand même de changer le mot de passe.

## 7. Configurer Darkice
Darkice va être le logiciel qui va enregistrer le son qui arrive de la platine vinyle via le port USB et va le transformer au format mp3.

Pour le configurer, il faut créer le fichier  `/etc/darkice.cfg` (avec la commande `sudo nano /etc/darkice.cfg`) et le remplir avec les éléments suivants:

```ini
[general]
duration        = 0      # duration in s, 0 forever
bufferSecs      = 1      # buffer, in seconds
reconnect       = yes    # reconnect if disconnected

[input]
device          = dmic_sv # Soundcard device for the audio input
sampleRate      = 44100   # sample rate 11025, 22050 or 44100
bitsPerSample   = 16      # bits
channel         = 2       # 2 = stereo

[icecast2-0]
bitrateMode     = cbr       # constant bit rate ('cbr' constant, 'abr' average)
#quality         = 1.0       # 1.0 is best quality (use only with vbr)
format          = mp3       # format. Choose 'vorbis' for OGG Vorbis
bitrate         = 320       # bitrate
server          = localhost # or IP
port            = 8000      # port for IceCast2 access
password        = raspberry    # source password for the IceCast2 server
mountPoint      = turntable.mp3  # mount point on the IceCast2 server .mp3 or .ogg
name            = Turntable
highpass        = 18
lowpass         = 20000
description	= Turntable
```

Attention à bien mettre mot de passe utilisé lors de l'installation de icecast à la ligne password (ici raspberry est celui par défaut).

## 8. Lancement automatique de Darkice

```bash
wget https://raw.githubusercontent.com/basdp/USB-Turntables-to-Sonos-with-RPi/master/init.d-darkice

sudo mv init.d-darkice /etc/init.d/darkice

sudo chmod 777 /etc/init.d/darkice

sudo update-rc.d darkice defaults
```

## 9. Vérifier si le streaming fonctionne

Redémarrez le Raspberry Pi dans le but de vérifier que tout fonctionne correctement  (commande `sudo reboot` dans le terminal). 

Une fois que le Raspberry à redémarré, lancé votre navigateur et connectez-vous à l'adresse suivante :  `http://ipDuRaspberry:8000`.

Pour récupérer le lien de streaming, il vous suffit de faire un clic droit sur le lien M3U et de copier l'adresse du lien.

![](/images/posts/darkrice.gif)



Ensuite pour faire un test, lancez un vinyle sur votre platine puis ouvrez VLC et allez dans :

`Fichier > Ouvir un flux réseau...`

Puis collez l'adresse du lien M3U.

Si tout a marché, vous devriez être en mesure d'entendre la musique via VLC.



## 10. Ajouter le stream à votre installation Sonos.

Pour ce faire, il vous faut obligatoirement la version de bureau de l'application Sonos (impossible de le faire sur la version mobile).

Dans l'application Sonos allez dans `Gérer > Ajouter une station de radio...`
et collez l'adresse du lien M3U dans la partie URL du flux puis donnez un titre à votre station (la mienne s'appelle Vinyle).

![](/images/posts/sonos.png)

Pour jouer la musique sur votre enceinte Sonos, il vous suffit d'aller ensuite dans `Radio via TuneIn > Mes stations de radio` puis de double cliquer sur le nom de la station (je vous conseille aussi de faire un `clic droit > Ajouter au favoris Sonos`).\


![](/images/posts/sonos2.png)

A partir de là, vous devriez être en mesure d'entendre le son de la platine sur votre enceinte Sonos.
Si vous avez rajouté la station à vos favoris, vous pourrez aussi lancer le streaming via l'application mobile.

![](/images/posts/sonosApp.jpeg)


Source : https://github.com/basdp/USB-Turntables-to-Sonos-with-RPi/blob/master/README.md


[![](/images/posts/20191028195532-Wordmark_black_vector.jpg)](https://www.buymeacoffee.com/robinfabre)