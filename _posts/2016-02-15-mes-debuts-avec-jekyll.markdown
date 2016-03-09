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

Vous pouvez télécharger ce conteneur directement depuis [mon DockerHub](https://hub.docker.com/r/kassovix/jekyll/).

## Utilisation de Jekyll

### Initialisation du blog

Dès le démarrage du conteneur, le site est disponible sur le port 4000. La commande (lancée par défaut par Docker) est la suivante :
{% highlight sh %}
jekyll serve
{% endhighlight %}

Pour initialiser un site (avec l'arborescence des dossiers + des fichiers d'exemple) dans le répertoire courant, il faut lancer :
{% highlight sh %}
jekyll new .
{% endhighlight %}

Les fichiers apparaissent également dans le répertoire Windows partagé avec le conteneur Docker.

### Rédaction du premier post

Pour rédiger un article, il suffit de s'inspirer de l'exemple présent dans le dossier __posts_, et de parcourir la [documentation officielle](https://jekyllrb.com/docs/posts/) pour apprendre la syntaxe. 
Vous pouvez également jeter un oeil au code source du présent article sur le [dépôt de ce blog](https://github.com/kassovix/kassovix.github.io).

**Avertissement /!\\** : Si vous avez rédigé un bel article, mais que celui ne s'affiche pas lors du rafraichissement du blog dans votre navigateur, ne désespérez pas tout de suite !
Pourtant, la commande _jekyll serve_ détecte normalement les fichiers modifiés et met à jour le site automatiquement, me direz-vous. 
Vrai, mais ici Docker apporte une limitation : les fichiers modifiés sous Windows ne sont pas détectés comme tels par Jekyll.

**Astuce \o/** : Un _touch_ dans la console du conteneur (à lancer depuis le lien EXEC de Kinematic par exemple) sur n'importe quel fichier lancera la mise à jour de votre site  :
{% highlight sh %}
touch index.html
{% endhighlight %}

### Personnalisation du format de la date

Par défaut, le format de la date est du type : _Feb 15, 2016_. Pas très frenchy, me direz-vous !

Pour utiliser un format plus francophone, il existe un plugin de localisation [_i18n_](https://github.com/gacha/gacha.id.lv/blob/master/_plugins/i18n_filter.rb), 
parmi la famille des [plugins Jekyll](http://jekyllrb.com/docs/plugins/), qui semble faire l'affaire (je ne l'ai pas testé).
Malheureusement, l'une des limitations de GitHub Pages est l'impossibilité d'utiliser un plugin...

Je me suis donc lancé à la recherche d'une solution, et je me suis inspiré de [cet article](http://christopheducamp.com/w/Jekyll-localiser-la-date#localisation_du_mois) pour créer un système de formattage de date personnalisé (cf ci-dessous). 
Afin que ce formatage soit réutilisable à plusieurs endroits du site, j'ai créé le fichier date-fr.html dans le répertoire __includes/_ avec le contenu suivant :
{% highlight liquid %}
{% raw %}
{% assign j = include.dateFR | date: "%w"  %}{% case j %}{% when '0' %}Dimanche{% when '1' %}Lundi{% when '2' %}Mardi{% when '3' %}Mercredi{% when '4' %}Jeudi{% when '5' %}Vendredi{% when '6' %}Samedi{% else %}{{ j }}{% endcase %}
{% assign d = include.dateFR | date: "%-d" %}{% case d %}{% when '1' %}{{ d }}er{% else %}{{ d }}{% endcase %}
{% assign m = include.dateFR | date: "%-m" %}{% case m %}{% when '1' %}Janvier{% when '2' %}Février{% when '3' %}Mars{% when '4' %}Avril{% when '5' %}Mai{% when '6' %}Juin{% when '7' %}Juillet{% when '8' %}Août{% when '9' %}Septembre{% when '10' %}Octobre{% when '11' %}Novembre{% when '12' %}Décembre{% endcase %} 
{{ include.dateFR | date: '%Y' }}
{% endraw %}
{% endhighlight %}

Je peux ainsi utiliser mon formatage avec la ligne suivante :
{% highlight liquid %}
{% raw %}
{% include /date-fr.html dateFR=post.date %}
{% endraw %}
{% endhighlight %}

**Astuce** : la date passée dans le paramètre _dateFR_ lors de l'include, est utilisable dans le fragment sous le nom _include.dateFR_.

### Tips d'hiver

Pour avoir la liste des langages qui peuvent être syntaxiquement colorés avec la balise `{% raw %}{% highlight %}{% endraw %}`, consulter la [page wiki](https://github.com/jneen/rouge/wiki/List-of-supported-languages-and-lexers) du projet _rouge_ (plugin de coloration utilisé par Jekyll) ou lancer en ligne de commande :
{% highlight sh %}
rougify list
{% endhighlight %}