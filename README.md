# LLM-TP
# Assistant RAG pour la documentation interne d'Altéa Services

Projet de TP : un petit assistant qui répond aux questions des salariés à partir de la documentation interne de l'entreprise (fictive), et qui dit toujours de quel document vient la réponse.

## Objectif

Un salarié pose une question (congés, VPN, notes de frais...). L'assistant cherche les passages les plus proches dans les documents, les donne à un modèle de langage, et affiche la réponse avec ses sources. Si l'information n'est pas dans les documents, il le dit au lieu d'inventer.

## Comment ça marche

```
Documents (RH, IT, General)
   -> collecte -> nettoyage -> découpage -> embeddings -> base vectorielle (ChromaDB)

Question -> embedding -> recherche des 3 chunks les plus proches -> prompt -> LLM -> réponse + sources
```

Choix faits :

| Élément | Choix |
|---|---|
| Embeddings | paraphrase-multilingual-MiniLM-L12-v2 (vecteurs de 384 nombres, bon pour le français) |
| Base vectorielle | ChromaDB, distance cosinus |
| Découpage | 500 caractères, chevauchement de 50 |
| Recherche | k = 3, filtre possible par service |
| LLM | Qwen2.5-1.5B-Instruct, sans échantillonnage (équivalent de temperature = 0) |
| Interface | Gradio |

Chaque chunk garde son fichier, sa page et son service. On s'en sert pour citer les sources et pour filtrer.

## Installation

Le plus simple est d'utiliser Google Colab, il n'y a rien à installer sur son ordinateur.

1. Ouvrir [colab.research.google.com](https://colab.research.google.com) et importer `TP_RAG_Altea_Colab.ipynb`.
2. (Facultatif) Activer un GPU : Exécution > Modifier le type d'exécution > T4 GPU. Sans GPU, remplacer le modèle par `Qwen/Qwen2.5-0.5B-Instruct` dans la cellule de l'étape 6.
3. Lancer les cellules.

En local, il faut Python 3.10 ou plus, puis :

```
pip install pypdf chromadb sentence-transformers reportlab gradio transformers torch
```

## Ordre de lancement

On exécute les cellules du notebook dans l'ordre :

1. Installation des paquets
2. Création des 15 documents fictifs (données tirées au hasard)
3. Étape 1 : collecte
4. Étape 2 : nettoyage
5. Étape 3 : découpage
6. Étape 4 : embeddings et indexation
7. Étape 5 : recherche
8. Étape 6 : génération
9. Étape 7 : évaluation
10. Étape 8 : interface web

Si on modifie ou ajoute un document, il faut relancer les étapes 1 à 4 pour reconstruire la base. Le dossier `base_vectorielle/` n'est pas sur GitHub (voir `.gitignore`), on le recrée avec l'étape 4.

## Exemple

Question : « Combien de jours de congés payés par an ? »

Réponse attendue : le nombre de jours indiqué dans le document, suivi de la source `[conges_absences.pdf, p.1]`.

Question sans réponse dans les documents : « Comment faire pour postuler ? » L'assistant répond : « Je n'ai pas trouvé cette information dans la documentation interne. »

## Résultats de l'évaluation (étape 7)

Test sur 10 questions dont on connaît la réponse, avec k = 3.

| Mesure | Résultat |
|---|---|
| Le bon document est dans les 3 premiers résultats | ... / 10 |
| Réponses justes | ... / 10 |
| Réponses avec la bonne source citée | ... / 10 |

Amélioration testée : ... (par exemple passer k de 3 à 5).
Résultat après : ... / 10 de réponses justes.

## Limites connues

- Le modèle utilisé est petit : il lui arrive de ne pas respecter le format de citation, ou de répondre à côté alors que le bon extrait est présent.
- Le découpage se fait au nombre de caractères, donc une phrase peut être coupée en deux malgré le chevauchement.
- Le nettoyage ne gère que l'en-tête « Altéa Services » et les numéros de page de nos documents. Un autre format demande d'adapter les expressions régulières.
- Le filtre par service est choisi par l'utilisateur. Ce n'est pas une vraie gestion des droits d'accès.
- Les documents sont fictifs et courts. Les résultats ne disent pas grand-chose sur un vrai corpus d'entreprise.
- Si un document change, il faut réindexer à la main.

## Avant une vraie mise en service

Droits d'accès selon le salarié, données personnelles dans les documents, mise à jour automatique de l'index, et choix de l'hébergement du modèle (local ou cloud) avec son coût.

## Auteurs

[Nom 1], [Nom 2]
