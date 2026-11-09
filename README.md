# Athanor

Chaîne d'outils locale et rapide qui transforme un petit fichier source, même cassé par une faute de frappe, en code C équivalent, puis en graphe de flot de données (DFG) au format `.dot`.

> **Statut : projet en cours de conception.** Les dossiers sont créés mais vides, rien n'est encore implémenté.

## Principe

```
source (C, C++, Rust, Go…)  ──►  mend  ──►  to2c  ──►  nodus  ──►  graphe .dot
      (peut ne pas compiler)    répare    traduit    construit
                                          en C       le DFG
```

Chaque étape est un programme indépendant, qui lit sur l'entrée standard et écrit sur la sortie standard, donc composable en pipe :

```bash
mend --lang rs in.rs | to2c --lang rs | nodus > out.dot
```

## Les programmes

| Programme | Rôle | Entrée | Sortie |
|---|---|---|---|
| [`mend`](mend/) | Répare les erreurs de syntaxe simples (`;` oublié, parenthèse manquante, faute de frappe…) | source possiblement invalide | source corrigé |
| [`to2c`](to2c/) | Traduit le source corrigé en C avec la même logique | source valide | fichier `.c` |
| [`nodus`](nodus/) | Construit le graphe de flot de données du code C | fichier `.c` | fichier `.dot` |
| [`core`](core/) | Bibliothèque statique partagée (lexer, tokens, diagnostics, inférence du petit modèle) | | |

## Périmètre initial

Volontairement restreint pour rester rapide et fiable :

- uniquement de l'arithmétique simple (déclarations, affectations, opérateurs, `return`) ;
- pas d'appel de fonction, pas d'allocation mémoire, pas d'import ;
- tout fichier hors de ce sous-ensemble est **rejeté** avec une erreur explicite plutôt que mal traduit.

## Choix techniques

- **C++17**, bibliothèque standard uniquement, CMake.
- Exécutables **liés statiquement** : aucune installation ni dépendance pour l'utilisateur.
- La réparation combine des heuristiques et un petit modèle appris. Le modèle est entraîné en Python (hors livraison), puis son inférence est réécrite en C++ avec les poids embarqués dans l'exécutable.
- Les vrais compilateurs ne servent que pour les tests, pas à l'exécution.

## Structure du dépôt

```
Athanor/
├── core/    bibliothèque partagée
├── mend/    réparation syntaxique
├── to2c/    traduction vers C
└── nodus/   génération du DFG
```

## Feuille de route

1. `core` et `to2c` pour deux langages, sans IA, pour valider la chaîne de bout en bout.
2. `nodus` déterministe.
3. `mend` avec heuristiques, puis un benchmark de corruptions synthétiques.
4. Entraînement du modèle, export des poids, inférence en C++, comparaison au baseline.
