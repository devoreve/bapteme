# Fetch

Salut <Insérer nom de l'apprenant.e ici>,
*fetch* est une fonction javascript native qui te permet d'envoyer des **requêtes HTTP**.

Si tu n'es pas bien sûr.e de savoir ce qu'est une requête HTTP, je te redonne une petite explication. En fait lorsque tu vas sur un site web, ton navigateur (aussi appelé *client*) envoie une requête HTTP vers le serveur de ce site web qui lui te renvoie ce qu'on appelle une réponse. Cette réponse est souvent au format HTML mais ça peut être chose (image, json...). Et donc ce site web peut te renvoyer par exemple une réponse au format HTML qui correspond à la page d'accueil du site ou bien encore un formulaire quelconque. Et cette page en HTML va s'afficher dans ton navigateur. C'est ce qui se passe quand on navigue sur le web.

Maintenant dans le cas de *fetch*, le principe est le même : tu envoies une requête HTTP à un serveur qui va te fournir une réponse. Sauf que la réponse, au lieu de s'afficher dans ton navigateur, tu vas la récupérer dans une variable en *javascript*. Et comme elle se trouve dans une variable, tu peux en faire ce que tu veux : l'afficher, la stocker quelque part, afficher certaines des informations récupérées lorsqu'un événement se déclenche...

Je te donne un petit exemple pour illustrer tout ça.

Dans ton projet tu as créé une page de recherche qui te permet de trouver des cartes correspondants à un nom (ou plutôt des cartes dont la recherche est contenue dans leur nom). Nous allons réutiliser ça pour notre exemple.

L'objectif va être de récupérer les cartes correspondant au texte tapé mais la liste va s'actualiser à chaque fois qu'une lettre est saisie dans le champ de recherche.

Pour ce faire, sur la vue *search.ejs*, je créé une div vide d'id *search-result* puis un fichier *app.js*.

Dans le fichier *app.js* je mets ce code là :

```javascript
const inputName = document.querySelector("#name");
const searchResult = document.querySelector('#search-result');

// Lorsque l'on saisit quelque chose dans le champ texte
inputName.addEventListener('input', () => {
    // On envoie une requête vers la page de recherche de noms
    // avec le texte du champ de recherche
    fetch(`/search/name?name=${inputName.value}`).
     then(response => response.text()).
     then(response => {
         // Lorsque le serveur a renvoyé sa réponse (une liste de cartes au format html)
         // je l'intègre dans ma div
         searchResult.innerHTML = response;
     });
});
```

Maintenant tu peux tester : lorsque l'on saisit du texte dans le champ de recherche, la div se met automatiquement à jour (et sans rechargement de la page) avec la liste des cartes correspondant au texte tapé.

Ce qui se passe dans le cas classique (sans utiliser *fetch*), c'est qu'au moment où on soumet le formulaire ça envoie une requête vers la page qui affiche les résultats. Avec *fetch*, on envoie une requête vers la page qui affiche les résultats, on récupère ces résultats dans une variable et on intègre son contenu dans un élément HTML. Cela nous permet d'éviter de devoir soumettre le formulaire puis revenir sur la page pour effectuer une nouvelle recherche. Là on a quelque chose qui est très dynamique (mise à jour dès que l'on tape dans le champ) et sans rechargement de page (et donc pas de rechargement des fichiers css, js, images...).

Toute cette mécanique est ce qu'on appelle de l'**AJAX**. 

Avant d'utiliser *fetch* pour faire de l'AJAX, on utilisait une classe en js qui s'appelle *XMLHttpRequest* mais qui est plus compliqué à utilise et qui ne permettait pas d'utiliser les promesses (la syntaxe avec les "then" ou "async/await"). J'en profite pour te montrer ce que ça donne avec la version *async/await* si tu es plus familier avec cette écriture.

```javascript
inputName.addEventListener('input', async () => {
    const response = await fetch(`/search/name?name=${inputName.value}`);
    const html = await response.text();
    searchResult.innerHTML = html;
});
```

On retrouve également *fetch* lorsque l'on veut faire des appels vers des *api* pour récupérer des données tierces et les utiliser sur nos sites.

Voilà j'espère t'avoir donné une meilleure idée de ce qu'est la fonction *fetch*.