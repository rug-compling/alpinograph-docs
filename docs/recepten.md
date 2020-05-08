# Receptenboek

## Ngram

De beschikbaarheid van de relatie :next maakt het zeer eenvoudig om naar specifieke woord-sequenties te zoeken. De volgende query vindt het trigram "in elk geval":

```text
match p = (:word{lemma:'in'})-[:next]->
          (:word{lemma:'elk'})-[:next]->
          (:word{lemma:'geval'})
return p
```

En het is niet veel ingewikkelder om naar woorden in de context van een Ngram te zoeken. Welke woorden treden op na het trigram "in elk geval":

```text
match (:word{lemma:'in'})-[:next]->
      (:word{lemma:'elk'})-[:next]->
      (:word{lemma:'geval'})-[:next]->(p)
return p
```

Het volgende voorbeeld zoekt naar adjectieven die in het patroon "vinden het ADJ dat" voorkomen:

```text
match (:word{lemma:'vinden'})-[:next]->
      (:word{lemma:'het'})-[:next]->
      (adj:word{pt:'adj'})-[:next]->
      (:word{lemma:'dat'})
return adj
```

De beschikbaarheid van de :next relatie zorgt ervoor dat deze voorbeelden heel eenvoudig zijn. In vergelijking hiermee zijn voorbeelden met XPath in PaQu erg omslachtig.


## Multi-word Units

Multi-word units (mwu) worden steeds gerepresenteerd als een knoop direct boven de woorden waaruit de mwu bestaat. Als extraatje heeft deze niet-lexicale knoop toch een waarde voor de attributen 'word' en 'lemma'. Je kunt dus eenvoudig zoeken naar een specifieke mwu:

```text
match (p:node{word:'ten opzichte van'})
return p
```

Bovendien kun je dus queries formuleren zonder dat je een verschil moet maken tussen mwu en woorden, indien je de lemma of het woord van een gevonden knoop terug wilt geven:

```text
match  (:node)-[:rel{rel:'svp'}]->(p:nw)
return p.word
```
Dit levert een tabel op waarin zowel woorden als mwu terecht komen.

Je kunt natuurlijk ook specifiek naar een deel van een mwu zoeken. De volgende query geeft mwu terug waarbij een van de onderdelen het woord "van" is:

```text
match (n:node{cat:'mwu'})-[:rel{rel:'mwp'}]->(:word{lemma:'van'})
return n
```

Het is iets ingewikkelder om te formuleren dat het woord "van" het laatste woord van de mwu moet zijn:

```text
match (n:node{cat:'mwu'})-[:rel{rel:'mwp'}]->(w:word{lemma:'van'})
where n.end = w.end
return n
```

Of waarbij "van" het derde woord moet zijn:

```text
match (n:node{cat:'mwu'})-[:rel{rel:'mwp'}]->(w:word{lemma:'van'})
where w.begin = n.begin + 2
return n
```

Een lastiger vraagstuk is een query waarbij je eisen wilt stellen aan de attributen van een woord, ofwel aan de attributen van een mwu. Bijvoorbeeld: geef me elke naam, waarbij een naam een mwu is waarvoor geldt dat een van de delen van de mwu de postag 'SPEC(deeleigen)' heeft, ofwel het is geen mwu maar een woord waarvoor geldt dat het attribuut 'ntype' de waarde 'eigen' heeft.

```text
match (w:word)<-[:rel*0..1{rel: 'mwp'}]-(n:nw)<-[r:rel]-()
where (      ( w.postag = 'SPEC(deeleigen)' or w.ntype = 'eigen') 
         and ( n.cat='mwu'or w.id = n.id )
         and r.rel != 'mwp'
      )
return distinct n
```
TODO: can this be done simpler?



## Tellen van reeksen

Stap 1: zoek iets

```text
match (n:node{_deste: true})
return distinct n.sentid as sid, n.id
order by sid, id
```

Stap 2: zoek de pt van woorden onder iets

<pre><code class="text">
match (n:node{_deste: true})
<span class="prediff">match (n)-[:rel*]->(w:word)</span>
return distinct n.sentid as sid, n.id<span class="prediff">, w.end as positie, w.pt as pt</span>
order by sid, id<span class="prediff">, positie</span>
</code></pre>

Stap 3: voeg de pt van woorden per iets samen

<pre><code class="text">
<span class="prediff">select sid, id, json_agg(pt) as pt_list
from (</span>
  match (n:node{_deste: true})
  match (n)-[:rel*]->(w:word)
  return distinct n.sentid as sid, n.id, w.end as positie, w.pt as pt
  order by sid, id, positie
<span class="prediff">) as foo
group by sid, id
order by sid, id</span>
</code></pre>

Stap 3a: maak het wat leesbaarder

<pre><code class="text">
select sid, id, <span class="prediff">string_agg(trim(both '"' from pt::text), ' ')</span> as pt_list
from (
  match (n:node{_deste: true})
  match (n)-[:rel*]->(w:word)
  return distinct n.sentid as sid, n.id, w.end as positie, w.pt as pt
  order by sid, id, positie
) as foo
group by sid, id
order by sid, id
</code></pre>

Stap 3b: variant om te kijken of de woorden wel direct achter elkaar staan

<pre><code class="text">
select sid, id, string_agg(trim(both '"' from pt::text), ' ') as pt_list
from (
  match (n:node{_deste: true})
  match (n)-[:rel*]->(w:word)
  return distinct n.sentid as sid, n.id, w.end as positie, <span class="prediff">w.end + ':' + w.pt</span> as pt
  order by sid, id, positie
) as foo
group by sid, id
order by sid, id
</code></pre>

Stap 4: tel de frequenties van pt van woorden onder iets

<pre><code class="text">
<span class="prediff">select count(pt_list) as aantal, pt_list
from (</span>
  select string_agg(trim(both '"' from pt::text), ' ') as pt_list
  from (
    match (n:node{_deste: true})
    match (n)-[:rel*]->(w:word)
    return distinct n.sentid as sid, n.id as id, w.end as positie, w.pt as pt
    order by sid, id, positie
  ) as foo
  group by sid, id
<span class="prediff">) as bar
group by pt_list
order by aantal desc, pt_list</span>
</code></pre>
