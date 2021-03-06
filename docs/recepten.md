# Receptenboek

## Ngram

De beschikbaarheid van de relatie `:next` maakt het zeer eenvoudig om naar specifieke woord-sequenties te zoeken. De volgende query vindt het trigram *in elk geval*:

```cypher
match p = (:word{lemma:'in'})-[:next]->
          (:word{lemma:'elk'})-[:next]->
          (:word{lemma:'geval'})
return p
```

En het is niet veel ingewikkelder om naar woorden in de context van een Ngram te zoeken. Welke woorden treden op na het trigram *in elk geval*:

```cypher
match (:word{lemma:'in'})-[:next]->
      (:word{lemma:'elk'})-[:next]->
      (:word{lemma:'geval'})-[:next]->(p)
return p
```

Het volgende voorbeeld zoekt naar adjectieven die in het patroon *vinden het ADJ dat* voorkomen:

```cypher
match (:word{lemma:'vinden'})-[:next]->
      (:word{lemma:'het'})-[:next]->
      (adj:word{pt:'adj'})-[:next]->
      (:word{lemma:'dat'})
return adj
```

De beschikbaarheid van de `:next`-relatie zorgt ervoor dat deze voorbeelden heel eenvoudig zijn. In vergelijking hiermee zijn voorbeelden met XPath in PaQu erg omslachtig.


## Multi-word Units

Multi-word units (mwu) worden steeds gerepresenteerd als een knoop direct boven de woorden waaruit de mwu bestaat. Als extraatje heeft deze niet-lexicale knoop toch een waarde voor de attributen `word` en `lemma`. Je kunt dus eenvoudig zoeken naar een specifieke mwu:

```cypher
match (p:node{word:'ten opzichte van'})
return p
```

Bovendien kun je dus queries formuleren zonder dat je een verschil moet maken tussen mwu en woorden, indien je de lemma of het woord van een gevonden knoop terug wilt geven:

```cypher
match (:node)-[:rel{rel:'svp'}]->(p:nw)
return p.word
```
Dit levert een tabel op waarin zowel woorden als mwu terecht komen.

Je kunt natuurlijk ook specifiek naar een deel van een mwu zoeken. De volgende query geeft mwu terug waarbij een van de onderdelen het woord *van* is:

```cypher
match (n:node{cat:'mwu'})-[:rel{rel:'mwp'}]->(:word{lemma:'van'})
return n
```

Het is iets ingewikkelder om te formuleren dat het woord *van* het laatste woord van de mwu moet zijn:

```cypher
match (n:node{cat:'mwu'})-[:rel{rel:'mwp'}]->(w:word{lemma:'van', end: w.end})
return n
```

Of waarbij *van* het derde woord moet zijn:

```cypher
match (n:node{cat:'mwu'})-[:rel{rel:'mwp'}]->(w:word{lemma:'van', begin: n.begin + 2})
return n
```

Een lastiger vraagstuk is een query waarbij je eisen wilt stellen aan
de attributen van een woord, ofwel aan de attributen van een mwu.
Bijvoorbeeld: geef me elke naam, waarbij een naam een mwu is waarvoor
geldt dat een van de delen van de mwu de `postag` *SPEC(deeleigen)*
heeft, ofwel het is geen mwu maar een woord waarvoor geldt dat het
attribuut `ntype` de waarde *eigen* heeft.

```cypher
match (w:word)<-[:rel*0..1{rel: 'mwp'}]-(n:nw)
where not exists ( (n)<-[r:rel{rel: 'mwp'}]-() )
  and ( w.postag = 'SPEC(deeleigen)' and n.cat = 'mwu'
     or w.id = n.id and w.ntype = 'eigen'
  )
return distinct n
```
Een bijkomend effect van het gebruik van `distinct` is dat de
resultaten gesorteerd worden. In dit geval krijg je eerst alle
multi-word units (type `:node`), en daarna de rest (type `:word`).

Dit kun je veranderen door niet op `n` te sorteren (d.m.v.
`distinct`), maar op attributen van `n`:

```cypher
match (w:word)<-[:rel*0..1{rel: 'mwp'}]-(n:nw)
where not exists ( (n)<-[r:rel{rel: 'mwp'}]-() )
  and ( w.postag = 'SPEC(deeleigen)' and n.cat = 'mwu'
     or w.id = n.id and w.ntype = 'eigen'
  )
return distinct n.sentid, n.id
```

## Multi-word Units en Universal Dependencies

In de Universal Dependency annotaties worden woorden die onderdeel zijn van een multi-word unit niet anders behandeld dan andere woorden. Als je dus een query formuleert waarbij het resultaat een woord is, krijg je ook woorden die deel uitmaken van een multi-word unit. Dit is vaak niet gewenst.

Bijvoorbeeld, de volgende query identificeert lijdend voorwerpen van het werkwoord eten:

```cypher
match (:word{lemma:'gebruiken'})-[:ud{rel:'obj'}]->(v)
return v.lemma
```

Als je de resultaten bekijkt op basis van de Alpino Treebank, krijg je lemma's zoals *machine*, *woord*, *stookolie*, *traangas*, maar bijvoorbeeld ook *soft*. Dat is eigenlijk een halve hit omdat het in de betreffende zin over *soft drugs* gaat. In UD is er in dit geval een *fixed* relatie van het eerste woord van de MWU met het tweede woord. Het zou aardig zijn als je in het overzicht van de lemma's dus *soft drug* in plaats van *soft* terug zou zien. Dat kan met het volgende recept, dat de UD relaties combineert met de relaties in de oorspronkelijke Alpino boom en vertrouwt op het feit dat MWU knopen ook een attribuut `lemma` (en `word`) hebben.

```cypher
match (:word{lemma:'gebruiken'})-[:ud{rel:'obj'}]->(v1:word)<-[:rel*0..1{rel:'mwp'}]-(v)
where not exists ( (v)<-[:rel{rel:'mwp'}]-()  )
return v.lemma
```

## Tellen van combinaties van waardes

Je kunt vrij gemakkelijk combinaties van de waardes van attributen tellen. Als voorbeeld kijken we hier naar werkwoorden met een vaste uitdrukking:

```cypher
match (v1:word{pt:'ww'})<-[:rel{rel:'hd'}]-()-[:rel{rel:'svp'}]->(v2:node)
return v1.lemma, v2.lemma
```

Deze paren van lemma's willen we vervolgens tellen en sorteren. Dat kan als volgt:

```cypher
match (v1:word{pt:'ww'})<-[:rel{rel:'hd'}]-()-[:rel{rel:'svp'}]->(v2:node)
return v1.lemma, v2.lemma, count(*) as aantal
order by aantal desc
```

Als een ander voorbeeld kun je dus gemakkelijk een frequentieoverzicht maken van de meest voorkomende paren van woordsoorten:

```cypher
match (w1)-[:next]->(w2)
return w1.postag, w2.postag, count(*) as aantal
order by aantal desc
```

Of een overzicht van zelfstandige naamwoorden en de relatieve voornaamwoorden waarmee ze gecombineerd worden:

```cypher
match (w1)<-[:rel{rel:'hd'}]-()-[:rel]->(:node{cat:'rel'})-[:rel{rel:'rhd'}]->(w2:word)
return w1.word, w2.word, count(*) as aantal
order by aantal desc
```


## Matches binnen matches

Je kunt queries soms opbouwen door een eerste selectie te maken en dan binnen die selectie een verdere selectie uit te voeren.

Stel dat we op zoek willen naar cross-serial verb clusters. Eén aanpak bestaat eruit om eerst de werkwoordclusters te identificeren, en vervolgens daarbinnen die gevallen te selecteren waarbij een werkwoord uit het cluster een lijdend voorwerp selecteert dat ook fungeert als het onderwerp van het VC-complement:

```cypher
match (x)<-[:rel]-(n:node{cat: 'inf'})<-[:rel{rel: 'vc'}]-(n1)-[:rel{rel: 'hd'}]->(w:word{pt: 'ww'})
where x.begin < w.begin
match (n)-[:rel{rel: 'su'}]->()<-[:rel{rel: 'obj1'}]-(n1)
return n
```

Het gebruik van de techniek om een patroon onder te verdelen in meerdere matches maakt de queries soms makkelijker te begrijpen.




## Tellen van reeksen

Stap 1: zoek iets

```cypher
match (n:node{_deste: true})
return distinct n.sentid as sid, n.id as id
order by sid, id
```

Stap 2: zoek de pt van woorden onder iets

```cypher hl_lines="2 4 6"
match (n:node{_deste: true})
match (n)-[:rel*]->(w:word)
return distinct n.sentid as sid, n.id as id
    , w.end as positie, w.pt as pt
order by sid, id
    , positie
```

Stap 3: voeg de pt van woorden per iets samen, de oude `return` wordt `with`

```cypher hl_lines="3 6"
match (n:node{_deste: true})
match (n)-[:rel*]->(w:word)
with
    distinct n.sentid as sid, n.id as id, w.end as positie, w.pt as pt
order by sid, id, positie
return sid, id, collect(pt) as pt_list
```

Stap 3a: variant om te kijken of de woorden wel direct achter elkaar staan

```cypher hl_lines="4"
match (n:node{_deste: true})
match (n)-[:rel*]->(w:word)
with distinct n.sentid as sid, n.id as id, w.end as positie, 
    w.end + ':' + w.pt as pt
order by sid, id, positie
return sid, id, collect(pt) as pt_list
```

Stap 4: tel de frequenties van pt van woorden onder iets

```cypher  hl_lines="5 7"
match (n:node{_deste: true})
match (n)-[:rel*]->(w:word)
with distinct n.sentid as sid, n.id as id, w.end as positie, w.pt as pt
order by sid, id, positie
with
    sid, id, collect(pt) as pt_list
return count(*) as aantal, pt_list
```

## Meta-data

### Overzicht van de meta-data

Als er voor een corpus meta-data beschikbaar is, dan zijn er knopen met als type :meta. De aard van de meta-data is verschillend per corpus, en daarom ook de attributen die voor :meta beschikbaar zijn. Maar er is altijd het attribuut `sentid` en de knopen van de syntactische analyse hebben ook allemaal dat `sentid` attribuut met dezelfde waarde. Je kunt dus met een where-clause de bijbehorende meta-data terugvinden.

De volgende query toont voor een corpus welk type meta-data beschikbaar is:

```cypher
match (m:meta)
return m.name, count(*)
```

De counts geven in dit geval aan voor hoeveel zinnen de meta-data van dat type beschikbaar is. En om te zien welke waardes in de meta-data worden gebruikt, kun je zoiets doen. Hier is 'country' dan een van de types meta-data die je in de vorige query hebt opgeleverd.

```cypher
match (m:meta{name:'country'})
return m.value, count(*)
```

Je kunt dit ook doen voor meerdere types meta-data tegelijk. Bijvoorbeeld, hoe is de man/vrouw verhouding voor het Nederlandse en Vlaamse deel van het CGN. Hier gebruiken we het attribuut `sentid` om te zien welke meta-data bij elkaar hoort.

```cypher
match (m:meta{name:'country'}),
      (m2:meta{name:'sex', sentid:m.sentid})
return m.value, m2.value, count(*)
```


### Gebruik van meta-data in queries

Als voorbeeld nemen we de volgende query die zoekt naar gevallen waarbij een NP de eerste woordgroep van een zin is, en als onderwerp fungeert:

```cypher
match (n:nw{_vorfeld: true, _np: true})<-[r:rel{rel:'su'}]-()
return n
```

Als we van de matchende zin ook de meta-data willen opleveren kunnen we gebruik maken van het feit dat alle knopen ook het attribuut `sentid` bevatten. Daarmee halen we dan de relevante meta-data op:

```cypher
match (n:nw{_vorfeld: true, _np: true})<-[r:rel{rel:'su'}]-(),
      (m:meta{name:'country', sentid: n.sentid})
return n, m.value
```

Je kunt dus ook eisen aan de relevante meta-data stellen. Of delen van de meta-data teruggeven. Of tellen. Dit voorbeeld werkt voor het CGN corpus:

```cypher
match (n:nw{_vorfeld: true, _np: true})<-[r:rel{rel:'su'}]-(),
      (m:meta{name:'country', sentid: n.sentid})
return m.value, count(*)
```

En ook meta-data combineren:

```cypher
match (n:nw{_vorfeld: true, _np: true})<-[r:rel]-(),
      (m:meta{name:'country', sentid: n.sentid}),
      (m2:meta{name:'sex', sentid: n.sentid})
where r.rel in ['su','sup'] and r.id is null
return m.value, m2.value, count(*)
```



## Aardige voorbeelden die nergens passen

### Wie is de baas?

De volgende query verzameld onderwerpen van het werkwoord *beslissen* waarbij dat onderwerp ofwel een multi-word unit is, ofwel een zelfstandig naamwoord.

```cypher
match (:word{lemma:'beslissen'})-[:pair{rel:'su'}]->(v)
where v.cat='mwu' or v.pt='n'
return v
```

Als je in dergelijke queries meerdere mogelijke lemma's wilt noemen kan dat met een `WHERE` clause:

```cypher
match (h:word)-[:pair{rel:'su'}]->(v)
where h.lemma in ['beslissen','besluiten']
  and (v.cat='mwu' or v.pt='n')
return v
```

### Man/vrouw verdeling

De volgende twee queries kun je gebruiken om eventueel verschillen te vinden tussen zelfstandige naamworden die met possesief *haar* of *zijn* worden gebruikt.

```cypher
match (w1:word)-[:ud{rel:'nmod:poss'}]->(w2:word{lemma:'haar'})
return w1
```


```cypher
match (w1:word)-[:ud{rel:'nmod:poss'}]->(w2:word{lemma:'zijn'})
return w1
```

Voor *haar* levert dat als top-10 op: *man, vriend, kind, moeder, dochter, zoon, vader, leven, rol, echt_genoot*. Voor *zijn* krijgen we de volgende lijst (DutchWebCorpus): *vrouw, ploeg, naam, auto, vader, collega, vriendin, leven, zoon, club*.

Merk op dat je bij deze queries wellicht de mededeling "Afgebroken" krijgt van AlpinoGraph, als je de tab "woorden" of "lemma's" gebruikt. Als je de telling voor het hele corpus wilt uitvoeren kun je dat doen door de telling expliciet in de query zelf uit te voeren:

```cypher
match (w1:word)-[:ud{rel:'nmod:poss'}]->(w2:word{lemma:'haar'})
return w1.lemma, count(*) as aantal
order by aantal desc
```

### Paren van coördinatoren

In het volgende voorbeeld zoeken we naar een conjunctie met twee
coördinatoren (*noch .. noch..*, *zowel .. als ..*). De eerste match
zorgt voor een coördinatie die twee coördinatoren bevat. De tweede
match eist dat de eerste coördinator vooraf gaat aan de tweede - op
die manier krijg je elke conjunctie maar één keer, en de betreffende
lemma's in de verwachte volgorde:

```cypher
match (v1:word) <-[:rel{rel:'crd'}]-(:node{cat: 'conj'})-[:rel{rel: 'crd'}]->(v2:word)
where v1.end < v2.end
return v1.lemma, v2.lemma, count(*) as aantal
order by aantal desc
```

### Verwante lemma's

Je zoekt vaker naar een bepaald lemma dan naar een bepaald woord,
omdat je constructies met alle woordvarianten van dat lemma wilt
vinden. Maar het kan zijn dat er meerdere varianten van aan elkaar
verwante lemma's zijn, en die kun je vaak vinden via overeenkomende
woorden.

Onderstaand voorbeeld geeft een aardig resultaat met het corpus
*BasiLex 1.0*

```cypher
match (w1:word{lemma: 'fiets'})
with distinct w1.word as word
match (w:word{word: word})
return distinct w.lemma, w.word, w.root, w.pt
order by lemma, root, pt, word
```

### Woorden die in verschillende soorten constructies gebruikt worden

Je kunt ook kijken of een bepaald woord of lemma in verschillende constructies
voorkomt. In het volgende voorbeeld willen we werkwoorden terugvinden die vaker
dan eenmalig voorkomen met zowel een whsub als vc ("hij vroeg waarom hij te laat was")
als een whrel als obj1 ("hij bedroog wie hij maar wilde"). Dat gaat makkelijk door
gebruik te maken van twee match clauses:

```cypher
match (w1:word{pt:'ww'})<-[:rel{rel:'hd'}]-()
         -[:rel{rel:'vc'}]->(:node{cat:'whsub'})
with w1.lemma as lemma1, count(*) as n
where n > 1
match (w:word{pt:'ww'})<-[:rel{rel:'hd'}]-()
         -[:rel{rel:'obj1'}]->(:node{cat:'whrel'})
where w.lemma in lemma1
return w
```

## Sonar

!!! warning "Let op"
    Dit is alleen voor het corpus Lassy Klein

Het corpus Lassy Klein bevat *named entities* die afkomstig zijn uit
SONAR, een onderdeel van Lassy. Zie [hier](../corpora/#lassy-klein).

Zo vind je het eerste woord van alle named entities:

```cypher
match (w:word)
where w.begin = w.sonar_ne_begin
return w
```

Sommige named entities lopen door naar een volgende zin, bijvoorbeeld
een boektitel die uit meerdere zinnen bestaat. Deze gevallen
vind je zo:

```cypher
match (w:word)
where w.begin = w.sonar_ne_begin
match (w2:word{sentid: w.sentid, last: true})
where w2.end < w.sonar_ne_end
return w
```

En zo vind je de stukken die horen bij een named entity die in een
vorige zin is begonnen:

```cypher
match (w:word{begin: 0})
where w.sonar_ne_begin < 0
return w
```

Zo vind je alle named entities, compleet en incompleet, met
bijbehorende woorden:

```cypher
match p = (w:word)-[:next*0..]->(w2:word{sentid: w.sentid})
where ( w.begin = w.sonar_ne_begin or ( w.begin = 0 and w.sonar_ne_begin < 0 ) )
  and ( w2.end = w.sonar_ne_end or ( w2.last = true and w.sonar_ne_end > w2.end ) )
return p
```

En zo vind je alleen de complete named entities:

```cypher
match p = (w:word)-[:next*0..]->(w2:word{sentid: w.sentid})
where w.begin = w.sonar_ne_begin
  and w2.end = w.sonar_ne_end     -- NIET w2.sonar_ne_end
return p
```

Zo vind je named entities die maar uit één woord bestaan:

```cypher
match (w:word)
where w.begin = w.sonar_ne_begin
  and w.end = w.sonar_ne_end
return w
```

Dit geeft hetzelfde resultaat:

```cypher
match (w:word)
where w.sonar_ne is not null
return w
```

Bij veel, maar lang niet alle, named entities van meerdere woorden hebben
de woorden samen en met geen enkel ander woord één gezamelijke parent. Zo'n
parent vind je zo:

```cypher
match (n:node)
where n.sonar_ne is not null
return n
```

En uiteraard kun je zoeken op een specifieke soort named entity:

```cypher
match p = (w:word{sonar_ne_class: 'loc'})-[:next*0..]->(w2:word)
where w.begin = w.sonar_ne_begin
  and w2.end = w.sonar_ne_end     -- NIET w2.sonar_ne_end
return p
```

Of:

```cypher
match (n:nw{sonar_ne: 'pro'})
return n
```

De laatste vindt niet alles, omdat het attribuut `sonar_ne` niet
altijd aanwezig is (zie boven).

Een overzicht van alle soorten:

```cypher
match (w:word)
where w.begin = w.sonar_ne_begin
return w.sonar_ne_class, count(*)
```

Een overzicht van de lengtes:

```cypher
match (w:word)
where w.begin = w.sonar_ne_begin
with w.sonar_ne_end - w.sonar_ne_begin as lengte
return lengte, count(*)
```

## Compound parts

Zoek een woord met *groen* als compound part:

```cypher
match (w:word)
where 'groen' in w._cp
return w
```

Zoek een woord dat begint met *groen* of *bruin*:

```cypher
match (w:word)
where head(w._cp) in ['groen','bruin']
return w
```

Zoek een woord dat eindigt met *groen* of *bruin*:

```cypher
match (w:word)
where last(w._cp) in ['groen','bruin']
return w
```

Zoek een woord waarvan het tweede deel *groen* of *bruin* is (telling
start bij 0):

```cypher
match (w:word)
where w._cp[1] in ['groen','bruin']
return w
```

Zoek een woord waravan het voorlaatste deel *groen* of *bruin* is (laatste is lengte - 1):

```cypher
match (w:word)
where w._cp[length(w._cp)-2] in ['groen','bruin']
return w
```

Het volgende voorbeeld maakt gebruik van deze macro:

```python
kleur = """
    'rood','groen','blauw','geel','oranje','bruin','roze','wit','zwart','grijs','paars','lila'
"""
```

Zoek woorden met een kleur:

```cypher
match (w:word)
where any(x in w._cp where x in [%kleur%])
return w.word, count(*)
order by count desc, word
```
