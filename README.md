# Utilisation de la médiathèque à travers l’API 

## Introduction

Le contexte du site est un restaurant qui souhaite présenter une série de photos correspondant à ces menus.
Ce dépôt est un tutoriel qui explique comment utiliser l'API de son site [Kiubi](http://www.kiubi.com) pour accéder aux données de la médiathèque.

Le premier exemple montre comment créer un slideshow en fond de page avec l'utilisation de plusieurs dossiers de la médiathèque.

Le deuxième exemple décrit un carrousel de photos issues d'un dossier de la médiathèque.

Enfin, le dernier exemple affiche une galerie photos également issue d’un dossier de la médiathèque.

## Prérequis

Ce tutoriel suppose que vous avez un site Kiubi et qu'il est bien configuré : 

 - l'API est activée
 - le site est en thème personnalisé, basé sur Shiroi

Il est préférable d'être à l'aise avec la manipulation des thèmes personnalisés. En cas de besoin, le [guide du designer](http://doc.kiubi.com) est là.

Ce tutoriel est applicable à tout thème graphique mais les exemples de codes donnés sont optimisés pour un rendu basé sur le thème Shiroi.


## Slideshow en arrière plan du site

L'objectif est d'intégrer un slideshow dans le fond de la page avec des images provenant d'un dossier de la médiathèque. Nous allons définir plusieurs dossiers d'images afin d'alimenter les slideshows.

Pour y arriver, on utilise ici plusieurs composants :

 - un type de billet du cms
 - le framework jQuery pour les manipulations javascript de base
 - le client Javascript API Front-office de Kiubi (qui est un plugin jQuery) pour récupérer les produits
 - un plugin jQuery pour gérer le slideshow
 - un fragment de template spécifique pour faciliter sa mise en place.

### Mise en place


#### Création d'un dossier par jour de la semaine dans la médiathèque

Pour stocker les différentes photos de chaque menu du jour, il faut créer un dossier par jour de la semaine dans la médiathèque. Cela permettra de mettre autant de photos que souhaitées, ainsi que d’en ajouter, retirer ou modifier très facilement. 

Déposez quelques photos dans chaque dossier.

#### Création d'un type de billet du CMS

Copier le fichier `theme/fr/billets/menu_du_jour/config.xml` dans votre thème pour définir notre type de billet :

<pre lang="xml">
&lt;?xml version="1.0" encoding="iso-8859-1"?> 
&lt;!DOCTYPE type SYSTEM "http://www.kiubi-admin.com/DTD/typesbillets.dtd">
	&lt;type tri="1">
		&lt;desc>Menu du jour&lt;/desc>
		&lt;listechamps>
			&lt;champ champ="titre" type="select" intitule="Jour">
				&lt;option value="Lundi">Lundi&lt;/option>
				&lt;option value="Mardi">Mardi&lt;/option>
				&lt;option value="Mercredi">Mercredi&lt;/option>
				&lt;option value="Jeudi">Jeudi&lt;/option>
				&lt;option value="Vendredi">Vendredi&lt;/option>
				&lt;option value="Samedi">Samedi&lt;/option>
				&lt;option value="Dimanche">Dimanche&lt;/option>
			&lt;/champ>
			&lt;champ champ="sstitre" type="text" intitule="Nom du menu" />
			&lt;champ champ="texte1" type="wysiwyg" intitule="Description" />
			&lt;champ champ="texte5" type="text" intitule="Clé API du dossier de la médiathèque pour les photos" />
		&lt;/listechamps>
	&lt;/type>
</pre>

#### Création d'une page avec un billet par jour de la semaine

Créez une page intitulée "Menus du jour" dans laquelle nous mettrons un billet par jour d’ouverture de notre restaurant afin d’y décrire nos menus du jour.

Il est également important de définir l’adresse de la page car elle sera utilisée dans le template afin de récupérer le contenu. On définit l’adresse de notre page à : `menus-du-jour` (c'est normalement déjà le cas).

![image](./docs/urlpage.png?raw=true)

Créez un billet de type "Menu du jour" pour chaque jour d'ouverture :

* Dans le champ `jour` : choisir le jour de la semaine

* Dans le champ `nom du menu` : indiquez uniquement le nom du menu

* Dans le champ `description` : décrivez les plats du menu

* Dans le champ `clé API` du dossier de la médiathèque nous mettons l’identifiant récupéré sur le détail du dossier correspondant :

![image](./docs/detail-dossier.png?raw=true)

Exemple de billet pour le Lundi :

![image](./docs/ex-billet.png?raw=true)

*Pensez à mettre votre identifiant de dossier et non celui de l'exemple*

#### Inclusions des bibliothèques javascript

Le plugin jQuery utilisé est Backstretch et est distribué sous licence MIT. Les sources sont disponibles à l’adresse suivante : [http://srobbin.com/jquery-plugins/backstretch/](http://srobbin.com/jquery-plugins/backstretch/)

On rajoute l’inclusion des deux fichiers grâce au module `Injection de code` et on met le code d’inclusion avant la balise `</head>`

<pre lang="javascript">
&lt;script type="text/javascript" src="{cdn}/js/kiubi.api.pfo.jquery-1.0.min.js">&lt;/script>
&lt;script type="text/javascript" src="//cdnjs.cloudflare.com/ajax/libs/jquery-backstretch/2.0.4/jquery.backstretch.min.js">&lt;/script>
</pre>
*Nous utilisons ici un cdn public tiers pour l’inclusion du plugin jQuery, vous pouvez également récupérer le fichier et le déposer dans votre thème puis l’inclure dans vos templates de pages.*

#### Création d'un fragment de template

C’est dans ce template que nous allons écrire le code javascript qui va utiliser l’API Kiubi et qui va afficher nos menus du jour avec le slideshow d’image.
Copiez le fichier `theme/fr/includes/menu_du_jour.html` dans votre thème.
Insérer le widget `Fragment de template` dans la mise en page de la page d'accueil et configurez le avec le template `menu_du_jour.html`

![image](./docs/miseenpage.png?raw=true)

Allez sur la page d'accueil pour visualiser le menu du jour avec la possibilité de changer de jour grâce au menu, et le slideshow des images de la médiathèque correspondant en fond de page.

### Explications

Examinons en détail le code HTML du fragment de template `menu_du_jour.html` :

<pre lang="javascript">
var menus = {
	'Lundi':	{files: [], active: false},
	'Mardi':	{files: [], active: false},
	'Mercredi': {files: [], active: false},
	'Jeudi':	{files: [], active: false},
	'Vendredi': {files: [], active: false},
	'Samedi':	{files: [], active: false},
	'Dimanche': {files: [], active: false}
};
</pre>

On définit une variable `menus` qui va stocker l’ensemble des données pour chaque billet affiché dans notre page. Le paramètre `files` pour chaque jour contiendra les urls des images à mettre dans le slideshow, et le paramètre `active` permettra de savoir si le billet est activé en backoffice, et donc à afficher.

#### Explications de la fonction "voir()

<pre lang="javascript">
function voir(d) {
	$('#menu-list span').remove();
	$('#menu-list a').removeAttr('style');
	$('#menu'+d).attr('style', 'color:red;').after('&lt;span>'+menus[d].titre+'&lt;/span>');
	$('#desc-menu').html(menus[d].desc);
	if(menus[d].files.length == 0) {
		kiubi.media.getFiles(menus[d].key).done(function(meta, data){
			for(var i=0; i&lt;data.length; i++) {
				menus[d].files.push(data[i].url);
			}
			slideshow(d);
		});
	} else {
		slideshow(d);
	}
}
</pre>

Cette fonction affiche un menu du jour lorsque le jour est sélectionné. Le paramètre `d` va contenir le jour de la semaine à afficher. 
On commence par retirer les éléments ajoutés lors de la sélection du jour précédent, puis on ajoute le nom du plat `menus[d].titre` à la suite du nom du jour : 
<pre lang="html">
$('#menu-list span').remove();
$('#menu-list a').removeAttr('style');
$('#menu'+d).attr('style', 'color:red;').after('&lt;span>'+menus[d].titre+'&lt;/span>');
</pre>
On change ensuite la description du menu précédent par celle du menu du jour choisit :
<pre lang="javascript">$('#desc-menu').html(menus[d].desc);</pre>

Ensuite, on teste si on a déjà chargé les images du jour sélectionné :
<pre lang="javascript">if(menus[d].files.length == 0) {</pre>
Si elles ne sont pas chargées, on fait appel à l’API pour récupérer les images du dossier, et on les stocke dans la variable afin d’économiser une nouvelle requête si le visiteur réaffiche le même menu du jour :

<pre lang="javascript">
kiubi.media.getFiles(menus[d].key).done(function(meta, data){
	for(var i=0; i&lt;data.length; i++) {
		menus[d].files.push(data[i].url);
	}
	slideshow(d);
});
</pre>
Une fois les images récupérées, ou si elles l’étaient déjà, on affiche le slideshow :
<pre lang="javascript">slideshow(d);</pre>

#### Explications de la fonction "slideshow()"
<pre lang="javascript">
function slideshow(d) {
	if($('body').data('backstretch')) { 
		$.backstretch('destroy');
	}
	$.backstretch(menus[d].files, {duration: 3000, fade: 750});
	$('#page').css('background-color', 'rgba(255, 255, 255, 0.8)');
	$('#main').css('background-color', 'rgba(255, 255, 255, 0)');
}
</pre>
Cette méthode va créer un slideshow dans le background du site avec les images récupérées du dossier de la médiathèque pour le jour sélectionné.
Lors d’un changement de menu, on supprime l’ancien slideshow :
<pre lang="javascript">
if($('body').data('backstretch')) { 
	$.backstretch('destroy');
}
</pre>
Puis on le crée avec les fichiers stockés dans notre variable `menus` :
<pre lang="javascript">$.backstretch(menus[d].files, {duration: 3000, fade: 750});</pre>
Pour un bel effet de transparence sur notre page, on réduit l’opacité du fond du conteneur de la page :
<pre lang="javascript">$('#page').css('background-color', 'rgba(255, 255, 255, 0.8)')</pre>

#### Explications de la partie du code suivant :
<pre lang="javascript">
$(function(){
	kiubi.cms.getPostsOfPage('menus-du-jour', {extra_fields:'texts'}).done(function(meta, data){
		if(!data.length) return;
		var jour = {1:'Lundi', 2:'Mardi', 3:'Mercredi', 4:'Jeudi', 5:'Vendredi', 6:'Samedi', 0:'Dimanche'};
		var li = '', nb = 0, dt = new Date(), d = dt.getDay(), h = dt.getHours();
		for(var i=0; i&lt;data.length; i++) {
			if(data[i].type != 'menu_du_jour' || !menus[data[i].title]) continue;			
			menus[data[i].title].titre = data[i].subtitle;
			menus[data[i].title].key = data[i].text5;
			menus[data[i].title].desc = data[i].text1;
			menus[data[i].title].active = true;
			li += '&lt;li>&lt;a id="menu'+data[i].title+'" href="javascript:voir(\''+data[i].title+'\');">'+data[i].title+'&lt;/a>&lt;/li>';
			nb++;			
		}
		$('#menu-list').append(li);
		if(h>14 &amp;&amp; nb>1) {
			do { d = ++d % 7; } 
			while(menus[jour[d]].active == false);
		}
		voir(jour[d]);
	});		
});
</pre>
Ce code est exécuté lorsque le DOM est chargé car il est inclus dans la fonction suivante :
<pre lang="javascript">$(function(){
...
}) ;</pre>

On commence par récupérer les billets de notre page grâce à l’API et la méthode `cms.getPostsOfPage()` du client API :
<pre lang="javascript">kiubi.cms.getPostsOfPage('menus-du-jour', {extra_fields:'texts'}).done(function(meta, data){</pre>
Noter l’importance de la correspondance entre l’adresse de la page en backoffice et le paramètre défini dans notre appel à l’API : `menus-du-jour`
On ajoute le paramètre `{extra_fields:'texts'}` afin de récupérer les champs texte1 et texte15 qui contiennent la description et surtout la clé API du dossier des médias pour le jour en question.
La méthode `done()` est appelée lorsque la requête vers l’API est résolue.

Dans le cas où aucun billet n’est publié, on arrête le script ici.
<pre lang="javascript">if(!data.length) return;</pre>

On définit des variables nécessaires au traitement suivant. La liste des jours va permettre de sélectionner le jour courant par défaut lors du chargement de la page, le nombre relatif à chaque jour dépend de l’objet natif javascript `Date`.
<pre lang="javascript">
var jour = {1:'Lundi', 2:'Mardi', 3:'Mercredi', 4:'Jeudi', 5:'Vendredi', 6:'Samedi', 0:'Dimanche'};
var li = '', nb = 0, dt = new Date(), d = dt.getDay(), h = dt.getHours();
</pre>

Ensuite on itère sur les billets reçus, si on a un billet qui n’est pas de type `Menu du jour` ou que le billet n’a pas un titre défini dans notre variable `menus` (d’où l’importance de bien faire correspondre le nom du jour en backoffice avec le libellé du jour dans le code), on n’en tient pas compte :

<pre lang="javascript">
for(var i=0; i&lt;data.length; i++) {
	if(data[i].type != 'menu_du_jour' || !menus[data[i].title]) continue;
</pre>

On stocke ensuite les informations de chaque billet dans notre variable `menus` au niveau de la définition de chaque jour :

<pre lang="javascript">
menus[data[i].title].titre = data[i].subtitle;
menus[data[i].title].key = data[i].text5;
menus[data[i].title].desc = data[i].text1;
menus[data[i].title].active = true;
</pre>
On construit ensuite notre liste de jours pour l’affichage du menu :

<pre lang="javascript">
li += '&lt;li>&lt;a id="menu'+data[i].title+'" href="javascript:voir(\''+data[i].title+'\');">'+data[i].title+'&lt;/a>&lt;/li>';</pre>
et enfin on incrémente la variable `nb` qui nous indique le nombre de billets de menu que nous avons :

<pre lang="javascript">nb++;</pre>

Une fois que tous les billets ont été traités, on affiche le contenu du menu dans la liste prévue pour : 
<pre lang="javascript">$('#menu-list').append(li);</pre>
Petite fonctionnalité supplémentaire, on vérifie si l’heure courante est supérieure à 14h ou si le menu du jour courant n'est pas actif, dans ce cas on affiche le menu du jour suivant :

<pre lang="javascript">
if(nb>0) {
	$('#menu-list').append(li);
	if(h>14 || menus[jour[d]].active == false) {
		do { d = ++d % 7; } 
		while(menus[jour[d]].active == false);
	}
	voir(jour[d]);
}
</pre>

On vérifie si le menu est disponible et on passe au jour suivant, en revenant au premier si on atteint la fin de la semaine, jusqu’à trouver un jour où le menu est disponible.
Cela permet de ne pas afficher de menu pour chaque jour tout en gardant le même fonctionnel à savoir afficher le menu suivant si l’heure du repas du midi est déjà passé.

Une fois que le jour à afficher est déterminé, il ne reste plus qu’à afficher ses informations :

<pre lang="javascript">voir(jour[d]);</pre>
	
#### Explications de la partie du code suivant :

<pre lang="html">
&lt;h2>Exemple 1 : Slideshow en arri&egrave;re-plan&lt;/h2>
&lt;div style="height: 200px;">
    &lt;ul class="sf-menu" id="menu-list">&lt;/ul>
    &lt;div style="padding-top: 30px;" id="desc-menu">&lt;/div>
&lt;/div>
</pre>
On ajoute un titre pour annoncer notre exemple, une liste qui va contenir les jours disposants d’un menu, et enfin un élément pour afficher la description du menu du jour sélectionné. Les ID des éléments sont utilisés pour gérer leurs contenus.

### Exemple complet

<pre lang="html">
&lt;script type="text/javascript">
var menus = {
	'Lundi':	{files: [], active: false},
	'Mardi':	{files: [], active: false},
	'Mercredi':  {files: [], active: false},
	'Jeudi':	{files: [], active: false},
	'Vendredi':  {files: [], active: false},
	'Samedi':	{files: [], active: false},
	'Dimanche':  {files: [], active: false}
};
function voir(d) {
	$('#menu-list span').remove();
	$('#menu-list a').removeAttr('style');
	$('#menu'+d).attr('style', 'color:red;').after('&lt;span>'+menus[d].titre+'&lt;/span>');
	$('#desc-menu').html(menus[d].desc);
	if(menus[d].files.length == 0) {
		kiubi.media.getFiles(menus[d].key).done(function(meta, data){
			for(var i=0; i&lt;data.length; i++) {
				menus[d].files.push(data[i].url);
			}
			slideshow(d);
		});
	} else {
		slideshow(d);
	}
}
function slideshow(d) {
	if($('body').data('backstretch')) { 
		$.backstretch('destroy');
	}
	$.backstretch(menus[d].files, {duration: 3000, fade: 750});
	$('#page').css('background-color', 'rgba(255, 255, 255, 0.8)');
	$('#main').css('background-color', 'rgba(255, 255, 255, 0)');
}
$(function(){
	kiubi.cms.getPostsOfPage('menus-du-jour', {extra_fields:'texts'}).done(function(meta, data){
		if(!data.length) return;
		var jour = {1:'Lundi', 2:'Mardi', 3:'Mercredi', 4:'Jeudi', 5:'Vendredi', 6:'Samedi', 0:'Dimanche'};
		var li = '', nb = 0, dt = new Date(), d = dt.getDay(), h = dt.getHours();
		for(var i=0; i&lt;data.length; i++) {
			if(data[i].type != 'menu_du_jour' || !menus[data[i].title]) continue;			
			menus[data[i].title].titre = data[i].subtitle;
			menus[data[i].title].key = data[i].text5;
			menus[data[i].title].desc = data[i].text1;
			menus[data[i].title].active = true;
			li += '&lt;li>&lt;a id="menu'+data[i].title+'" href="javascript:voir(\''+data[i].title+'\');">'+data[i].title+'&lt;/a>&lt;/li>';
			nb++;			
		}
		$('#menu-list').append(li);
		if(nb>0) {
			$('#menu-list').append(li);
			if(h>14 || menus[jour[d]].active == false) {
				do { d = ++d % 7; } 
				while(menus[jour[d]].active == false);
			}
			voir(jour[d]);
		}
	});		
});
&lt;/script>
&lt;h2>Exemple 1 : Slideshow en arri&egrave;re-plan&lt;/h2>
&lt;div style="height: 200px;">
    &lt;ul class="sf-menu" id="menu-list">&lt;/ul>
    &lt;div style="padding-top: 30px;" id="desc-menu">&lt;/div>
&lt;/div>
</pre>

## Carrousel d'images

L'objectif est d'afficher un carrousel contenant des photos issues d'un dossier de la médiathèque en utilisant l'API au travers d'un fragment de template.

Pour ce faire nous allons utiliser le plugin jQuery NivoSlider, ce plugin est réalisé sous licence MIT et est disponible à l’adresse suivante : [http://dev7studios.com/nivo-slider](http://dev7studios.com/nivo-slider) 
Nous incluons ce plugin dans le module injection de code de Kiubi :

<pre lang="javascript">
&lt;script type="text/javascript" src="{cdn}/js/kiubi.api.pfo.jquery-1.0.min.js">&lt;/script>
&lt;script type="text/javascript" src="//cdnjs.cloudflare.com/ajax/libs/jquery-backstretch/2.0.4/jquery.backstretch.min.js">&lt;/script>
&lt;script type="text/javascript" src="//cdnjs.cloudflare.com/ajax/libs/jquery-nivoslider/3.2/jquery.nivo.slider.pack.js">&lt;/script>
&lt;link rel="stylesheet" href="//cdnjs.cloudflare.com/ajax/libs/jquery-nivoslider/3.2/nivo-slider.min.css" type="text/css" media="screen" />
</pre>
*Nous utilisons ici un cdn public tiers pour l’inclusion du plugin jQuery, vous pouvez également récupérer le fichier et le déposer dans votre thème puis l’inclure dans vos templates de pages.*

### Mise en place

#### Création d'un dossier dans la médiathèque

Nous créons un dossier dans la médiathèque et y déposons les photos qui seront affichées dans le carrousel. On récupère l’identifiant API du dossier que nous utiliserons plus tard
 
![image](./docs/dossiercarrousel.png?raw=true)

#### Création d'un fragment de template

Copiez le fichier `theme/fr/includes/carrousel.html` dans votre thème puis insérez le widget `Fragment de template` dans la mise en page de votre page.

Editez le fichier et remplacer en ligne 3 l'identifiant `16-4a3a734ce116522` par celui de votre dossier. 

Allez sur la page de votre site, le carrousel s'affiche et les images du dossier défilent.

### Explications

Examinons en détail le code HTML du fragment de template `carrousel.html` :

* Au chargement de la page, on récupère les médias de notre dossier grâce à l’identifiant de notre dossier : `16-4a3a734ce116522` et on demande de récupérer les urls des vignettes en plus des informations de base.
La fonction `done()` est appelée lorsque la requête est terminée et que les données des médias sont récupérées.

	*Pensez à mettre votre identifiant de dossier et non celui de l'exemple*
	
<pre lang="javascript">
jQuery(function($){
	kiubi.media.getFiles('16-4a3a734ce116522',{extra_fields:'thumb'}).done(function(meta, data){
</pre>
 
* On parcours l’ensemble des médias et on crée une image pour chaque média qui est ajoutée à l’élément ayant l’id `slider`.
On utilise la description de l’image dans l’attribut `title` qui sera affiché comme caption au bas de l’image dans le carrousel. On définit l’url de la miniature dans un attribut `data-thumb` afin que le plugin génère une navigation avec des miniatures.

<pre lang="javascript">
for(var i=0; i&lt;data.length; i++) {
	$('#slider').append('&lt;img src="'+data[i].url+'" alt="" title="'+data[i].description+'" data-thumb="'+data[i].thumb.url_miniature+'"/>');
}
</pre>

* Une fois les images ajoutées dans la page, on déclenche le carrousel avec l’appel au plugin nivoSlider.
On définit des paramètres de configuration en masquant les liens de navigation suivant/précédent, une transition entre les images à 1 seconde et on ajoute une navigation entre images par miniatures.
La liste complètes des options disponibles dans le plugin est disponible à cette adresse : [http://dev7studios.com/nivo-slider#/documentation](http://dev7studios.com/nivo-slider#/documentation)

<pre lang="javascript">
$('#slider').nivoSlider({
	directionNav: false,
	animSpeed: 1000,
	controlNavThumbs: true
});
</pre>

* On affiche un titre au dessus de notre carrousel et on définit notre élément conteneur dans lequel seront placées les images.

<pre lang="html">
&lt;h2>Exemple 2 : Carrousel de photos&lt;/h2>
&lt;div class="slider-wrapper">
	&lt;div id="slider">&lt;/div>
&lt;/div>
</pre>

### Exemple complet

<pre lang="html">
&lt;script type="text/javascript">
jQuery(function($){
	kiubi.media.getFiles('16-4a3a734ce116522', {extra_fields:'thumb'}).done(function(meta, data){
		for(var i=0; i&lt;data.length; i++) {
			$('#slider').append('&lt;img src="'+data[i].url+'" alt="" title="'+data[i].description+'" data-thumb="'+data[i].thumb.url_miniature+'"/>');
		}
		$('#slider').nivoSlider({
			directionNav: false,
			animSpeed: 1000,
			controlNavThumbs: true
		});
	});
});	
&lt;/script>
&lt;h2>Exemple 2 : Carrousel de photos&lt;/h2>
&lt;div class="slider-wrapper">
	&lt;div id="slider">&lt;/div>
&lt;/div>
</pre>

## Galerie photos

Toujours sur le même principe, nous allons ajouter une galerie photos basée sur un dossier de la médiathèque. Les fichiers seront récupérés via l'API et un plugin jQuery se chargera du rendu final.

### Mise en place

Nous allons utiliser le plugin jQuery Masonry disponible sous licence MIT à l’adresse suivante : [http://masonry.desandro.com/](http://masonry.desandro.com/) 
Nous incluons ce plugin dans le module injection de code de Kiubi :

<pre lang="javascript">
&lt;script type="text/javascript" src="{cdn}/js/kiubi.api.pfo.jquery-1.0.min.js">&lt;/script>
&lt;script type="text/javascript" src="//cdnjs.cloudflare.com/ajax/libs/jquery-backstretch/2.0.4/jquery.backstretch.min.js">&lt;/script>
&lt;script type="text/javascript" src="//cdnjs.cloudflare.com/ajax/libs/jquery-nivoslider/3.2/jquery.nivo.slider.pack.js">&lt;/script>
&lt;link rel="stylesheet" href="//cdnjs.cloudflare.com/ajax/libs/jquery-nivoslider/3.2/nivo-slider.min.css" type="text/css" media="screen" />
&lt;script type="text/javascript" src="//cdnjs.cloudflare.com/ajax/libs/masonry/3.1.1/masonry.pkgd.min.js">&lt;/script>
</pre>
*Nous utilisons ici un cdn public tiers pour l’inclusion du plugin jQuery, vous pouvez également récupérer le fichier et le déposer dans votre thème puis l’inclure dans vos templates de pages.*

#### Création d'un dossier dans la médiathèque

Nous créons un dossier dans la médiathèque et y déposons les photos qui seront affichées dans le carrousel. On récupère l’identifiant API du dossier que nous utiliserons plus tard
 
![image](./docs/dossiergalerie.png?raw=true) 

#### Création d'un fragment de template

Copiez le fichier `theme/fr/includes/galerie.html` dans votre thème puis insérer le widget `Fragment de template` dans la mise en page de votre page.

Editez le fichier et remplacer en ligne 3 l'identifiant `17-3bb845eaf8fbcbc` par celui de votre dossier. 

Allez sur la page de votre site, la galerie photos s'affiche, et un clic sur une image agrandit l'image en reposionnant les autres.

### Explications

Examinons en détail le code HTML du fragment de template `galerie.html` :

* Au chargement de la page, on récupère les médias de notre dossier grâce à l’identifiant de notre dossier : `17-3bb845eaf8fbcbc` , on demande à récupérer les urls des vignettes en plus des informations de base et on récupère 25 fichiers à la fois, à configurer en fonction du nombre de vos images.
La fonction `done()` est appelée lorsque la requête est terminée et que les données des médias sont récupérées.

	*Pensez à mettre votre identifiant de dossier et non celui de l'exemple*


<pre lang="javascript">
jQuery(function($){
	kiubi.media.getFiles('17-3bb845eaf8fbcbc', {extra_fields:'thumb', limit: 25}).done(function(meta, data){
</pre>

* On parcours l’ensemble des médias et on crée un conteneur pour chaque média qui est ajouté à l’élément ayant l’id `galerie`. Ce conteneur est stylé afin d'avoir le média en fond et on y ajoute deux attributs pour avoir une miniature de l’image ainsi que l’image en taille réelle. On définit la classe `item` sur chaque conteneur, cette classe sera utilisée par le plugin afin de gérer les élements de la galerie.

<pre lang="javascript">
for(var i=0; i&lt;data.length; i++) {
	$('#galerie').append('&lt;div class="item" style="background-image:url('+data[i].thumb.url_g_miniature+')" data-mini="'+data[i].thumb.url_g_miniature+'" data-max="'+data[i].thumb.url+'" />&lt;/div>');
}
</pre>

* Une fois la structure de notre galerie en place, on déclenche le plugin avec `masonry()` en y spécifiant des paramètres : la classe de chaque élément de la galerie, la largeur d’une colonne et l’espacement entre les colonnes.

<pre lang="javascript">
$('#galerie').masonry({
	itemSelector: '.item',
	columnWidth: 140,
	gutter: 10
});
</pre>

* On ajoute une gestion particulière au clic sur une image afin d’avoir un affichage plus grand de l’image. On commence par réinitialiser les éléments de notre galerie pour remettre chaque miniature en place, puis sur l’image cliquée on ajoute la classe css `big` et on change la source de l’image par l’url stockée dans l’attribut `data-max`
Enfin, on relance le plugin sur notre galerie afin que l’affichage s’adapte avec la grande image.

<pre lang="javascript">
$('.item').click(function(){
	$('.item').removeClass('big');
	$('.item').each(function(){
		$(this).css('background-image', 'url('+$(this).data('mini')+')');
	});
	$(this).addClass('big');
	$(this).css('background-image', 'url('+$(this).data('max')+')');
	$('#galerie').masonry();
});
</pre>

* On définit du css pour spécifier la taille de chaque élément de notre galerie, dans son format miniature et son format agrandi.

<pre lang="css">
.item {
	width: 140px;
	height: 105px;
	margin-bottom: 10px;
	background-size: 100% 100%;
}
.item.big { width: 590px; height: 450px; max-width: 100%; }
</pre>

* On affiche un titre au dessus de notre galerie et on définit notre élément conteneur dans lequel seront placées les images.

<pre lang="html">
&lt;h2>Exemple 3 : Galerie Photos&lt;/h2>
&lt;div id="galerie">&lt;/div>
</pre>

### Exemple complet 

<pre lang="html">
&lt;script type="text/javascript">
jQuery(function($){
	kiubi.media.getFiles('17-3bb845eaf8fbcbc', {extra_fields:'thumb', limit: 25}).done(function(meta, data){
		for(var i=0; i&lt;data.length; i++) {
			$('#galerie').append('&lt;div class="item" style="background-image:url('+data[i].thumb.url_g_miniature+')" data-mini="'+data[i].thumb.url_g_miniature+'" data-max="'+data[i].thumb.url+'" />&lt;/div>');
		}
		$('#galerie').masonry({
			itemSelector: '.item',
			columnWidth: 140,
			gutter: 10
		});	
		$('.item').click(function(){
			$('.item').removeClass('big');
			$('.item').each(function(){
				$(this).css('background-image', 'url('+$(this).data('mini')+')');
			});
			$(this).addClass('big');
			$(this).css('background-image', 'url('+$(this).data('max')+')');
			$('#galerie').masonry();
		});
	});
});	
&lt;/script>
&lt;style>
.item {
    width: 140px;
    height: 105px;
    margin-bottom: 10px;
	background-size: 100% 100%;
}
.item.big { width: 590px; height: 450px; max-width: 100%; }	
&lt;/style>
&lt;h2>Exemple 3 : Galerie Photos&lt;/h2>
&lt;div id="galerie">&lt;/div>
</pre>