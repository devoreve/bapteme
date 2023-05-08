# MCD

Salut,

La relation entre les utilisateurs et les produits est une "One-to-One" (1-1 des 2 côtés). Ce qui voudrait dire qu'un utilisateur peut "liker" un seul et unique produit. De la même manière un produit ne pourrait être "liké" que par un seul et unique utilisateur.

On veut plutôt qu'un utilisateur puisse "liker" un ou plusieurs produits et qu'un produit puisse être "liké" par un ou plusieurs utilisateurs, c'est-à-dire une relation "Many-to-Many" (0-n des 2 côtés).

Le changement de cette relation va avoir un impact sur ton schéma de base de données puisqu'il va impliquer la création d'une table supplémentaire.

J'espère que mon explication t'a semblé claire.

En bonus voici une plateforme que j'utilise pour créer des MCD que je peux ensuite télécharger : [Aller vers le site](https://app.diagrams.net/).