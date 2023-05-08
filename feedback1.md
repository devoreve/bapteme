# Apprenant 1

Salut,
J'ai regardé ton code et il n'y a pas grand-chose à redire dessus, tu as réussi à implémenter toutes les fonctionnalités et le code est plutôt propre, c'est du bon travail.

Si je peux apporter ma pierre à l'édifice, je dirai que tu pourrais améliorer la lisibilité de ton code. En effet tu as beaucoup de *console.log* que tu utilises à des fins de test (et c'est une très bonne chose) mais une fois que tu as confirmé que tu récupères bien le résultat escompté, pense bien à les retirer pour pas que ça n'encombre trop le code.

C'est pareil pour les bouts de code que tu mets parfois en commentaire et qui sont probablement des anciennes versions de ce que tu as codé. Par la suite tu utiliseras un système de versioning (ou peut-être que tu en utilises déjà un) comme git et toutes les versions de ton code seront dans tous les cas conservées, tu n'auras donc pas besoin de les garder en commentaires. Là encore le but c'est de ne pas trop encombrer ton code, tu verras que ça te permet de programmer plus efficacement.

Dernière remarque concernant ton projet : il y a une petite erreur d'affichage lorsque tu effectues une recherche par élément (et que tu spécifies bien un élément), elle est due à un symbole "+" en trop dans une chaîne de caractères

```javascript
'Résultat de la recherche : ' + (searchedElement === 'null' ? ' sans élément' : + searchedElement)
```

sans ce symbole, le problème n'est plus.

J'espère que ces quelques conseils te seront utiles.