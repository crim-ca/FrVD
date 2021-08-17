# Fr-VD

**Fr-VD** est un jeu de données comprenant des références à des clips video (non-inclus), 
des vidéodescriptions (VD), et des métadonnées (scène, personnage, actions),
en **français** pour l'apprentissage profond.

Il est accompagné d'un utilitaire de visualisation des clips vidéos, des vidéodescriptions et des actions.

## Contexte

Au cours des dernières années, le [CRIM][CRIM] a développé une expertise dans la production et la diffusion de 
vidéodescriptions dans des œuvres audiovisuelles : nous avons vidéo-décris 142 films ou téléséries en français 
en y ajoutant des métadonnées d'identification de scènes et de personnages.


Dans le cadre d'un partenariat avec le [Fond d'accessibilité à la radiodiffusion][FAR], nous avons voulu enrichir
ce corpus préalable en intégrant aussi l'identification des actions pour construire un jeu de données complet 
en français utile à l'apprentissage profond pour la production de VD 
automatique et/ou pour la détection d'éléments visuels (scène, personnage, action). 

Durant le projet, nous avons ainsi cherché à détecter les actions :
- dans les clips vidéos en utilisant le modèle vidéo d'apprentissage automatique [SlowFast][SlowFast] ;
- et dans les VDs (via une annotation manuelle et une annotation automatique).

En résumé le jeu de données contient : 
- les `références` aux clips vidéos (film ou télésérie) ;
- par `intervalle de temps` : 
    - des annotations de `scènes` et de `personnages` (et _acteurs_) ;
    - la `vidédescription` rédigée manuellement en FR ;
    - des `annotations linguistiques` classiques : segmentation de la VD en phrase, jetons, catégorie grammaticales, 
      indice de position des jetons en utilisant la librairie [Stanza][Stanza] ;
    - les `annotations de la VD` : annotations manuelles ou automatiques (précisé selon le cas) des actions et des 
      rôles (sujet, objet)` identifiées dans la VD en format IOB 
      (voir le cadre d'annotation [plus bas](#Cadre-d'annotation)) ;
    - les `actions AVA SlowFast` : les actions du jeu de données AVA détectées par le modèle video SlowFast 
      (en anglais - 80 actions possibles) ;
    - les `actions AVA de la VD`: les actions du jeu de données AVA détectées en se basant sur les 
      annotations de la VD (plusieurs méthodes d'alignement sont proposées - voir plus bas).

[CRIM]:https://www.crim.ca/fr/
[FAR]:https://www.baf-far.ca/fr   
[Stanza]: https://stanfordnlp.github.io/stanza/
    
## Annotations de la vidéodescription

Nous avons annoté les VD pour identifier les actions et les entités qui prennent part à l'action.
L'annotation a été faite manuellement sur 45 % du corpus puis les 55 % restant sont le résultat d'une annotation 
automatique après utilisation d'un [modèle bi-LSTM][bi-LSTM] (développé au CRIM) entraîné sur les données 
annotées manuellement.

[bi-LSTM]:https://tac.nist.gov/publications/2017/participant.papers/TAC2017.CRIM.proceedings.pdf

### Cadre d'annotation

Sont identifiés : 
 - `Action` dénotée par un verbe, en distinguant : 
    - `cas général` (ex : _Des visiteurs **marchent** dans le parc_) ; 
    - `verbe passif` (ex : _Une serveuse **est projetée** sur une étagère de verres_) ; 
    - `verbe support` (dans ce cas, quel est le verbe de substitution)
      (ex : _faire la vaisselle_, _prendre une bouffée de cigarette_).
- Attributs ajoutés : 
    - `type négation`
    - `type verbe de substitution` (pour les verbes supports) 
      (_faire la vaisselle_ = _nettoyer_, _prendre une bouffée de cigarette_ = _fumer_).
 - Entités impliquées dans l'action, en distinguant :
    - `sujet` 
    - `objet direct` 
    - `objet indirect essentiel`
    - en précisant pour chacun son type, c'est-à-dire :
        - `humain`,
        - `animal`, 
        - `objet` ou
        - `concept`.
      

## Annotations d'actions depuis la vidéo

Implémentation à l'aide de [SlowFast][SlowFast].

[SlowFast]: https://github.com/facebookresearch/SlowFast  

Code original de SlowFast, permettant d'utiliser des poids plus récents.  
 - SlowFast pré-entraîné sur [Kinetics600][Kinetics600] ;

[Kinetics600]: https://deepmind.com/research/open-source/kinetics

- SlowFast pré-entraîné sur [AVA][AVA] avec améliorations.

[AVA]: https://research.google.com/ava/

Utilisation de [detectron2][detectron2] pour la reconnaissance d'actions par individu plutôt que globales.

[detectron2]: https://github.com/facebookresearch/detectron2

## Annotations d'actions depuis la VD

Nous avons fait correspondre les annotations d'actions de la VD avec le jeu d'action [AVA][AVA] (80 étiquettes).

Pour ce faire, mous avons :
- filtré et extrait les actions avec sujets humains (avec ou sans objet direct) depuis les annotations de la VD,
- normalisé (les objets directs humains = "_quelqu'un_"),
- obtenu au final environ ~9 000 actions uniques en français (Ex : _sortir, apercevoir QQUN_, etc.)

### Méthodes d'alignement

Pour aligner les ~9000 actions avec le jeu d'action AVA, plusieurs méthodes ont été explorées. 
Elles apparaissent chacune dans le jeu de données Fr-VD :
- utilisation de **plongements lexicaux multilingues** : 
  - [Muse][Muse]
  - [fastText][fasttext]
  - [SentenceBert][SB]
  
[Muse]:https://ai.facebook.com/tools/muse/
[fasttext]:https://fasttext.cc/docs/en/crawl-vectors.html
[SB]:https://www.sbert.net/

- utilisation de **ressources lexicales** (`Resslex`) par augmentation de données 
  (récupération de synonymes en traduisant le jeu d'actions [AVA]), en concaténant les ressources suivantes : 
  - [Dictionnaire électronique de synonymes][Des]
  - [Wonef][Wonef]
  - [Wolf][Wolf]
  
[Des]:https://crisco2.unicaen.fr/des/
[Wonef]:https://aclanthology.org/W14-0105.pdf
[Wolf]:https://gforge.inria.fr/projects/wolf/

- utilisation d'un **alignement fait manuellement** pour les 300 actions les plus fréquentes du corpus 
  (représente plus de 50% des occurrences du corpus) : 
  - `manuel` : 
    l'action alignée manuellement (est vide (`-`) si ne fait pas partie des 300 actions, 
    est nul (`_`) quand l'alignement est impossible)
  - `prox` : 
    le degré de proximité sémantique par rapport au `gold`
    (`1` pour un alignement adéquat, `2` pour un alignement plus relâché, `0` si aucune action AVA ne correspond) 
 
    
### Gold de 6 films

Nous avons également procédé à un alignement manuel complet sur quelques films du corpus de genres très différents : 
- **films** : 
  - _[Bon Cop, Bad Cop][Bon Cop, Bad Cop]_ (policier), 
  - _[Le démantèlement][Le démantèlement]_ (agriculture),
  - _[Ernest et Célestine][Ernest et Célestine]_ (dessin animé avec des animaux),
    
[Bon Cop, Bad Cop]:https://en.wikipedia.org/wiki/Bon_Cop,_Bad_Cop
[Le démantèlement]:https://fr.wikipedia.org/wiki/Le_Démantèlement
[Ernest et Célestine]:https://fr.wikipedia.org/wiki/Ernest_et_C%C3%A9lestine_
    
- **téléséries** : 
  - _[La vie en vert][La vie en vert]_, épisode 13 (écologie), 
  - _[Le Rebut global][Le Rebut global]_, épisode 10 (construction), 
  - _[Dans une galaxie près de chez vous][Dans une galaxie près de chez vous]_, épisode 18 (science-fiction loufoque).

[La vie en vert]:https://fr.wikipedia.org/wiki/La_Vie_en_vert
[Le Rebut global]:https://fr.wikipedia.org/wiki/Le_Rebut_global
[Dans une galaxie près de chez vous]:https://fr.wikipedia.org/wiki/Dans_une_galaxie_pr%C3%A8s_de_chez_vous

Dans ces cas l'alignement est sous la catégorie `gold` avec le champ `prox` pour indiquer 
le degré de proximité sémantique.

# Contenu des fichiers

![contents-shield](https://img.shields.io/badge/contenu-bientôt%20disponible-yellow)


# Contributeurs et remerciements

Ces jeux de données ont été produits par les [contributeurs du CRIM](AUTHORS.md). 

Le projet a bénéficié du soutien financier du [Fond d'accessibilité à la radiodiffusion][FAR].

Les vidéodescriptions avaient été rédigées dans le cadre de projets passés bénéficiant 
du soutien de l'Office des personnes handicapées du Québec ([OPHQ][OPHQ]) et du 
Programme de soutien, à la valorisation et au transfert (PSVT).

[OPHQ]: https://www.ophq.gouv.qc.ca/

# Références

### FR-VD

Veuillez référencer à l'aide de la [citation suivante](README.md#Citation).


### Ressources utilisées pour la constitution de Fr-VD

Voir [la page dédiée](REFERENCES.md) contenant les références complètes des ressources employées.