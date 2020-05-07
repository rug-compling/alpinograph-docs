# Inleiding

AlpinoGraph is een tool om syntactisch geannoteerde corpora te doorzoeken. De tool maakt gebruik van AgensGraph. AgensGraph combineert database technologie (PostgresSQL) en Cypher, de standaard zoektaal voor grafen. De zoek-queries die je in AlpinoGraph kunt gebruiken zijn daarom een mix van SQL en Cypher. Daar voegt AlpinoGraph nog enkele extra uitbreidingen aan toe, zoals een eenvoudig maar handig systeem van macro's, en visualizatie van de resultaten.

De combinatie van databasetechnologie en graaf-gebaseerde zoektechnologie zorgt voor een erg flexibele en relatief efficiënte zoekmachine. Deze flexibiliteit betekent bijvoorbeeld dat je patronen kunt definiëren om zinnen die aan het patroon te voldoen terug te vinden. Maar je kunt ook woorden of woordgroepen terugvinden en aggregeren over de informatie van die woorden of woordgroepen (bijvoorbeeld voor het maken van frequentieoverzichten) door middel van de database-primitieven van SQL. 

Niet alleen is de zoekmachne veel flexibeler dan bijvoorbeeld XPath (zoals beschikbaar in PaQu), ook is het relatief makkelijk om verschillende annotatielagen te combineren. In AlpinoGraph zijn drie van zulke lagen beschikbaar:

- de CGN/Lassy/Alpino dependentiestructuren
- de hieruit automatisch afgeleide Universal Dependency structuren (zowel de standaard als de "enhanced" variant)
- de woord-paren relaties, zoals die in de eenvoudige zoektab in PaQu beschikbaar zijn

Deze drie lagen kunnen in queries indien gewenst eenvoudig gecombineerd worden.

In AlpinoGraph zijn een aantal syntactisch geannoteerde corpora beschikbaar. AlpinoGraph ondersteunt handmatig geverifieerde treebanks zoals CGN, Lassy Klein en de Alpino Treebank. Daarnaast worden ook automatisch door Alpino geanalyseerde corpora ondersteund, zoals Lassy Groot.

## Syntactische analyse als graaf

### Woorden

De syntactische analyse van een zin is beschikbaar, zoals hierboven al genoemd, in meerdere lagen: de dependentiestructuur zoals bekend van CGN/Lassy/Alpino, de Universal Dependency-structuur (zowel standaard als enhanced), en de woord-paren structuur van PaQu. Deze vier lagen zijn allemaal gebaseerd op de woorden van de zin. Deze woorden zijn de bouwstenen van de graaf die al deze annotatielagen combineert. De volgende query zoekt alle woorden waarvoor het lemma de waarde "fiets" heeft: 

```text
match (n:word{lemma: 'fiets'}) 
return n;
```

Een woord in de graafstructuur is een knoop van het type "word". Een knoop ziet eruit als 

```text
(:word)

(:word{...})
```

waarbij tussen de accolades dan attributen en waardes kunnen worden gespecificeerd. In het voorbeeld fungeert "n" als een variabele.

### Andere knopen

Naast woorden zijn er ook nog de andere knopen voor bijvoorbeeld NP, PP, SMAIN. Deze knopen zien eruit als

```text
(:node)

(:node{...})
```

Je kunt dus bijvoorbeeld zoeken naar alle niet-lexicale knopen met als categorie PP:

```text
match (n:node{cat: 'pp'}) 
return n
```

Indien je wilt zoeken naar een knoop maar die knoop mag zowel een woord zijn als een hogere knoop, dan gebruik je de notatie

```text
(:nw)

(:nw{...})
```

### Verbindingen tussen knopen

Je kunt dus direct naar woorden en woordgroepen zoeken, maar het wordt pas echt een klein beetje interessant wanneer je ook relaties tussen woorden en woordgroepen kunt specificeren. In Cypher ziet zo'n relatie tussen twee knopen (woorden of woordgroepen) er zo uit, naar keuze:

```text
() -[]-> () 

() <-[]- ()
```


Tussen de ronde haken staat dan verder de informatie van de relevante knopen waartussen de relatie bestaat. Tussen de vierkante haken staat verdere informatie over de aard van de relatie (de "edge" in grafentaal).

Er zijn meerdere soorten relaties beschikbaar, waaronder:

- :rel (zoals het @rel attribuut in Alpino)
- :ud (voor standaard universal dependencies)
- :eud (voor enhanced universal dependencies)
- :pair (voor de PaQu relaties tussen woorden)
- :next (voor het volgende woord in de zin)

#### :rel

Om te zoeken naar een PP die een dochter heeft met het "hdf" relatie attribuut kun je de volgende query formuleren:

```text
match (n:node{cat:'pp'})-[:rel{rel:'hdf'}]->(:nw)
return n
```

We specificeren hier dus eerst via ":rel" dat het gaat om de Alpino relaties. En daarbinnen geven we aan dat de waarde van het "rel"-attribuut de waarde "hdf" heeft. Een edge met als type :rel vertrekt vanuit een :node en eindigt bij een :node of een :word.

#### :ud

Een universal dependency edge wordt gerepresenteert met het ud: type. Zulke edges bestaan alleen tussen woorden. De volgende query vindt alle lijdend voorwerpen met de waarde "NOUN" voor het attribuut UPOS van "drinken":

```text
match (:word{lemma:'drinken'})-[:ud{main:'obj'}]->(o:word{upos:'NOUN'})
return o
```

#### :next

Een woord heeft altijd een edge met als type :next naar het volgende woord. Het is dus erg eenvoudig om te zoeken naar een bigram, bijvoorbeeld: "in geval":

```text
match (w1:word{lemma:'in'})-[:next]->(w2:word{lemma:'geval'})
return w1,w2
```

Dit voorbeeld toont ook aan dat het heel wel mogelijk is om als resultaat van een query meerdere knopen terug te geven: in dit geval de variabelen w1 en w2.

Het is ook mogelijk om patronen te maken waarbij meerdere nodes en edges tegelijk voorkomen. De volgende query identificeert woorden die direct volgen op de bigram "in geval":

```text
match (:word{lemma:'in'})-[:next]->(:word{lemma:'geval'})-[:next]->(w:word)
return w
```


## Flexibel zoeken met SQL 

Hierboven waren de voorbeelden steeds van het type: match een bepaald patroon, en geef een of meerdere delen van de match terug als resultaat. Door gebruik te maken van SQL is hier veel meer mogelijk. Hieronder tonen we een paar veelgebruikte technieken, zonder de ambitie een tutorial voor SQL te verzorgen.

### where clause

Verdere condities aan een patroon kun je (ook) specificeren met behulp van een "where"-clause. Bijvoorbeeld, de query die naar lijdend voorwerpen van "drinken" zoekt kun je uitbreiden door ook varianten van "drinken" toe te staan:

```text
match (w1:word)-[:ud{main:'obj'}]->(o:word{upos:'NOUN'})
where w1.lemma in ['eten','op_eten','drinken','op_drinken']
return o
```

Zo'n where clause is ook een makkelijke manier om uit te drukken dat een bepaalde waarde nu juist niet mag voorkomen. In het volgende voorbeeld zoeken we werkwoorden met lijdend voorwerpen waarbij dat lijdend voorwerp geen voornaamwoord mag zijn:

```text
match (w1:word{upos:'VERB'})-[:ud{main:'obj'}]->(w2:word)
where w2.upos != 'PRON'
return w1, w2
```

### tellen en sorteren

Vaak is het interessant om te aggregeren over de relevante delen van een match. Dat is natuurlijk makkelijk in een database te doen. 

```text
match (w:word)-[:ud{main:'obj'}]->(w2:word{upos:'NOUN'})
where w.lemma in ['eten','op_eten','drinken','op_drinken']
return w2.lemma, count(w2.lemma)
```

Omdat in zulke gevallen het resultaat geen node of word is, zal in de AlpinoGraph interface de resulterende tabel worden getoond (in plaats van de gevonden zinnen met de matchende delen). De namen van de kolommen van de tabel kun je eventueel expliciet aangeven met "as Name", zoals in dit voorbeeld:

```text
match (w:word)-[:ud{main:'obj'}]->(w2:word{upos:'NOUN'})
where w.lemma in ['eten','op_eten','drinken','op_drinken']
return w2.lemma as woord, count(w2.lemma) as aantal
```

De namen van de kolommen zijn ook relevant indien je wilt sorteren. Je verwijst dan naar de kolom op basis waarvan je wilt sorteren. 

```text
match (w:word)-[:ud{main:'obj'}]->(w2:word{upos:'NOUN'})
where w.lemma in ['eten','op_eten','drinken','op_drinken']
return w2.lemma as woord, count(w2.lemma) as aantal
order by aantal
```

Je kunt natuurlijk ook omgekeerd sorteren, dat gaat als volgt:

```text
match (w:word)-[:ud{main:'obj'}]->(w2:word{upos:'NOUN'})
where w.lemma in ['eten','op_eten','drinken','op_drinken']
return w2.lemma as woord, count(w2.lemma) as aantal
order by aantal desc
```

En ten slotte kun je nog op basis van het woord sorteren indien de tellingen gelijk zijn, door een volgende kolomnaam toe te voegen bij het "order" regel:

```text
match (w:word)-[:ud{main:'obj'}]->(w2:word{upos:'NOUN'})
where w.lemma in ['eten','op_eten','drinken','op_drinken']
return w2.lemma as woord, count(w2.lemma) as aantal
order by aantal desc, woord
```

### TODO meer SQL trucjes

## Geavanceerd zoeken met CYPHER

TODO
