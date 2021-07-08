# Fr-VD

Un jeu de données de références de clips video, de videodescriptions (VD), et de métadonnées (scène, personnage, actions) 
en français pour l'apprentissage profond.
Accompagné d'un utilitaire de visualisation des clips vidéos, des vidéosdescriptions et des actions.

## Contexte

Au cours des dernières années le CRIM a développé une expertise dans la production et la diffusion de vidéodescriptions 
dans des œuvres audiovisuelles : nous avons ainsi vidéodécris 142 films ou téléséries en français 
en y ajoutant des métadonnées d'identification de scènes et de personnages.

Dans le cadre d'un partenariat avec le Fond d'accessibilité à la radiodiffusion (le FAR), nous avons voulu enrichir
ce corpus préalable pour construire un jeu de données en français utile à l'apprentissage profond pour la production de VD 
automatique et/ou pour la détection d'éléments visuels (scène, personnage, action)

Durant le projet, nous avons ainsi cherché à identifier les actions dans les clips vidéos 
puisque c'est cette information qu'il nous manquait pour créer le jeu de données. 
Pour ce faire, nous avons détecté les actions dans les clips vidéos via un modèle vidéo d'apprentissage automatique (SlowFast)
et dans les VDs (annotation manuelle et automatique).

En résumé le jeu de données contient : 
- les `références` aux clips vidéos ( film ou télésérie)
- par `intervalle de temps` : 
    - des annotations de `scènes` et de `personnages` (et _acteurs_)
    - la `vidédescriptions` rédigée manuellement en FR
    - des `annotations linguistiques` classiques (librairie Stanza) : segmentation de la VD en phrase, jetons, catégorie grammaticales,
    indice de position des jetons
    - des `annotations manuelles ou automatiques (précisé selon le cas) des actions et des entités (sujet, objet)` identifiées dans la VD
      en format IOB ( voir plus bas le cadre d'annotation)
    - les `actions du jeu de données AVA détectées par un modèle video SlowFast` (en anglais - 80 actions possibles)
    - les `actions du jeu de données AVA détectées en se basant sur les annotations de la VD` (plusieurs méthodes d'alignement sont proposées - voir plus bas)

## Annotations de la VD

Nous avons annoté les VD pour identifier les actions et les entités qui prennent part à l'action.
L'annotation a été faite manuellement sur 45 % du corpus puis les 55% restant sont le résultat d'une annotation 
automatique après utilisation d'un modèle bi-LSTM entraîné sur les données annotées manuellement.

### Cadre d'annotation
Sont identifiés : 
 - `Action` dénotée par un verbe, en distinguant : 
    - `cas général` : (ex : _Des visiteurs **marchent** dans le parc_); 
    - `verbe passif` : (ex : _Une serveuse **est projetée** sur une étagère de verres_); 
    - `verbe support` (dans ce cas, quel est le verbe de substitution)(ex : _faire la vaisselle_, _prendre une bouffée de cigarette_). 
 
- Attributs ajoutés  : 
    - `type négation`
    - `type verbe de substitution` (pour les verbes supports) ( _faire la vaisselle_ = _nettoyer_, _prendre une bouffée de cigarette_ = _fumer_). 


 - Entités impliquées dans l’action, en distinguant :
    - `sujet` 
    - `objet direct` 
    -` objet indirect essentiel` 
    en précisant pour chacun son type si c’est un `humain`, un `animal`, un `objet` ou un `concept`.
      

## Annotations d'actions depuis la vidéo

Implémentation à l’aide de https://github.com/facebookresearch/SlowFast

Code original de SlowFast, permettant d’utiliser des poids plus récents.  
 - SlowFast préentraîné sur Kinetics600
- SlowFast préentraîné sur AVA avec améliorations

Utilisation de detectron2 pour la reconnaissance d’actions par individu plutôt que globales

## Annotations d'actions depuis la VD

Nous avons fait correspondre les annotations d'actions de la VD avec le jeu d'action AVA ( 80 étiquettes). 
Pour ce faire, mous avons :
 - filtré et extrait les actions avec sujets humains (avec ou sans objet direct) depuis les annotations de la VD.
- normalisé (les objets directs humains = "_quelqu'un_" )
- obtenu au final environ ~9 000 actions uniques en français (Ex : _sortir, apercevoir QQUN_, etc.)

### Méthodes d'alignement

Pour les aligner avec le jeu d'action AVA, plusieurs méthodes ont été explorées, elles apparaissent chacune dans le jeu de données Fr-VD  :
- utilisation de **plongements lexicaux multilingues** : `Muse`, `fastText` et `SentenceBert`
- utilisation de **ressources lexicales** par augmentation de données (récupération de synonymes en traduisant le jeu d'actions AVA), en concaténant les ressources suivantes : 
      Dictionnaire électronique de synonymes, Wonef et WOlf  : `Resslex`
 - utilisation d'un **alignement fait manuellement** pour les 300 actions les plus fréquentes du corpus ( représente plus de 50% des occurences du corpus) : 
   - `manuel` : l'action alignée manuellement (est vide (`-`) si ne fait pas partie des 300 actions, est nul (`_`) quand l'alignement est impossible)
    - `prox` : le degré de proximité sémantique (`1` pour un alignement adéquat, `2` pour un alignement plus relâché , `0` si aucune action AVA ne correspond) 
 
    
### Gold de 6 films

Nous avons également procédé à un alignement manuel complet sur quelques films du corpus de genres très différents : 
 - films : _Bon cop, Bad cop_ (policier), _Le démantèlement_ (agriculture), _Ernest et Célestine_ (dessin animé avec des animaux)
- téléséries : _La vie en vert_ (écologie), _Habitat_ (construction), _Une galaxie près de chez vous_ (science-fiction loufoque)
Dans ces cas l'alignement est sous la catégorie `gold` avec le champ `prox` pour indiquer le degré de proximité sémantique.



## Fichiers




## Contributeurs et remerciements
Ces jeux de données ont été produits par Francis Charette-Migneault, Edith Galy, Lise Rebout et Mathieu Provencher du CRIM. 

Le projet a bénéficié du soutien financier du Fond d'accessibilité à la radiodiffusion (FAR). 

Les vidéodescriptions avaient été rédigées dans le cadre de projets passés bénéficiant 
du soutien de l'Office des personnes handicapées du Québec et du Programme de soutien, à la valorisation et au transfert (PSVT).

## Références

### FR-VD
Référence pour le jeu de données Fr-VD
```
@techreport{FrVD_TechicalReport,
  title       = "Projet FAR-VVD: Rapport final des travaux au 18e mois du projet",
  author      = "Francis Charette-Mignault, Edith Galy, Lise Rebout, Mathieu Provencher",
  institution = "Centre de recherche informatique de Montréal (CRIM)",
  address     = "405 Ogilvy Avenue #101, Montréal, QC H3N 1M3",
  year        = 2021,
  month       = jul
}
```

### Ressources utilisées pour la constitution de Fr-VD
- `SlowFast` : Feichtenhofer, Christoph, Haoqi Fan, Jitendra Malik, and Kaiming He. 2018. “SlowFast Networks for Video Recognition.” arXiv [cs.CV]. arXiv. http://arxiv.org/abs/1812.03982. https://github.com/facebookresearch/SlowFast/blob/master/MODEL_ZOO.md
- `Modèle Bi-LSTM-CRF` :   Gabriel Bernier-Colborne, Caroline Barrière, Pierre André Ménard :CRIM’s Systems for the Tri-lingual Entity Detection and Linking Task. TAC 2017 
- `Dictionnaire électronique de synonymes` : Ploux, S. & Victorri, B. (1998). Construction d’espaces sémantiques à l’aide de dictionnaires de synonymes. TAL- Traitement Automatique des Langues, 39-1, 161-182. https://crisco2.unicaen.fr/des/
- `Wonef` : Pradet, Quentin & Desormeaux, Jeanne & de Chalendar, Gaël & Danlos, Laurence. (2013). WoNeF : amélioration, extension et évaluation d'une traduction française automatique de WordNet. 
- `Wolf` : Sagot Benoît et Fišer Darja (2008). Building a free French wordnet from multilingual resources. In Ontolex 2008, Marrakech, Maroc
- `Stanza` : Qi, Peng, Yuhao Zhang, Yuhui Zhang, Jason Bolton, and Christopher D. Manning. 2020. “Stanza: A Python Natural Language Processing Toolkit for Many Human Languages.” arXiv [cs.CL]. arXiv. http://arxiv.org/abs/2003.07082. https://stanfordnlp.github.io/stanza/
- `Muse` : Conneau, Alexis, Guillaume Lample, Marc ’aurelio Ranzato, Ludovic Denoyer, and Hervé Jégou. 2017. “Word Translation Without Parallel Data.” arXiv [cs.CL]. arXiv. http://arxiv.org/abs/1710.04087 https://ai.facebook.com/tools/muse/
- `FastText` : Grave, Edouard, Piotr Bojanowski, Prakhar Gupta, Armand Joulin, and Tomas Mikolov. 2018. “Learning Word Vectors for 157 Languages.” arXiv [cs.CL]. arXiv. http://arxiv.org/abs/1802.06893. https://fasttext.cc/docs/en/crawl-vectors.html
- `SentenceBert` : Reimers, Nils, and Iryna Gurevych. 2020. “Making Monolingual Sentence Embeddings Multilingual Using Knowledge Distillation.” arXiv [cs.CL]. arXiv. http://arxiv.org/abs/2004.09813.   https://www.sbert.net/
