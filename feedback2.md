# Apprenant 2

Salut,
Après analyse de ton code, tu trouveras ci-dessous quelques pistes d'améliorations.

## Détail de la carte

Tout d'abord la requête que tu utilises pour récupérer la carte à partir de son id est correcte. Néanmoins attention il faut que tu utilises les requêtes avec les paramètres. C'est-à-dire qu'au lieu de faire ceci

```javascript
const selectQuery = 
  `
  SELECT *
  FROM "card"
  WHERE "id" = ${id}
  `
;
const cardResults = await client.query(selectQuery);
```

il faut que tu optes pour cette syntaxe là :

```javascript
const selectQuery = 
  `
  SELECT *
  FROM "card"
  WHERE "id" = $1
  `
;
const cardResults = await client.query(selectQuery, [id]);
```

En apparence le code donne le même résultat (d'ailleurs ce que tu as fait fonctionne) mais il est important d'utiliser cette 2ème syntaxe pour des raisons de sécurité (pour éviter ce qu'on appelle des injections sql). Je te joins la page de la documentation qui explique ça et qui donne quelques exemples : [Aller vers la doc](https://node-postgres.com/features/queries#parameterized-query).

Maintenant qu'on sait comment récupérer les informations de la carte à partir de son id, voyons comment les afficher. Mais avant que se passe t-il si l'id ne correspond à aucune carte ? Si l'on tape un numéro dans l'url qui ne se trouve pas dans notre base de données ?

Actuellement lorsque l'on fait ça (tu peux d'ailleurs le tester de ton côté aussi) on se retrouve avec une erreur, ce qui est normal puisque l'on essaie d'afficher les propriétés d'une donnée qui est... null. Il faut donc gérer ce cas particulier avec une condition.

```javascript
const card = await dataMapper.getOneCard(id);

if (card) {
  // Si la carte a été trouvée on affiche ses informations
  res.render('card', { card });
} else {
  // Sinon on affiche une page 404
  res.status(404).send(`La carte d'id ${id} n'existe pas`);
```

Bien sûr tu peux créer ta propre page 404 pour que ce soit plus joli.

Enfin on peut passer à l'affichage ! Là-dessus pas grand-chose à dire, tu arrives bien à afficher les informations. On pourrait peut-être juste rajouter l'information "Aucun élément" lorsque la carte n'a pas d'élément (actuellement rien n'est affiché).

Modifions la ligne suivante

```ejs
<p class="card-item__element"><%= card.element %></p>
```

en 

```ejs
<p class="card-item__element"><%= card.element || 'aucun' %></p>
```

## Recherche par élément

Quand tu effectues une recherche par élément, tu récupères la liste de toutes les cartes avec un élément puis tu les filtres toutes en fonction de l'élément qui t'intéresse. Alors ça fonctionne mais on pourrait optimiser ça, en faisant une requête SQL qui récupère directement les cartes avec l'élément souhaité.

Modifions donc l'exécution de la requête SQL dans la méthode *getElements* du *dataMapper* qui donnerait ça :

```javascript
const query = 'SELECT * FROM card WHERE "element" = $1';
const result = await database.query(query, [element]);
```

Il faudrait donc passer l'élément récupéré dans le contrôleur en paramètre de cette méthode *getElements*. Avant d'aller plus loin, revenons rapidement sur le nom de cette méthode.

En programmation il est important de bien nommer ses variables et fonctions. Pourquoi ? Par soucis de lisibilité. Si tu reviens sur ton code un peu plus tard, tu ne sauras plus exactement ce que fait ta fonction et le nom peut t'y aider (et t'éviter de relire tout le code de la fonction). Ici tu as appelé ta fonction *getElements* qui signifierait en français "Récupérer tous les éléments" ce qui n'est actuellement pas ce que l'on fait. J'aurais donc plutôt tendance à lui donner un nom tel que *getCardsByElement* qui voudrait dire littéralement "Récupérer les cartes à partir de l'élément" ce qui exactement ce que l'on souhaite.

Bref garde bien en tête que nommer ses variables et fonctions le plus précisément possible te permettra de travailler mieux et d'y retrouver plus facilement. Bien sûr ça prendre un peu de temps pour que ce soit naturel mais essaie de le faire plus souvent possible pour prendre l'habitude.

Fermons cette parenthèse et continuons notre fonctionnalité. Nous avons maintenant une méthode qui nous renvoie la liste des cartes d'un élément donné mais si l'on recherche les cartes sans élément ? Là encore on peut les récupérer à l'aide d'une requête SQL.

```javascript
const query = 'SELECT * FROM card WHERE "element" IS NULL';
const result = await database.query(query);
```

Bien sûr on ne va pas faire les 2 en même temps, cela dépendra de la valeur de l'élément. Si ce dernier vaut "null" on récupérera toutes les cartes sans élément et si un élément est fourni on récupérera toutes les cartes de cet élément. On va donc utiliser une condition pour gérer ces 2 différents cas. Ci-dessous tu trouveras le code complet de notre méthode dans le *dataMapper*.

```javascript
async getCardsByElement(element) {
  let result;
  if (element === 'null') {
    const query = 'SELECT * FROM card WHERE "element" IS NULL';
    result = await database.query(query);
  } else {
    const query = 'SELECT * FROM card WHERE "element" = $1';
    result = await database.query(query, [element]);
  }
  return result.rows;
}
```

Maintenant qu'on arrive à récupérer la liste des cartes, il ne nous reste plus qu'à les afficher. Tu as créé une autre vue pour l'affichage mais en fait nous avons déjà un template qui s'occupe d'afficher la liste des cartes et donc nous allons le réutiliser. Il est important en programmation de toujours réutiliser ce que l'on peut, cela nous évite d'écrire du code en plus ET de gagner en maintenabilité (si tu veux changer la manière dont la liste des cartes s'affiche, il n'y a qu'un endroit où il faut changer le code).

Nous allons appeler la vue *cardList* en adaptant le titre en fonction de si un élément est fourni ou non. Pour ce faire on peut mettre en place une condition (sous forme de ternaire), ce qui donnerait ceci :

```javascript
res.render("cardList" , {
  cards: cards,
  title: 'Liste des cartes ' + (cardElement === 'null' ? "sans élément" : `de l'élément ${cardElement}`)
});
```

## Affichage du deck

L'affichage du deck fonctionne correctement mais encore une fois on pourrait réutiliser du code déjà existant et en particulier notre template qui s'occupe d'afficher une liste de cartes.

```javascript
// On vérifie si le deck existe dans la session au préalable
if (!req.session.deck) {
  req.session.deck = [];
}

res.render('cardList', {
    cards : req.session.deck,
    title: 'Liste des cartes du deck'
});
```

Et voilà, de nouveau on réutilise notre vue *cardList*. On fait toutefois au préalable une vérification pour s'assurer que le deck existe dans la session et sinon créer un tableau vide. D'ailleurs cette vérification est faite à plusieurs endroits (dans la méthode *addDeck* également) et on se retrouve alors avec du code dupliqué, ce qui n'est jamais une bonne chose. Pour éviter ça on va créer un middleware dans le fichier *index.js* qui s'occupera de faire cette vérification.

```javascript
app.use((req, res, next) => {
  // Si la session ne contient pas encore de tableau pour le deck
  // On en créé un vide
  if (!req.session.deck) {
    req.session.deck = [];
  }

  // On passe au middleware suivant
  next();
});
```

Grâce à ces quelques modifications, notre deck s'affiche correctement et de la même manière que les autres pages.

## Ajout d'une carte au deck

Ton code permet bien d'ajouter une carte au deck. Néanmoins on va essayer de l'optimiser un peu et de prévoir le cas où l'on essaie d'ajouter plus de 5 cartes.

Pour commencer voyons comment ne pas ajouter plus de 5 cartes. Pour cela il faut vérifier avant d'ajouter si la taille du tableau dans la session est inférieure à 5. On peut faire ça à l'aide de la propriété *length*.

```javascript
// Si on n'a pas encore 5 cartes dans le deck on peut ajouter
if (!foundDeckCard && req.session.deck.length < 5) {
  req.session.deck.push(deckCard);
}
```

Après avoir récupèré la carte associée à l'id depuis la base de données il faut en général vérifier que celle-ci existe bien et c'est ce que tu as essayé de faire. Par contre attention la fonction *next* appelle le prochain **middleware** qui a été configuré or tu n'en as pas après la route d'ajout de la carte au deck. Donc soit tu ajoutes après un middleware qui affiche une page 404 soit au lieu de la fonction *next*, tu utilises ce que j'ai montré un peu plus haut pour afficher une page 404 :

```javascript
res.status(404).send(`La carte d'id ${id} n'existe pas`);
```

Maintenant tu as tout ce qu'il faut pour faire fonctionner l'ajout de la carte au deck. On peut toutefois optimiser un petit peu notre code. Cette partie est un peu plus poussée donc c'est normal de ne pas forcément y avoir pensé. Le but est surtout de comprendre la refléxion derrière.

Ta méthode pour vérifier si une carte est présente dans le deck a été bien réalisée. On peut cependant optimiser le moment où tu l'appelles. Pour résumer voilà ce que tu fais comme étapes :
1. Récupération de la carte correspondant à l'id dans la base de données
2. Si l'id de la carte ne se trouve pas dans le deck on redirige vers la page du deck
3. Ajout de la carte

Mais si l'id de la carte n'a pas été trouvé (en utilisant ta méthode), on n'a pas besoin d'aller faire une requête SQL et on peut directement rediriger vers la page du deck (ça fait une requête SQL en moins).

Voici un exemple de comment on pourrait faire pour implémenter ça :

```javascript
const id = Number(req.params.id);
const foundDeckCard = req.session.deck.find((deck) => deck.id === id);

// Si l'id de la carte se trouve déjà dans le deck ou si on a déjà 5 cartes dans le deck
// On redirige vers la page du deck
if (foundDeckCard || req.session.deck.length >= 5) {
  res.redirect('/deck');
} else {
  // Sinon on peut récupérer la carte et l'ajouter dans la session
  const deckCard = await dataMapper.getOneCard(id);

  // On récupère la carte correspondante à l'id si elle existe
  // Si elle n'existe pas on envoie une page 404
  if (deckCard) {
    // Ajout dans la session puis redirection vers la page du deck
    req.session.deck.push(deckCard);
    res.redirect('/deck');
  } else {
    res.status(404).send(`La carte d'id ${id} n'existe pas`);
  }
}
```

Comme j'ai dit cette partie est un peu plus compliquée. Tu n'es pas obligé d'essayer d'avoir un code nickel tout de suite, tu peux revenir plus tard dessus pour essayer de l'optimiser (c'est ce qu'on appelle du **refactoring**). Et ça peut être un très bon exercice.

## Conclusion

Voilà j'espère que toutes indications te permettront d'y voir plus clair. N'hésite pas à tester très souvent ton code, même si tu n'as pas écrit grand-chose (plus il y a de code, plus c'est compliqué de voir d'où peuvent venir les éventuels problèmes).