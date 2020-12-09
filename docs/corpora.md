# Corpora

Een overzicht van corpora die beschikbaar zijn in de officiële versie
van AlpinoGraph.

!!! warning "Let op"
    De afleiding van Universal Dependencies is ook bij
    handmatig verwerkte corpora automatisch gedaan.

## Alpino Treebank

!!! info
    Aantal zinnen:  7.136 <br>
    Verwerking: handmatig

Dit bevat de handmatig geannoteerde zinnen die gedistribueerd worden
als onderdeel van het Alpino systeem. De zinnen bestaan uit het
dagbladdeel (cdbl) van het Eindhoven corpus. De eerste versie van de
Alpino Treebank is verschenen op CDROM en werd in november 2002
feestelijk overhandigd aan de eerste computationeel-taalkundige van
Nederland: Hugo Brandt Corstius.

De volgende attributen op `(:word)` zijn wel automatisch gegenereerd:

> aform
> case
> comparative
> def
> frame
> gen
> iets
> infl
> lcat
> neclass
> num
> per
> personalized
> pron
> refl
> rnum
> sc
> sense
> special
> stype
> tense
> vform
> wh
> wk

Meer informatie op de [Alpino Treebank website](https://www.let.rug.nl/~vannoord/trees/).

Laatste versie beschikbaar op [GitHub](https://github.com/rug-compling/Alpino).

## BasiLex 1.0

!!! info
    Aantal zinnen: 1.635.680 <br>
    Verwerking: automatisch <br>
    Metadata: `grade`, `level`, `level_determination`, `maintype`, `prod_date`, `type`

Het BasiLex-corpus is een geannoteerde verzameling van teksten geschreven voor kinderen in de basisschoolleeftijd. Het corpus bevat 13,5 miljoen tokens, waarvan 11,5 miljoen woorden. De tokens komen voor ongeveer 40% uit educatieve materialen, 40% uit kinderliteratuur en 20% uit media.

Laatste versie beschikbaar bij het [Instituut voor de Nederlandse taal](https://taalmaterialen.ivdnt.org/download/tstc-basilex-corpus/).

## BasiScript 1.0: Opstellen

!!! info
    Aantal zinnen: 782.179 <br>
    Verwerking: automatisch <br>
    Metadata: `date`, `gender`, `grade`, `location`, `name`, `type`

BasiScript is een corpus met 9 miljoen woorden geschreven tekst geproduceerd door leerlingen van de Nederlandse basisschool. In AlpinoGraph is het “opstellen”-deel opgenomen.

Het corpus bevat longitudinale data verzameld over drie achtereenvolgende jaren (najaar 2012 – voorjaar 2015). Het BasiScript-corpus is ontworpen om zowel de educatieve diversiteit (type school) als de geografische regio’s van Nederland te kunnen vergelijken.

De data bevat voornamelijk handgeschreven teksten en een klein aantal teksten geproduceerd met een tekstverwerker (met automatische spelling- en grammaticacontrole uitgeschakeld). De data is geanonimiseerd.

Laatste versie beschikbaar bij het [Instituut voor de Nederlandse taal](https://taalmaterialen.ivdnt.org/download/tstc-basiscript-corpus/).

## Childes Dutch

!!! info
    Aantal zinnen: 545.476 <br>
    Verwerking: automatisch <br>
    Metadata: `age`, `code`, `months`, `paqu.path1`, `paqu.path2`, `paqu.path3`, `role`, `sex`

Childes is een corpus van gesproken taal van jonge kinderen en hun
gesprekspartners. De versie die in AlpinoGraph is opgenomen is op 18
november 2015 gedownload.
Zie [CHILDES: Child Language Data Exchange System](https://childes.talkbank.org/).

Het corpus bevat de volgende onderdelen:

 * CLPF
     * Catootje, David, Elke, Enzo, Eva, Jarmo, Leon, Leonie, Noortje, Robin, Tirza, Tom
 * DeHouwer
     * Dieter, Katrien, Kim, Michiel
 * Gillis
 * Groningen
     * Abel, Daan, Iris, Josse, Matthijs, Peter, Tomas
 * Schaerlaekens
     * Arnold, Diederik, Gijs, Joost, Katelijne, Maria
 * VanKampen
 * Wijnen
 * Zink
     * David, Judith, Laurien, Meinder



## CLEF

!!! info
    Aantal zinnen: 4.266.515 <br>
    Verwerking: automatisch

Dit corpus bevat alle zinnen van het Algemeen Dagblad en de NRC van
1994 en 1995. De zinnen zijn automatisch geannoteerd met de Alpino
parser. Deze data is destijds gebruikt voor de CLEF shared tasks op
het gebied van Question Answering.

## Corpus Gesproken Nederlands

!!! info
    Aantal zinnen: 129.921 <br>
    Verwerking: handmatig <br>
    Metadata: `birthyear`, `country`, `sex`, `source`, `speaker_id`, `talk_id`

Dit bevat de handmatig geannoteerde zinnen van het CGN (ongeveer 1 miljoen woorden), Versie 2.

Meer informatie op de [website van het Corpus Gesproken Nederlands](http://lands.let.ru.nl/cgn/).

Laatste versie beschikbaar bij het [Instituut voor de Nederlandse taal](https://taalmaterialen.ivdnt.org/download/tstc-corpus-gesproken-nederlands/).

## Dutch Web Corpus

!!! info
    Aantal zinnen: 1.498.479 <br>
    Verwerking: automatisch

This automatically annotated treebank contains the first 1.5 million
sentences of a crawled newspaper corpus. The corpus has been collected
by Wietse de Vries, as additional data for training his Bertje
language model.

Wietse de Vries, Andreas van Cranenburgh, Arianna Bisazza, Tommaso
Caselli, Gertjan van Noord, Malvina Nissim, *BERTje: A Dutch BERT
Model*. [Arxiv 1912.09582](https://arxiv.org/abs/1912.09582).

## Eindhoven

!!! info
    Aantal zinnen: 40.524 <br>
    Verwerking: automatisch

Het Eindhoven-corpus is al begin jaren zeventig verzameld. Jarenlang
was de copyright status van dit corpus onduidelijk, maar inmiddels is
een versie van het corpus via het
[Instituut voor de Nederlandse taal](https://taalmaterialen.ivdnt.org/download/tstc-eindhoven-corpus/)
te
downloaden. De versie in AlpinoGraph gaat terug op een versie waarvan de
preciese geschiedenis in nevelen is gehuld.

## Lassy Groot: Kranten

!!! info
    Aantal zinnen: 14.974.458 <br>
    Verwerking: automatisch

Dit is het deel `WR-P-P-G` van het corpus Lassy Groot. Dit betreft materiaal afkomstig uit dagbladen.

Meer informatie op de [Lassy website](https://www.let.rug.nl/vannoord/Lassy/).

Laatste versie beschikbaar bij het [Instituut voor de Nederlandse taal](https://taalmaterialen.ivdnt.org/download/tstc-lassy-groot-corpus/)

## Lassy Klein

!!! info
    Aantal zinnen: 65.200 <br>
    Verwerking: handmatig <br>
    Metadata: `source`, `type`, `description` <br>
    Extra attributen: `dscmanual`, `dscsense`, `sonar_ne`, `sonar_ne_begin`, `sonar_ne_class`, `sonar_ne_end`

Lassy Klein is een handmatig geannoteerd corpus van ongeveer 1 miljoen
woorden. De huidige versie betreft een ontwikkelingsversie uit 2020.

De volgende attributen op `(:word)` zijn wel automatisch gegenereerd:

> aform
> case
> comparative
> def
> frame
> gen
> iets
> infl
> lcat
> neclass
> num
> per
> personalized
> pron
> refl
> rnum
> sc
> sense
> special
> stype
> tense
> vform
> wh
> wk

Lassy Klein bevat delen uit een voorlopige versie van het corpus SONAR500 (de
codering in bestandsnamen
[wijkt af](https://www.let.rug.nl/vannoord/Lassy/Lassy-Klein-Groot.txt) van
die in de definitieve versie van SONAR500), Dutch Parallel Corpus, en Wikipedia.

De attributen `dscmanual` en `dscsense` bevatten sense-informatie uit
het Dutch semantic corpus.
Zie [DutchSemCor Project Homepage](http://wordpress.let.vupr.nl/dutchsemcor/)

De attributen  `sonar_ne`, `sonar_ne_begin`, `sonar_ne_class` en
`sonar_ne_end` bevatten informatie over *named entities* uit SONAR500.
Zie [Sonar](../recepten/#sonar) in het receptenboek.

Meer informatie op de [Lassy website](https://www.let.rug.nl/vannoord/Lassy/).

Laatste versie beschikbaar bij het [Instituut voor de Nederlandse taal](https://taalmaterialen.ivdnt.org/download/tstc-lassy-klein-corpus/).

## NL-wiki 2017

!!! info
    Aantal zinnen: 16.073.845 <br>
    Verwerking: automatisch

Dit corpus bevat alle zinnen van de dump van de Nederlandse Wikipedia van 1 Augustus 2017.

## Wablieft

!!! info
    Aantal zinnen: 256.729 <br>
    Verwerking: automatisch <br>
    Metadata: `datum`, `issue`, `rubriek`

it betreft het Wablieft corpus versie 1.2. Het Wablieft-corpus bevat
het digitaal archief van de Wablieft-krant (periode 2011-2017), zoals
ook beschikbaar op de
[wablieft website](http://www.wablieft.be/krant/archief).
Het bevat 2 miljoen woorden krantenmateriaal in eenvoudig te lezen
Nederlands.

Meer informatie:
[Wablieft: An Easy-to-Read Newspaper Corpus for Dutch](https://lirias.kuleuven.be/retrieve/548433) (PDF).

Laatste versie beschikbaar bij het [Instituut voor de Nederlandse taal](https://taalmaterialen.ivdnt.org/download/tstc-wablieft-corpus-1-2/).
