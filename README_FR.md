# Fr-VD

> This content is also [available in English](README.md).

## Table des matières

- [Table des matières](#table-des-matières)
- [Description du projet](#description-du-projet)
  - [Contexte](#contexte)
- [Détails du jeu de données](#détails-du-jeu-de-données)
  - [Annotations de la vidéodescription](#annotations-de-la-vidéodescription)
    - [Cadre d'annotation](#cadre-dannotation)
  - [Annotations d'actions depuis la vidéo](#annotations-dactions-depuis-la-vidéo)
  - [Annotations d'actions depuis la VD](#annotations-dactions-depuis-la-vd)
    - [Méthodes d'alignement](#méthodes-dalignement)
    - [Ensemble de référence (Gold)](#ensemble-de-référence-gold)
- [Contenu des fichiers de métadonnées](#contenu-des-fichiers-de-métadonnées)
- [Outil de visualisation](#outil-de-visualisation)
- [Contributeurs et remerciements](#contributeurs-et-remerciements)
- [Références](#références)
  - [Fr-VD : Jeu de données et outil de visualisation](#fr-vd--jeu-de-données-et-outil-de-visualisation)
  - [Ressources bibliographiques supplémentaires](#ressources-bibliographiques-supplémentaires)

## Description du projet

[FrVD]: https://github.com/crim-ca/FrVD

**Fr-VD** est un jeu de données comprenant des références à des clips video (non-inclus),
des vidéodescriptions (VD), et des métadonnées (scène, personnage, actions),
en **français** pour l'apprentissage profond.

Il est accompagné d'un utilitaire de visualisation des clips vidéos, des vidéodescriptions et des actions.

### Contexte

Au cours des dernières années, le [CRIM][CRIM] a développé une expertise dans la production et la diffusion de
vidéodescriptions dans des œuvres audiovisuelles : nous avons vidéo-décris 142 films ou téléséries en français
en y ajoutant des métadonnées d'identification de scènes, de personnages et d'acteurs.

Dans le cadre d'un partenariat avec le [Fond d'accessibilité à la radiodiffusion][FAR] (FAR), nous avons voulu enrichir
ce corpus préalable en intégrant aussi l'identification des actions pour construire un jeu de données complet
en français utile à l'apprentissage profond pour la production de VD
automatique et/ou pour la détection d'éléments visuels (scène, personnage, action).

Durant le projet, nous avons ainsi cherché à détecter les actions :

- dans les clips vidéos en utilisant le modèle vidéo d'apprentissage automatique [SlowFast][SlowFast] ;
- et dans les VDs (via une annotation manuelle et une annotation automatique).

## Détails du jeu de données

En résumé le jeu de données contient :

- les `références` aux clips vidéos (film ou épisodes de télésérie) ;
- par `intervalle de temps` dans le video pour chaque item :
  - des annotations de `scènes` et de `personnages` (et `acteurs`) ;
  - la `vidédescription` rédigée manuellement en français (FR) ;
  - des `annotations linguistiques` classiques : segmentation de la VD en phrase, jetons, catégorie grammaticales,
    indice de position des jetons en utilisant la librairie [Stanza][Stanza] ;
  - les `annotations de la VD` : annotations manuelles ou automatiques (précisé selon le cas) des actions et des
    rôles (sujet, objet)` identifiées dans la VD en format IOB
    (voir le cadre d'annotation [plus bas](#Cadre-d'annotation)) ;
  - les `actions AVA de la VD`: les actions du jeu de données [AVA][AVA] détectées en se basant sur les
    annotations de la VD (plusieurs [méthodes d'alignement](#Méthodes-d'alignement) sont proposées).
  - les `actions AVA SlowFast` : les actions du jeu de données [AVA][AVA] détectées par le modèle video [SlowFast][SlowFast]
    (en anglais - 80 actions possibles) ;
  - les `actions Kinetics SlowFast` : les actions du jeu de données [Kinetics-600][Kinetics600] détectées par le modèle video [SlowFast][SlowFast]
    (en anglais - 600 actions possibles) ;

[CRIM]:https://www.crim.ca/fr/
[FAR]:https://www.baf-far.ca/fr
[Stanza]: https://stanfordnlp.github.io/stanza/

Les sections suivantes détaillent chacun des aspects présentés ci-haut.

### Annotations de la vidéodescription

Nous avons annoté les VD pour identifier les actions et les entités qui prennent part à l'action.
L'annotation a été faite manuellement sur 45% du corpus puis les 55% restant sont le résultat d'une annotation
automatique après utilisation d'un [modèle bi-LSTM][bi-LSTM] (développé au [CRIM][CRIM]) entraîné sur les données
annotées manuellement.

[bi-LSTM]:https://tac.nist.gov/publications/2017/participant.papers/TAC2017.CRIM.proceedings.pdf

#### Cadre d'annotation

Sont identifiés :

- `action` dénotée par un verbe, en distinguant :
  - `cas général` <br>
    (ex : _Des visiteurs **marchent** dans le parc_) ;
  - `verbe passif` <br>
    (ex : _Une serveuse **est projetée** sur une étagère de verres_) ;
  - `verbe support` (dans ce cas, quel est le verbe de substitution) <br>
    (ex : _faire la vaisselle_, _prendre une bouffée de cigarette_).
- Attributs ajoutés :
  - `type négation`
  - `type verbe de substitution` (pour les verbes supports) <br>
    (ex: _faire la vaisselle_ => _nettoyer_, _prendre une bouffée de cigarette_ => _fumer_).
- Entités impliquées dans l'action, en distinguant :
  - `sujet`
  - `objet direct`
  - `objet indirect essentiel`
  - en précisant pour chacun son type, c'est-à-dire :
    - `humain`,
    - `animal`,
    - `objet` ou
    - `concept`.

### Annotations d'actions depuis la vidéo

Implémentation à l'aide de [SlowFast][SlowFast].

[SlowFast]: https://github.com/facebookresearch/SlowFast

Code original de SlowFast, permettant d'utiliser des poids plus récents.
Des fonctionnalités supplémentaires (voir [SlowFast Pull Request #358][PR-SlowFast]) ont également été ajoutée de sorte
à pouvoir produire les logs de prédictions nécessaires pour extraire les actions générées par l'inférence sur vidéos.

[PR-SlowFast]: https://github.com/facebookresearch/SlowFast/pull/358

- SlowFast pré-entraîné sur [Kinetics-600][Kinetics600] ;

[Kinetics600]: https://deepmind.com/research/open-source/kinetics

- SlowFast pré-entraîné sur [AVA][AVA] avec améliorations.

[AVA]: https://research.google.com/ava/

Le modèle [detectron2][detectron2] est utilisé pour permettre la reconnaissance d'actions par individu en employant
les régions d'intérêt détectées plutôt que la prédiction d'actions globales sur l'intégralité de la trame vidéo.

[detectron2]: https://github.com/facebookresearch/detectron2

### Annotations d'actions depuis la VD

Nous avons fait correspondre les annotations d'actions de la VD avec le jeu d'action [AVA][AVA] (80 étiquettes).

Pour ce faire, nous avons :

- filtré et extrait les actions avec sujets humains (avec ou sans objet direct) depuis les annotations de la VD ;
- normalisé les sujets (les objets directs humains => "_quelqu'un_") ;
- obtenu au final ~9000 actions uniques en français <br>
  (ex: _sortir_, _apercevoir QQUN_, etc.)

#### Méthodes d'alignement

Ceci représente les stratégies employées pour parvenir à l'alignement des actions entre les domaines du texte
(provenant des [annotations de VD](#annotations-dactions-depuis-la-vd)) et du video
(provenant des [inférences sur vidéo](#annotations-dactions-depuis-la-vidéo)).

Pour aligner les ~9000 actions avec le jeu d'action [AVA][AVA], plusieurs méthodes ont été explorées.
Elles apparaissent chacune dans le jeu de données **Fr-VD** avec les attributs suivant nommés en fonction des méthodes
employées selon les cas :

- utilisation de **plongements lexicaux multilingues** :
  - [Muse][Muse]
  - [fastText][fasttext]
  - [SentenceBert][SB]

[Muse]:https://ai.facebook.com/tools/muse/
[fasttext]:https://fasttext.cc/docs/en/crawl-vectors.html
[SB]:https://www.sbert.net/

- utilisation de **ressources lexicales** (`Resslex`) par augmentation de données
  (récupération de synonymes en traduisant le jeu d'actions [AVA] de l'anglais en français),
  et en concaténant les ressources suivantes :
  - [Dictionnaire électronique de synonymes][Des]
  - [Wonef][Wonef]
  - [Wolf][Wolf]

[Des]:https://crisco2.unicaen.fr/des/
[Wonef]:https://aclanthology.org/W14-0105.pdf
[Wolf]:https://gforge.inria.fr/projects/wolf/

- utilisation d'un **alignement manuel** pour les 300 actions les plus fréquentes du corpus
  (représente plus de 50% des occurrences du corpus) :
  - `manuel` :
    l'action alignée manuellement
    (le caractère vide `-` est employé si l'action ne fait pas partie des 300 actions principales,
    et un caractère `_` est utilisé quand l'alignement est impossible)
  - `prox` :
    le degré de proximité sémantique par rapport au `gold`
    (`1` pour un alignement adéquat, `2` pour un alignement plus relâché, `0` si aucune action [AVA][AVA] ne correspond)

#### Ensemble de référence (Gold)

Nous avons également procédé à un alignement manuel complet sur 6 films et épisodes du
corpus de genres très différents afin de former un ensemble de référence `gold`:

- **films** :
  - _[Bon Cop, Bad Cop][Bon-Cop-Bad-Cop]_ (comédie-thriller, contexte policier/crime),
  - _[Le démantèlement][démantèlement]_ (drame, contexte en agriculture/ferme),
  - _[Ernest et Célestine][Ernest-Célestine]_ (dessin animé avec des animaux),

[Bon-Cop-Bad-Cop]: https://en.wikipedia.org/wiki/Bon_Cop,_Bad_Cop
[démantèlement]: https://fr.wikipedia.org/wiki/Le_Démantèlement
[Ernest-Célestine]: https://fr.wikipedia.org/wiki/Ernest_et_C%C3%A9lestine_

- **téléséries** :
  - _[La vie en vert][vie-en-vert]_, épisode 13 (documentaire écologie),
  - _[Le Rebut global][Rebut-global]_, épisode 10 (documentaire sur la construction),
  - _[Dans une galaxie près de chez vous][DUGPDCV]_, épisode 18 (science-fiction loufoque).

[vie-en-vert]: https://fr.wikipedia.org/wiki/La_Vie_en_vert
[Rebut-global]: https://fr.wikipedia.org/wiki/Le_Rebut_global
[DUGPDCV]: https://fr.wikipedia.org/wiki/Dans_une_galaxie_pr%C3%A8s_de_chez_vous

Dans ces cas, l'alignement est sous la catégorie `gold` avec le champ `prox` pour indiquer
le degré de proximité sémantique. Seuls les cas ci-haut possèdent cette catégorie dans le
jeu de données **Fr-VD**.

## Contenu des fichiers de métadonnées

![contents-shield](https://img.shields.io/badge/contenu-bientôt%20disponible-yellow)

**Note**: *Les vidéos ne sont pas inclus.*

## Outil de visualisation

L'outil disponible sur le répertoire [crim-ca/FrVD-visualization-tool][viewer-tool]
peut être employé afin d'afficher de manière synchronisée les métadonnées d'annotations des fichiers
de la base de données [FrVD][FrVD] avec les vidéos correspondants de films et téléséries.

[viewer-tool]: https://github.com/crim-ca/FrVD-visualization-tool

## Contributeurs et remerciements

Le jeu de données (**Fr-VD**) a été produit par les [contributeurs du CRIM](AUTHORS.md).

Le projet a bénéficié du soutien financier du [Fond d'accessibilité à la radiodiffusion][FAR] (FAR).

Les vidéodescriptions avaient été rédigées dans le cadre de projets passés bénéficiant
du soutien de l'_Office des personnes handicapées du Québec_ ([OPHQ][OPHQ]) et du
_Programme de soutien, à la valorisation et au transfert_ (PSVT).

[OPHQ]: https://www.ophq.gouv.qc.ca/

## Références

### Fr-VD : Jeu de données et outil de visualisation

Veuillez référencer à l'aide de la [citation suivante](README.md#Citation).

![report-shield](https://img.shields.io/badge/référence%20au%20rapport-bientôt%20disponible-yellow)

### Ressources bibliographiques supplémentaires

Voir [la page dédiée](REFERENCES.md) contenant les références complètes des ressources employées.
