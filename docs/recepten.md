# Receptenboek

## Ngram

De beschikbaarheid van de relatie `:next` maakt het zeer eenvoudig om naar specifieke woord-sequenties te zoeken. De volgende query vindt het trigram *in elk geval*:

```text
match p = (:word{lemma:'in'})-[:next]->
          (:word{lemma:'elk'})-[:next]->
          (:word{lemma:'geval'})
return p
```

En het is niet veel ingewikkelder om naar woorden in de context van een Ngram te zoeken. Welke woorden treden op na het trigram *in elk geval*:

```text
match (:word{lemma:'in'})-[:next]->
      (:word{lemma:'elk'})-[:next]->
      (:word{lemma:'geval'})-[:next]->(p)
return p
```

Het volgende voorbeeld zoekt naar adjectieven die in het patroon *vinden het ADJ dat* voorkomen:

```text
match (:word{lemma:'vinden'})-[:next]->
      (:word{lemma:'het'})-[:next]->
      (adj:word{pt:'adj'})-[:next]->
      (:word{lemma:'dat'})
return adj
```

De beschikbaarheid van de `:next`-relatie zorgt ervoor dat deze voorbeelden heel eenvoudig zijn. In vergelijking hiermee zijn voorbeelden met XPath in PaQu erg omslachtig.


## Multi-word Units

Multi-word units (mwu) worden steeds gerepresenteerd als een knoop direct boven de woorden waaruit de mwu bestaat. Als extraatje heeft deze niet-lexicale knoop toch een waarde voor de attributen `word` en `lemma`. Je kunt dus eenvoudig zoeken naar een specifieke mwu:

```text
match (p:node{word:'ten opzichte van'})
return p
```

Bovendien kun je dus queries formuleren zonder dat je een verschil moet maken tussen mwu en woorden, indien je de lemma of het woord van een gevonden knoop terug wilt geven:

```text
match (:node)-[:rel{rel:'svp'}]->(p:nw)
return p.word
```
Dit levert een tabel op waarin zowel woorden als mwu terecht komen.

Je kunt natuurlijk ook specifiek naar een deel van een mwu zoeken. De volgende query geeft mwu terug waarbij een van de onderdelen het woord *van* is:

```text
match (n:node{cat:'mwu'})-[:rel{rel:'mwp'}]->(:word{lemma:'van'})
return n
```

Het is iets ingewikkelder om te formuleren dat het woord *van* het laatste woord van de mwu moet zijn:

```text
match (n:node{cat:'mwu'})-[:rel{rel:'mwp'}]->(w:word{lemma:'van', end: w.end})
return n
```

Of waarbij *van* het derde woord moet zijn:

```text
match (n:node{cat:'mwu'})-[:rel{rel:'mwp'}]->(w:word{lemma:'van', begin: n.begin + 2})
return n
```

Een lastiger vraagstuk is een query waarbij je eisen wilt stellen aan
de attributen van een woord, ofwel aan de attributen van een mwu.
Bijvoorbeeld: geef me elke naam, waarbij een naam een mwu is waarvoor
geldt dat een van de delen van de mwu de `postag` *SPEC(deeleigen)*
heeft, ofwel het is geen mwu maar een woord waarvoor geldt dat het
attribuut `ntype` de waarde *eigen* heeft.

```text
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

```text
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

```text
match (:word{lemma:'gebruiken'})-[:ud{rel:'obj'}]->(v)
return v.lemma
```

Als je de resultaten bekijkt op basis van de Alpino Treebank, krijg je lemma's zoals *machine*, *woord*, *stookolie*, *traangas*, maar bijvoorbeeld ook *soft*. Dat is eigenlijk een halve hit omdat het in de betreffende zin over *soft drugs* gaat. In UD is er in dit geval een *fixed* relatie van het eerste woord van de MWU met het tweede woord. Het zou aardig zijn als je in het overzicht van de lemma's dus *soft drug* in plaats van *soft* terug zou zien. Dat kan met het volgende recept, dat de UD relaties combineert met de relaties in de oorspronkelijke Alpino boom en vertrouwt op het feit dat MWU knopen ook een attribuut `lemma` (en `word`) hebben.

```text
match (:word{lemma:'gebruiken'})-[:ud{rel:'obj'}]->(v1:word)<-[:rel*0..1{rel:'mwp'}]-(v)
where not exists ( (v)<-[:rel{rel:'mwp'}]-()  )
return v.lemma
```

## Tellen van combinaties van waardes

Je kunt vrij gemakkelijk combinaties van de waardes van attributen tellen. Als voorbeeld kijken we hier naar werkwoorden met een vaste uitdrukking:

```text
match (v1:word{pt:'ww'})<-[:rel{rel:'hd'}]-()-[:rel{rel:'svp'}]->(v2:node)
return v1.lemma, v2.lemma
```

Deze paren van lemma's willen we vervolgens tellen en sorteren. Dat kan als volgt:

```text
match (v1:word{pt:'ww'})<-[:rel{rel:'hd'}]-()-[:rel{rel:'svp'}]->(v2:node)
return v1.lemma, v2.lemma, count(v1.lemma + ' ' + v2.lemma) as aantal
order by aantal desc
```

Als een ander voorbeeld kun je dus gemakkelijk een frequentieoverzicht maken van de meest voorkomende paren van woordsoorten:

```text
match (w1)-[:next]->(w2)
return w1.postag, w2.postag, count(w1.postag + ' ' + w2.postag) as aantal
order by aantal desc
```

Of een overzicht van zelfstandige naamwoorden en de relatieve voornaamwoorden waarmee ze gecombineerd worden:

```text
match (w1)<-[:rel{rel:'hd'}]-()-[:rel]->(:node{cat:'rel'})-[:rel{rel:'rhd'}]->(w2:word)
return w1.word, w2.word, count(w1.word + ' ' + w2.word) as aantal
order by aantal desc
```


## Matches binnen matches

Je kunt queries soms opbouwen door een eerste selectie te maken en dan binnen die selectie een verdere selectie uit te voeren.

Stel dat we op zoek willen naar cross-serial verb clusters. Eén aanpak bestaat eruit om eerst de werkwoordclusters te identificeren, en vervolgens daarbinnen die gevallen te selecteren waarbij een werkwoord uit het cluster een lijdend voorwerp selecteert dat ook fungeert als het onderwerp van het VC-complement:

```text
match (x)<-[:rel]-(n:node{cat: 'inf'})<-[:rel{rel: 'vc'}]-(n1)-[:rel{rel: 'hd'}]->(w:word{pt: 'ww'})
where x.begin < w.begin
match (n)-[:rel{rel: 'su'}]->()<-[:rel{rel: 'obj1'}]-(n1)
return n
```

Het gebruik van de techniek om een patroon onder te verdelen in meerdere matches maakt de queries soms makkelijker te begrijpen.




## Tellen van reeksen

Stap 1: zoek iets

```text
match (n:node{_deste: true})
return distinct n.sentid as sid, n.id as id
order by sid, id
```

Stap 2: zoek de pt van woorden onder iets

<pre><code class="text">match (n:node{_deste: true})
<span class="prediff">match (n)-[:rel*]->(w:word)</span>
return distinct n.sentid as sid, n.id as id<span class="prediff">, w.end as positie, w.pt as pt</span>
order by sid, id<span class="prediff">, positie</span>
</code></pre>

Stap 3: voeg de pt van woorden per iets samen, de oude `return` wordt `with`

<pre><code class="text">match (n:node{_deste: true})
match (n)-[:rel*]->(w:word)
<span class="prediff">with</span> distinct n.sentid as sid, n.id as id, w.end as positie, w.pt as pt
order by sid, id, positie
<span class="prediff">return sid, id, collect(pt) as pt_list</span>
</code></pre>


Stap 3a: variant om te kijken of de woorden wel direct achter elkaar staan

<pre><code class="text">match (n:node{_deste: true})
match (n)-[:rel*]->(w:word)
with distinct n.sentid as sid, n.id as id, w.end as positie, <span class="prediff">w.end + ':' +</span> w.pt as pt
order by sid, id, positie
return sid, id, collect(pt) as pt_list
</code></pre>

Stap 4: tel de frequenties van pt van woorden onder iets

<pre><code class="text">match (n:node{_deste: true})
match (n)-[:rel*]->(w:word)
with distinct n.sentid as sid, n.id as id, w.end as positie, w.pt as pt
order by sid, id, positie
<span class="prediff">with</span> sid <span class="prediff">as sid</span>, id <span class="prediff">as id</span>, collect(pt) as pt_list
<span class="prediff">return count(pt_list) as aantal, pt_list</span>
</code></pre>

## Meta-data

### Overzicht van de meta-data

Als er voor een corpus meta-data beschikbaar is, dan zijn er knopen met als type :meta. De aard van de meta-data is verschillend per corpus, en daarom ook de attributen die voor :meta beschikbaar zijn. Maar er is altijd het attribuut `sentid` en de knopen van de syntactische analyse hebben ook allemaal dat `sentid` attribuut met dezelfde waarde. Je kunt dus met een where-clause de bijbehorende meta-data terugvinden.

De volgende query toont voor een corpus welk type meta-data beschikbaar is:

```text
match (m:meta)
return m.name, count(m.name)
```

De counts geven in dit geval aan voor hoeveel zinnen de meta-data van dat type beschikbaar is. En om te zien welke waardes in de meta-data worden gebruikt, kun je zoiets doen. Hier is 'country' dan een van de types meta-data die je in de vorige query hebt opgeleverd.

```text
match (m:meta{name:'country'})
return m.value, count(m.value)
```

Je kunt dit ook doen voor meerdere types meta-data tegelijk. Bijvoorbeeld, hoe is de man/vrouw verhouding voor het Nederlandse en Vlaamse deel van het CGN. Hier gebruiken we het attribuut `sentid` om te zien welke meta-data bij elkaar hoort.

```text
match (m:meta{name:'country'}),
      (m2:meta{name:'sex', sentid:m.sentid})
return m.value, m2.value, count(m.value + ' ' + m2.value)
```


### Gebruik van meta-data in queries

Als voorbeeld nemen we de volgende query die zoekt naar gevallen waarbij een NP de eerste woordgroep van een zin is, en als onderwerp fungeert:

```text
match (n:nw{_vorfeld: true, _np: true})<-[r:rel{rel:'su'}]-()
return n
```

Als we van de matchende zin ook de meta-data willen opleveren kunnen we gebruik maken van het feit dat alle knopen ook het attribuut `sentid` bevatten. Daarmee halen we dan de relevante meta-data op:

```text
match (n:nw{_vorfeld: true, _np: true})<-[r:rel{rel:'su'}]-(),
      (m:meta{name:'country', sentid: n.sentid})
return n, m.value
```

Je kunt dus ook eisen aan de relevante meta-data stellen. Of delen van de meta-data teruggeven. Of tellen. Dit voorbeeld werkt voor het CGN corpus:

```text
match (n:nw{_vorfeld: true, _np: true})<-[r:rel{rel:'su'}]-(),
      (m:meta{name:'country', sentid: n.sentid})
return m.value, count(m.value)
```

En ook meta-data combineren:

```text
match (n:nw{_vorfeld: true, _np: true})<-[r:rel]-(),
      (m:meta{name:'country', sentid: n.sentid}),
      (m2:meta{name:'sex', sentid: n.sentid})
where r.rel in ['su','sup'] and r.id is null
return m.value, m2.value, count(m.value + ' ' + m2.value)
```



## Aardige voorbeelden die nergens passen

### Wie is de baas?

De volgende query verzameld onderwerpen van het werkwoord *beslissen* waarbij dat onderwerp ofwel een multi-word unit is, ofwel een zelfstandig naamwoord.

```text
match (:word{lemma:'beslissen'})-[:pair{rel:'su'}]->(v)
where v.cat='mwu' or v.pt='n'
return v
```

Als je in dergelijke queries meerdere mogelijke lemma's wilt noemen kan dat met een `WHERE` clause:

```text
match (h:word)-[:pair{rel:'su'}]->(v)
where h.lemma in ['beslissen','besluiten']
  and (v.cat='mwu' or v.pt='n')
return v
```

### Man/vrouw verdeling

De volgende twee queries kun je gebruiken om eventueel verschillen te vinden tussen zelfstandige naamworden die met possesief *haar* of *zijn* worden gebruikt.

```text
match (w1:word)-[:ud{rel:'nmod:poss'}]->(w2:word{lemma:'haar'})
return w1
```


```text
match (w1:word)-[:ud{rel:'nmod:poss'}]->(w2:word{lemma:'zijn'})
return w1
```

Voor *haar* levert dat als top-10 op: *man, vriend, kind, moeder, dochter, zoon, vader, leven, rol, echt_genoot*. Voor *zijn* krijgen we de volgende lijst (DutchWebCorpus): *vrouw, ploeg, naam, auto, vader, collega, vriendin, leven, zoon, club*.

Merk op dat je bij deze queries wellicht de mededeling "Afgebroken" krijgt van AlpinoGraph, als je de tab "woorden" of "lemma's" gebruikt. Als je de telling voor het hele corpus wilt uitvoeren kun je dat doen door de telling expliciet in de query zelf uit te voeren:

```text
match (w1:word)-[:ud{rel:'nmod:poss'}]->(w2:word{lemma:'haar'})
return w1.lemma, count(w1.lemma) as aantal
order by aantal desc
```

### Paren van coördinatoren

In het volgende voorbeeld zoeken we naar een conjunctie met twee
coördinatoren (*noch .. noch..*, *zowel .. als ..*). De eerste match
zorgt voor een coördinatie die twee coördinatoren bevat. De tweede
match eist dat de eerste coördinator vooraf gaat aan de tweede - op
die manier krijg je elke conjunctie maar één keer, en de betreffende
lemma's in de verwachte volgorde:

```text
match (v1:word) <-[:rel{rel:'crd'}]-(:node{cat: 'conj'})-[:rel{rel: 'crd'}]->(v2:word)
where v1.end < v2.end
return v1.lemma, v2.lemma, count(v1.lemma + ' ' + v2.lemma) as aantal
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

```text
match (w1:word{lemma: 'fiets'})
with distinct w1.word as word
match (w:word{word: word})
return distinct w.lemma, w.word, w.root, w.pt
order by lemma, root, pt, word
```

### Woorden die in verschillende soorten constructies gebruikt worden

TODO: plek, titel, toelichting (Gertjan)

```
match (w1:word{pt:'ww'})<-[:rel{rel:'hd'}]-()
         -[:rel{rel:'vc'}]->(:node{cat:'whsub'})
with w1.lemma as lemma1, count(w1.lemma) as n
where n > 1
match (w:word{pt:'ww'})<-[:rel{rel:'hd'}]-()
         -[:rel{rel:'obj1'}]->(:node{cat:'whrel'})
where w.lemma in lemma1
return w
```

### Vorfeld

TODO

## Sonar

!!! warning "Let op"
    Dit is alleen voor het corpus Lassy Klein

Het corpus Lassy Klein bevat *named entities* die afkomstig zijn uit
SONAR, een onderdeel van Lassy. TODO

Zo vind je het eerste woord van alle named entities:

```text
match (w:word)
where w.begin = w.sonar_ne_begin
return w
```

Sommige named entities lopen door naar een volgende zin, bijvoorbeeld
een boektitel die uit meerde woorden bestaat. Deze gevallen
vind je zo:

```text
match (w:word)
where w.begin = w.sonar_ne_begin
match (w2:word{sentid: w.sentid, last: true})
where w2.end < w.sonar_ne_end
return w
```

En zo vind je de stukken die horen bij een named entity die in een
vorige zin is begonnen:

```text
match (w:word{begin: 0})
where w.sonar_ne_begin < 0
return w
```

Zo vind je alle named entities, compleet en incompleet, met
bijbehorende woorden:

```text
match p = (w:word)-[:next*0..]->(w2:word{sentid: w.sentid})
where ( w.begin = w.sonar_ne_begin or ( w.begin = 0 and w.sonar_ne_begin < 0 ) )
  and ( w2.end = w.sonar_ne_end or ( w2.last = true and w.sonar_ne_end > w2.end ) )
return p
```

En zo vind je alleen de complete named entities:

```text
match p = (w:word)-[:next*0..]->(w2:word{sentid: w.sentid})
where w.begin = w.sonar_ne_begin
  and w2.end = w.sonar_ne_end
return p
```

Zo vind je named entities die maar uit één woord bestaan:

```text
match (w:word)
where w.begin = w.sonar_ne_begin
  and w.end = w.sonar_ne_end
return w
```

Dit geeft hetzelfde resultaat:

```text
match (w:word)
where w.sonar_ne is not null
return w
```

Bij veel, maar lang niet alle, named entities van meerdere woorden hebben
de woorden samen en met geen enkel ander woord één gezamelijke parent. Zo'n
parent vind je zo:

```text
match (n:node)
where n.sonar_ne is not null
return n
```

En uiteraard kun je zoeken op een specifieke soort named entity:

```text
match p = (w:word{sonar_ne_class: 'loc'})-[:next*0..]->(w2:word)
where w.begin = w.sonar_ne_begin
  and w2.end = w.sonar_ne_end     -- NIET w2.sonar_ne_end
return p
```

Of:

```text
match (n:nw{sonar_ne: 'pro'})
return n
```

De laatste vindt niet alles, omdat het attribuut `sonar_ne` niet
altijd aanwezig is (zie boven).
