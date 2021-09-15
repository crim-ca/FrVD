# FrVD: French Video Description dataset

## License

[![CC BY-NC-SA 4.0][cc-by-nc-sa-shield]][cc-by-nc-sa]
<br>
This work is licensed under a
[Creative Commons Attribution-NonCommercial-ShareAlike 4.0 International License][cc-by-nc-sa].

[![CC BY-NC-SA 4.0][cc-by-nc-sa-image]][cc-by-nc-sa]

[cc-by-nc-sa]: http://creativecommons.org/licenses/by-nc-sa/4.0/
[cc-by-nc-sa-image]: https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png
[cc-by-nc-sa-shield]: https://img.shields.io/badge/License-CC%20BY--NC--SA%204.0-lightgrey.svg


## Citation

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

## Description


> Le contenu suivant est également [disponible en français](README_FR.md).


Fr-VD is a dataset composed of video clip references (videos not included), of video-descriptions (VD), 
and metadata (scenes, characters, actions) in French for deep learning.

It comes with a utility tool for visualization of video clips, video-descriptions and actions.  

## Context

During recent years, [CRIM][CRIM] has developed an expertise in the production and diffusion of video-description for
audio-visual works. We have produced video-descriptions of 142 movies and TV shows in French by adding metadata for
the identification of scenes, characters and actors.

As part of a partnership with the _[Fond d'accessibilité à la radiodiffusion][FAR]_ (FAR), the previous corpus has been
enriched by integrating the identification of recognized visual actions in order to construct a complete dataset 
in French that could be employed for tasks such as deep learning for the automated production of VD or the detection
of visual elements (scene, characters, actions).

During this project, methods have been explored to detect actions: 
- in video clips using the visual model [SlowFast][SlowFast]; 
- in VD using both manual and automatic annotation strategies.

To summarize, the produced dataset contains: 
- `references` to video clips (either a film or TV show episode)
- `time intervals` within the video for each annotated item: 
  - annotations of `scenes` and `characters` (including `actors`);
  - manually transcribed `videodescription` in French (FR);
  - common `liguistic annotations`: segmentation of VD into sentences, tokens, grammatical categories, 
    and positional indices of tokens within sentences using [Stanza][Stanza] library;
  - textual `videodescription annotations`: using both manually and automatically annotated actions 
    (each variant is specified where applicable in metadata of the dataset);
  - textual `AVA VD annoatations`: actions from the [AVA][AVA] dataset detected directly 
    from transcribed video-descriptions (using multiple proposed [alignement strategies](#alignment-strategies)).
  - visual `AVA SlowFast annotations`: annotated actions using model [SlowFast][SlowFast] pretrained 
    with the [AVA][AVA] dataset (in English - 80 actions); 
  - visual `Kinetics SlowFast annotations`: annotated actions using model [SlowFast][SlowFast] pretrained 
    with the [Kinetics-600][Kinetics600] dataset (in English - 600 actions);

[CRIM]:https://www.crim.ca/fr/
[FAR]:https://www.baf-far.ca/fr   
[Stanza]: https://stanfordnlp.github.io/stanza/

## Annotations in Video-Descriptions

Textual annotations of the transcribed video-description has been produced to identify 
actions and entities related to the actions.
Annotations have been created manually for 45% of the entire VD corpus, and automatically for the remaining 55% using a
[bi-LSTM model][bi-LSTM] (developed by [CRIM][CRIM]) trained onto the manually annotated portion.

[bi-LSTM]:https://tac.nist.gov/publications/2017/participant.papers/TAC2017.CRIM.proceedings.pdf

### Annotations Format

Annotations are identified with the following definitions. Note that the French terminology is employed to preserve 
correspondance with real annotations produced in the dataset. 

- `action` denotes a verb, distinguished into categories: 
  - `cas général` <br>
    (e.g.: _Des visiteurs **marchent** dans le parc_, i.e.: _visitors **walk** in the parc_); 
  - `verbe passif` <br>
    (e.g.: _Une serveuse **est projetée** sur une étagère de verres_, 
    i.e.: _A waitress **is projected** onto a shelf of glasses_); 
  - `verbe support` (in this case, which is the substituted verb) <br>
    (e.g.: _faire la vaisselle_, _prendre une bouffée de cigarette_, i.e.: _do the dishes_, _take a puff of cigarette_).
- Added Attributes: 
  - `type négation`
  - `type verbe de substitution` (for verbs with a support attribute) <br> 
    (e.g.: _faire la vaisselle_ => _nettoyer_, _prendre une bouffée de cigarette_ => _fumer_, 
    i.e.: _do the dishes_ => _cleaning_, _take a puff of cigarette_ => _smoke_).
- Entities implied with action, distinguishing for each case components:
  - `sujet`
  - `objet direct`
  - `objet indirect essentiel`
  - and providing the specific type of each one, using one of:  
    - `humain`
    - `animal` 
    - `objet`
    - `concept`

## Actions Annotations from Video 

Actions detected from videos employ the [SlowFast][SlowFast] implementation.

[SlowFast]: https://github.com/facebookresearch/SlowFast

The original code of the authors linked above allows the use of the most recent pretrained weights. 
Some additional features (see [SlowFast Pull Request #358][PR-SlowFast]) have also been added in 
order to produce necessary prediction logs to retrieve inference actions from videos. 

[PR-SlowFast]: https://github.com/facebookresearch/SlowFast/pull/358

- Pretrained model weights for the [Kinetics-600][Kinetics600] dataset have been employed to produce the 
  corresponding annotations in the dataset;

[Kinetics600]: https://deepmind.com/research/open-source/kinetics

- Pretrained model weights for the [AVA][AVA] dataset have also been employed to generate corresponding 
  annotations in the dataset. These annotations also make use of the various improvements relevant to AVA.

[AVA]: https://research.google.com/ava/

The [detectron2][detectron2] model is employed (within SlowFast) to allow recognition of actions per individual
using detected bounding boxes rather than predicting actions globally over the full video frames.

[detectron2]: https://github.com/facebookresearch/detectron2

## Actions Annotations from VD

Annotations from VD has been aligned with available actions from the [AVA][AVA] dataset (80 labels).

To do so, following steps have been applied: 
- filtering and extraction of actions with human subjects (with or without `objet direct`) from VD annotations; 
- normalization of subjects (`object direct` are generalized into "_someone_", i.e.: "_quelqu'un_" in French)
- processed and obtained in total ~9000 unique actions in French <br>
  (e.g.: _sortir_, _apercevoir QQUN_, i.e.: _leave_, _to notice SOMEONE_)

### Alignment Strategies

In order to align the ~9000 actions with the [AVA][AVA] labels, many alignment methods have been explored.
Each of the explored strategies are available in the **Fr-VD** dataset. They are provided using the following 
attributes named after the corresponding methods applied for each case: 

- using **multilingual lexical embeddings**:
  - [Muse][Muse]
  - [fastText][fasttext]
  - [SentenceBert][SB]
  
[Muse]:https://ai.facebook.com/tools/muse/
[fasttext]:https://fasttext.cc/docs/en/crawl-vectors.html
[SB]:https://www.sbert.net/

- using **lexical resources** (`resslex` for _ressource lexicales_ in French) with data augmentation 
  (retrieval of synonyms following translation of English [AVA] actions in French), with concatenation 
  of following resources: 
  - [Dictionnaire électronique de synonymes][Des]
  - [Wonef][Wonef]
  - [Wolf][Wolf]
  
[Des]:https://crisco2.unicaen.fr/des/
[Wonef]:https://aclanthology.org/W14-0105.pdf
[Wolf]:https://gforge.inria.fr/projects/wolf/

- using **manual alignment** for the 300 most frequent actions within the corpus 
  (which represents more than 50% of all occurrences): 
  - `manual`: representing the manually aligned action when available 
    (empty value `-` is used if the action is not one of the 300 principal actions, 
    and `_` is used when alignment is impossible)
  - `prox`: represents the degree of semantic proximity of the action against the `gold` annotations
    (`1` meaning an adequate alignment was made, `2` for a loose alignment, 
    and `0` if no [AVA][AVA] action could be matched for alignment)

### Reference Gold Samples

Complete annotation and manual alignment has been accomplished over 6 movies and TV show episodes within the corpus. 
They are based on very different genres in order to form a diverse and robust 
set of `gold` reference annotation samples. Selected videos are: 

- **movies**:
  - _[Bon Cop, Bad Cop][Bon-Cop-Bad-Cop]_ (comedy-thriller, police/crime setting), 
  - _[Le démantèlement][démantèlement]_ (drama, agricultural/farm setting),
  - _[Ernest et Célestine][Ernest-Célestine]_ (animated comics with animals),
    
[Bon-Cop-Bad-Cop]:https://en.wikipedia.org/wiki/Bon_Cop,_Bad_Cop
[démantèlement]:https://fr.wikipedia.org/wiki/Le_Démantèlement
[Ernest-Célestine]:https://fr.wikipedia.org/wiki/Ernest_et_C%C3%A9lestine_
    
- **TV shows** : 
  - _[La vie en vert][vie-en-vert]_, episode 13 (ecological documentary), 
  - _[Le Rebut global][Rebut-global]_, episode 10 (construction documentary), 
  - _[Dans une galaxie près de chez vous][DUGPDCV]_, episode 18 (satirical humor, science-fiction).

[vie-en-vert]:https://fr.wikipedia.org/wiki/La_Vie_en_vert
[Rebut-global]:https://fr.wikipedia.org/wiki/Le_Rebut_global
[DUGPDCV]:https://fr.wikipedia.org/wiki/Dans_une_galaxie_pr%C3%A8s_de_chez_vous

Only above cases offer the alignment categorised as `gold` combined with `prox` indicator described
in the [Alignment Strategies](#Alignment-Strategies) in **Fr-VD** dataset. 

## File Contents

![contents-shield](https://img.shields.io/badge/contents-coming%20soon-yellow)


## Contributors and Acknowledgement

The offered dataset (**Fr-VD**) has been created by [CRIM contributors](AUTHORS.md).

The project received financial support from _[Fond d'accessibilité à la radiodiffusion][FAR]_ (FAR).

Video-descriptions have been originally transcribed in the context of past projects that received 
financial support from the _Office des personnes handicapées du Québec_ ([OPHQ][OPHQ]) and from 
the _Programme de soutien, à la valorisation et au transfert_ (PSVT).

[OPHQ]: https://www.ophq.gouv.qc.ca/

## References

### Fr-VD

Please refer to the above [citation](#citation) for referencing this dataset and work.

![report-shield](https://img.shields.io/badge/report%20reference-coming%20soon-yellow)



### Bibliographical references employed by this work 

See [references page](REFERENCES.md) that contains detailed and complete references to ressources.
