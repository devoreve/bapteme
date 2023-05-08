# Apprenant 3

Salut,
Après analyse de ton code, tu trouveras ci-dessous quelques pistes d'améliorations.

## Détail de la carte

Tu arrives bien à récupérer la carte souhaitée (à partir son id) dans la base de données. Par contre au niveau de l'affichage du détail de cette carte, tu essaies de la parcourir avec une boucle. Or tu as récupéré une seule et unique carte, tu n'as pas donc pas besoin de faire une boucle. Garde bien en tête que tu as besoin d'une boucle lorsque tu as **plusieurs** éléments à parcourir. 

D'ailleurs bien nommer ses variables te permettront de plus facilement voir si tu dois parcourir ou non cet objet (je reviendrai sur ce point un peu plus loin). En l'occurence ici tu as appelé ta variable *oneCard* ce qui est un nom clair (tu indiques que tu n'as qu'une seule carte dans ta variable). En revanche fais bien attention de bien faire correspondre le nom que tu as choisi dans le contrôleur et le nom que tu utilises dans ta vue.

```javascript
// Depuis ton contrôleur tu transmets ici une variable oneCard à ta vue cardDetails 
response.render('main/cardDetails', {oneCard});
```

```javascript
<%# Depuis ta vue tu utilises le nom singleCard comme nom de variable au lieu de oneCard %>
for (const singleCard of card)
```

Quelques remarques supplémentaires par rapport au détail de la carte :
* l'image ne s'affiche pas car il y a l'extension qui est appelée 2 fois; lorsque les images ne s'affichent pas n'hésite pas à inspecter l'élément pour vérifier si lien vers l'image est correct (par exemple si tu inspectes ici le lien vers l'image, tu verras rapidement que l'extension apparaît 2 fois)
* les informations comme le niveau, l'élément, les valeurs (nord, sud, est, ouest) ont été affichées dans un seul élément *p*, ce serait plus facile à lire si tu les affichais dans un plusieurs éléments *p* ou encore mieux dans une liste *ul*

Pour finir sur la partie détail de la carte, attention au middleware que tu as configuré dans le fichier *index.js*.

```javascript
app.use(mainController.cardPage);
```

Avec ce middleware en place, cette action sera appelée lorsque tu utiliseras la fonction *next* dans tes contrôleurs. Or il n'y a à priori pas de raison de retourner sur cette page en plus de l'action se trouvant dans le routeur.
Par exemple si on essaie d'aller sur la page d'une carte qui n'existe pas, ce code là :

```javascript
if (oneCard){
    response.render('main/cardDetails', {oneCard});
}else{
    next();
}
```

te renvoie sur le prochain middleware (c'est ce que je fais la fonction [next](https://reflectoring.io/express-middleware/#understanding-the-next-function)), c'est-à-dire la page détail de la carte alors que l'on souhaite aller sur la page d'erreur 404. Tu n'as donc à priori pas besoin de mettre ce middleware.

## Recherche par élément

La page de recherche par élément s'affiche correctement. Par contre il y a quelques changements à apporter sur la page de traitement de la recherche. Regardons ensemble comment on peut améliorer ça.

```javascript
const element = request.params.element
```

Pour commencer attention à la façon dont tu récupères les données. Tu essaies de récupérer le paramètre d'une route (comme pour la route *router.get('/card/:id', mainController.cardPage)* qui dispose d'un paramètre *id*) mais ici il s'agit d'un paramètre de la chaîne de requête (query string). En effet ton formulaire te renvoie le résultat de la recherche dans l'url, par exemple http://localhost:1234/search/element?**element=glace**. Il faut donc récupérer le résultat de cette manière :

```javascript
const element = request.query.element;
```

Ensuite regardons la méthode qui te permet de récupérer les cartes correspondant à cette recherche.

```javascript
const result = await dataMapper.cardElement(element);
```

Tu essaies d'appeler une méthode *cardElement* de ton *dataMapper*. Or si tu regardes dans ton fichier *dataMapper* tu verras qu'il n'existe pas de méthode *cardElement*. Ton navigateur t'indiquera alors qu'il y a une erreur et qu'il ne trouve pas la méthode *cardElement*. En général quand tu reçois ce genre de messages d'erreurs c'est bien souvent que la fonction n'a pas été créée ou qu'il y a une erreur dans le nom. Vérifie donc bien que le nom est bien le même, au niveau de la déclaration et au niveau de l'appel.

D'ailleurs quand tu appelles cette méthode, tu lui passes un paramètre, l'élément (ce qui est correct puisque l'on veut les cartes de cet élément), mais ce paramètre n'a pas été précisé lors de la déclaration de la fonction :

```javascript
getCardByElement: async () => {...}
```

La fonction *getCardByElement* ne pourra donc pas récupérer l'élément que tu souhaites lui passer. Il faut ajouter ce paramètre dans la déclaration : 

```javascript
getCardByElement: async (element) => {...}
```

Cette méthode actuellement ne récupère que les cartes qui ont un élément, ce qui n'est pas tout à fait ce que nous voulons : on veut récupérer uniquement les cartes d'un élément bien défini (l'élément qui est passé en paramètre). Il faut donc modifier la requête SQL du *dataMapper* pour obtenir ce résultat, ce qui nous donne :

```js
queryResult = await database.query(`
	SELECT *
    FROM "card"
    WHERE "element" = $1`, [element]
);
```

Avec cette requête nous allons pouvoir récupérer toutes les cartes d'un élément donné. Mais que se passe t-il si l'on ne sélectionne aucun élément ? Dans ce cas particulier il faudrait renvoyer toutes les cartes qui n'ont pas d'élément. En programmation lorsque l'on a des cas particuliers à gérer, on utilise des conditions et ici c'est ce qu'on va faire. On modifie notre *dataMapper* pour qu'il puisse gérer les 2 cas (un élément en particulier ou pas d'élément) :

```javascript
let queryResult;
// L'élément existe ? On récupère les cartes de cet élément
if (element !== 'null') {
  const sql = `SELECT *
               FROM "card"
               WHERE "element" = $1`;
  queryResult = await database.query(sql, [element]);
} else {
  // L'élément n'existe pas ? On récupère toutes les cartes qui n'ont pas d'élément
  const sql = `SELECT *
               FROM "card"
               WHERE "element" IS NULL`;
  queryResult = await database.query(sql);
}
```

Maintenant la méthode dans notre contrôleur va nous permettre de récupérer les bons résultats. Il ne reste plus qu'à les afficher. Dans ton code, tu essaies d'afficher les informations de cette manière :

```javascript
const result = await dataMapper.getCardByElement(element);
if (result){
    response.render('main/element/', {element});
}
```

Attention ici ce n'est pas l'élément que tu veux afficher mais bien les résultats donc c'est ça qu'il faut transmettre à ta vue. 

Et d'ailleurs avant de se concentrer sur l'affichage, parlons un peu du nom des variables. En programmation la lisibilité du code est très importante pour s'y retrouver. "result" est un nom un peu générique qui ne t'aidera pas vraiment à savoir ce que tu as codé (quand tu reviendras sur ce code un peu plus tard et qu'il ne sera plus très frais dans ta tête). Il est fortement recommandé de nommer ses variables en fonction du contexte. Par exemple ici on récupère des cartes, on pourrait donc appeler notre variable "cards". 

Autre convention de développeur assez répandue : le "s" à la fin du nom de la variable. En mettant un "s" tu indiques implicitement que ta variable contient **plusieurs** cartes et qu'il faudra donc la parcourir (avec une boucle for). Lorsque l'on ne met pas de "s" à notre variable, c'est que cette dernière ne contient qu'une seule donnée. Par exemple si on avait appelé notre variable "card", ça aurait voulu dire que notre variable ne contenait qu'une seule carte (donc pas besoin de la parcourir).

Encore une fois ce sont des conventions de développeur, ton code ne va pas mieux marcher parce que tu les respectes mais ça te permettra de t'y retrouver mieux dedans et de coder plus efficacement. Essaie de les respecter le plus possible pour que ça devienne une habitude.

Revenons au projet. La vue que tu as créée ne permet pas encore d'afficher les résultats mais plutôt que de créer une autre vue, regardons si nous n'avons pas déjà un fichier qui s'occupe d'afficher une liste de cartes. Et en l'occurence nous en avons bien un : le fichier *main/cardList.ejs*. Ce fichier affiche une liste de cartes et en plus ça tombe bien le nom de la variable utilisée dans la vue s'appelle "cards" !

En programmation il est très important de voir s'il n'est pas possible de réutiliser quelque chose que l'on a déjà fait.

## Conclusion

J'espère que ces remarques t'auront permis d'y voir plus clair. N'oublie pas de toujours tester ton code petit bout par petit bout et d'abuser des console.log pour t'assurer que tu es sur la bonne voie.