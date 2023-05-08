# Apprenant 4

Salut,
Revoyons ensemble ton code pour essayer de l'améliorer.

## Détail de la carte

Quand tu veux créer une page, la première chose est de configurer correctement la route. Voici la route que tu as mis en place pour le détail de la page :

```javascript
router.get('/card', mainController.cardPage);
```

Comme tu l'as bien compris, tu vas avoir besoin de l'id pour récupérer la carte. Or actuellement le chemin de la route ne nous permet pas de récupérer un id. On va donc lui rajouter un paramètre :

```javascript
router.get('/card/:id', mainController.cardPage);
```

Paramètre que l'on pourra ensuite récupérer dans ton *mainController* et sa méthode *cardPage*. Grâce à ce bout de code ci-dessous, on peut récupérer l'id qui se trouve dans l'url (et ce grâce à ce ":id" que l'on a mis dans la route).

```javascript
// N'hésite pas à faire un console.log() pour voir ce que tu récupères
const id = Number(req.params.id);
```

Maintenant qu'on a l'id de la carte, on va pouvoir le transmettre à ton *dataMapper*. En effet ton *dataMapper* a besoin de l'id pour trouver la carte correspondante. Et comme il ne connaît pas cet id, c'est à ton contrôleur de lui fournir. Comment fournir des informations extérieures à une méthode ? Grâce aux paramètres !

```javascript
// On appelle la méthode getCard et on lui passe l'id récupéré au préalable
const card = await dataMapper.getCard(id);
```

Ensuite dans ta méthode *getCard*, il faut bien penser à utiliser cet id qui a été transmis dans ta requête.

```javascript
const queryResult = await client.query(
  `SELECT * 
  FROM card 
  WHERE card.id = $1;
  `, [id]
);
```

Ta variable *id* ici va remplacer le paramètre "$1" dans la requête. Puis on récupère les résultats (attention à bien réutiliser le bon nom de variable, ici elle s'appelle *queryResult* et non *resultQuery*). En dehors de l'erreur sur le nom de variable, ton code pour récupérer les données est correct : on cherche bien à récupérer une seule carte.

Maintenant que l'on a récupéré la carte correspondant à l'id, il ne nous reste plus qu'à l'afficher. On affiche donc le template *card* et il faut lui passer les données récupérées. Pour cela il faut passer un 2ème paramètre en plus du nom du template, ce 2ème paramètre correspond aux données transmises à la vue.

```javascript
res.render('card',{
  card
});
```

Ici on appelle bien le template *card.ejs* et on lui fournit le contenu de nos données dans une variable qui va s'appeler *card*.

Dans la vue, pas besoin de faire de boucle, nous avons **une** seule carte à afficher. Une boucle est utile lorsque tu reçois plusieurs données (qui sont souvent stockées dans un tableau). On peut rajouter le niveau sur template à l'aide de la proprité *level* de notre variable *card*.

Attention que se passe t-il si l'id dans l'url ne correspond à aucune carte ? Si tu essaies de taper dans l'url une id n'existant pas (par exemple 12345), tu tomberas sur une erreur (car on essaie de lire une carte qui est nulle). Dans ce cas-là il faut penser à vérifier si la carte existe et renvoyer une page 404 si ce n'est pas le cas. Voilà comment il faut faire :

```javascript
if (card) {
  res.render('card',{
    card
  });
} else {
  res.status(404).send(`La carte d'id ${id} n'existe pas`);
}
```

Bien maintenant nous arrivons à récupérer le détail de la carte correctement. Mais on veut pouvoir y accéder qu'en tapant l'id dans l'url. Rajoutons sur la page de la liste des cartes le lien vers chacune de ces cartes. Sur la vue "cardList", dans le lien sur chacune des cartes, remplaçons le lien présent par celui-ci :

```ejs
<a href="/card/<%= card.id %>">
```

Et voilà on arrive bien à aller sur le détail d'une carte à partir de la page liste des cartes !

## Conclusion

J'espère que tout ça est maintenant un peu plus clair. Essaie de reprendre tout ça et fais des console.log à chacune des étapes pour voir ce que tu obtiens.