---
layout: post
title:  "Mes débuts avec Jekyll"
date:   2016-02-15 20:55:06 +0000
categories: jekyll update
---

Dans le but de mettre sur pied un blog utilisant GitHub Pages, j'ai voulu expérimenter le moteur de rendu de Jekyll. Pour ce faire, l'installation des pré-requis étant particulièrement ardu sous Windows, j'ai légèrement modifié le conteneur Jekyll officiel pour mon besoin (partagé sur [mon Docker Hub]). Je vais ici partager les étapes que j'ai mis en oeuvre.

## Environnement de développement 

### Installation de Docker
Pour l'installation et les premiers pas sous Docker, j'ai simplement suivi le [tutoriel officiel](https://docs.docker.com/windows/) pour Windows.

### Conteneur Jekyll
Sur le DockerHub, une simple recherche de Jekyll m'a amené sur le [conteneur officiel Jekyll](https://hub.docker.com/r/jekyll/jekyll/).
Malheureusement, celui-ci ne possède pas de Volume permettant de faire facilement le pont avec les fichiers Windows, comme on peut le constater dans Kitematic :

![Conteneur Jekyll sous Kitematic]({{site.baseurl}}/images/2016-02-21 21_43_14-Kitematic.png "Conteneur Jekyll sous Kitematic : pas de volume partagé")


### Personnalisation du conteneur
J'ai créé les Dockerfile suivant, pour ajouter un Volume au conteneur :
{% highlight docker %}

FROM jekyll/jekyll

VOLUME /srv/jekyll

{% endhighlight %}

J'ai donc la possibilité de partager le répertoire de travail entre le conteneur et Windows, qui est également le dépôt Git à pousser sur GitHub pour la sauvegarde (ainsi que la mise en ligne) :

![Conteneur kassovix/Jekyll sous Kitematic]({{site.baseurl}}/images/2016-02-21 22_01_14-Kitematic.png "Conteneur kassovix/Jekyll sous Kitematic : avec volume partagé")

## Initialisation du blog

1. Commandes Jekyll
jekyll serve

La mise à jour n'est pas automatique car la VM ne voit pas les modifications. Astuce :
touch index.html

Localiser la date des posts (sans plugin, car GitHub Pages...)
http://christopheducamp.com/w/Jekyll-localiser-la-date#localisation_du_mois

[tutoriel officiel]
[image officielle Jekyll]: 
[mon Docker Hub]: https://hub.docker.com/r/kassovix/jekyll/