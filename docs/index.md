# Inleiding

AlpinoGraph is een tool om syntactisch geannoteerde corpora te doorzoeken. De tool maakt gebruik van AgensGraph. AgensGraph combineert databasetechnologie (PostgreSQL) en Cypher, de standaard zoektaal voor grafen. De zoek-queries die je in AlpinoGraph kunt gebruiken zijn daarom een mix van SQL en Cypher. Daar voegt AlpinoGraph nog enkele extra uitbreidingen aan toe, zoals een eenvoudig maar handig systeem van macro's, en visualizatie van de resultaten.

De combinatie van databasetechnologie en graaf-gebaseerde zoektechnologie zorgt voor een erg flexibele en relatief efficiënte zoekmachine. Deze flexibiliteit betekent bijvoorbeeld dat je patronen kunt definiëren om zinnen die aan het patroon te voldoen terug te vinden. Maar je kunt ook woorden of woordgroepen terugvinden en aggregeren over de informatie van die woorden of woordgroepen (bijvoorbeeld voor het maken van frequentieoverzichten) door middel van de database-primitieven van SQL. 

Niet alleen is de zoekmachne veel flexibeler dan bijvoorbeeld XPath (zoals beschikbaar in PaQu), ook is het relatief makkelijk om verschillende annotatielagen te combineren. In AlpinoGraph zijn meerdere annotatielagen beschikbaar:

- de CGN/Lassy/Alpino dependentiestructuren
- de hieruit automatisch afgeleide Universal Dependency structuren (zowel de standaard als de "enhanced" variant)
- de woord-paren relaties, zoals die in de eenvoudige zoektab in PaQu beschikbaar zijn

Deze drie lagen kunnen in queries indien gewenst eenvoudig gecombineerd worden.

In AlpinoGraph zijn een aantal syntactisch geannoteerde corpora beschikbaar. AlpinoGraph ondersteunt handmatig geverifieerde treebanks zoals CGN, Lassy Klein en de Alpino Treebank. Daarnaast worden ook automatisch door Alpino geanalyseerde corpora ondersteund, zoals Lassy Groot.

## Syntactische analyse als graaf

### Woorden

De syntactische analyse van een zin is beschikbaar, zoals hierboven al genoemd, in meerdere lagen: de dependentiestructuur zoals bekend van CGN/Lassy/Alpino, de Universal Dependency-structuur (zowel standaard als enhanced), en de woord-paren structuur van PaQu. Deze lagen zijn allemaal gebaseerd op de woorden van de zin. Deze woorden zijn de bouwstenen van de graaf die al deze annotatielagen combineert. De volgende query zoekt alle woorden waarvoor het lemma de waarde "fiets" heeft: 

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
- :pair (voor de PaQu woordpaarrelaties)
- :next (voor het volgende woord in de zin)

#### :rel

Om te zoeken naar een PP die een dochter heeft met het "hdf" relatie attribuut kun je de volgende query formuleren:

```text
match (n:node{cat:'pp'})-[:rel{rel:'hdf'}]->(:nw)
return n
```

We specificeren hier dus eerst via ":rel" dat het gaat om de Alpino relaties. En daarbinnen geven we aan dat de waarde van het "rel"-attribuut de waarde "hdf" heeft. Een edge met als type :rel vertrekt altijd vanuit een node en eindigt bij een :node of een :word.

#### :ud

Een universal dependency relatie wordt gerepresenteert met het :ud type. Zulke relaties bestaan alleen tussen woorden. De volgende query vindt alle lijdend voorwerpen van 'drinken' waarbij het attribuut UPOS de waarde 'NOUN' heeft:

```text
match (:word{lemma:'drinken'})-[:ud{main:'obj'}]->(o:word{upos:'NOUN'})
return o
```

#### :next

Een woord heeft altijd een relatie met als type :next naar het volgende woord. Het is dus erg eenvoudig om te zoeken naar een bigram, bijvoorbeeld: "in geval":

```text
match (w1:word{lemma:'in'})-[:next]->(w2:word{lemma:'geval'})
return w1,w2
```

Dit voorbeeld toont ook aan dat het heel wel mogelijk is om als resultaat van een query meerdere knopen terug te geven: in dit geval de variabelen w1 en w2.

Het is ook mogelijk om patronen te maken waarbij meerdere knopen en relaties tegelijk voorkomen. De volgende query identificeert woorden die direct volgen op "in geval":

```text
match (:word{lemma:'in'})-[:next]->(:word{lemma:'geval'})-[:next]->(w:word)
return w
```

### Wat geeft de query als resultaat terug?

In veel gevallen wordt een query gedefinieerd waarbij een knoop wordt gespecificeerd die je als resultaat terug wilt krijgen.  Zo'n typisch geval is de volgende query, waarbij we via de Alpino relaties zoeken naar lijdend voorwerpen van een werkwoord:

```text
match (:word{pt:'ww'})<-[:rel{rel:'hd'}]-(:node)-[:rel{rel:'obj1'}]->(w:node)
return w
```

Een voordeel in vergelijking met XPath is, dat je indien je dezelfde hierarchische relatie wilt beschrijven, maar een andere knoop als resultaat terug wilt krijgen, slechts de variabele op een andere plek in het patroon kunt plaatsen. In XPath moet je in zulke gevallen de query herschrijven. Hier wordt het:

```text
match (w:word{pt:'ww'})<-[:rel{rel:'hd'}]-(:node)-[:rel{rel:'obj1'}]->(:node)
return w
```

En het is dus ook mogelijk om beide knopen als resultaat terug te geven, indien gewenst:

```text
match (v:word{pt:'ww'})<-[:rel{rel:'hd'}]-(:node)-[:rel{rel:'obj1'}]->(w:node)
return v, w
```

Een ander typisch voorbeeld is de optie om als resultaat van een query de waarde(s) van een of meer attributen te specificeren. Bijvoorbeeld:

```text
match (v:word{pt:'ww'})<-[:rel{rel:'hd'}]-(:node)-[:rel{rel:'obj2'}]->(w:node)
return v.lemma, w.cat
```

Er zijn nog allerlei andere mogelijkheden, waarvan een aantal in de volgende sectie wordt geillustreerd. Een speciaal geval is de optie om een sub-graaf als resultaat terug te geven. Dat kan bijvoorbeeld als volgt:

```text
match p=(:word{pt:'ww'})<-[:rel{rel:'hd'}]-(:node)-[:rel{rel:'obj1'}]->(:node)
return p
```

TODO: liever hier een voorbeeld waar dit ook zinvol is.


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

- optional match voor negatie
- where exists
- ...

## Resultaten van een query in AlpinoGraph

In het algemeen levert elke query een tabel op. Maar AlpinoGraph behandelt niet elke tabel op dezelfde wijze. In de gevallen waarbij de query een knoop in de graaf of een deelgraaf oplevert, kiest AlpinoGraph ervoor om de zin die hoort bij die knoop of deelgraaf als resultaat te presenteren. Als je vervolgens op een zin klikt krijg je de visualizatie van de graaf te zien, in meerdere varianten.

Indien gewenst kun je alsnog de resultaten in tabelvorm bekijken door op de betreffende button te klikken. Twee andere opties zijn "woorden" en "lemma's". In die laatste gevallen wordt per hit de woordgroep (in woorden dan wel in lemma's) opgehaald die door de matchende knoop wordt gedomineerd. Van al die woordgroepen wordt vervolgens een frequentieoverzicht gemaakt.

Dus voor deze query:

```text
match (w:word{lemma:'zijn'})
return w.word
```

krijg je alleen een tabel te zien, terwijl je voor de volgende query de zinnen terug krijgt:


```text
match (w:word{lemma:'zijn'})
return w
```

Je kunt hier dus nu kiezen voor de andere opties. Als je nu voor "tabel" kiest, krijg je erg veel informatie: alle attributen van elke matchende knoop. Kies je voor "woorden", dan krijg je een mooi frequentie-overzicht van alle vormen van het werkwoord 'zijn'. In dit specifieke geval is het niet erg inzichtelijk om te kiezen voor "lemma's". 

Het volgende voorbeeld suggereert wanneer het nuttig kan zijn om voor "lemma's" te kiezen:

```text
match (:word{lemma:'eten'})-[:ud{main:'obj'}]->(w2:word)
return w2
```

Dit levert dus alle zinnen op waarin "eten" een lijdend voorwerp heeft. En als je op "lemma's" klikt krijg je een mooi frequentieoverzicht van al de (hoofden van) die lijdend voorwerpen.

In bovenstaande gevallen zijn de matchende knopen steeds woorden, maar dit werkt dus op vergelijkbare wijze voor hogere knopen. De volgende query vindt alle hogere knopen die in relatie "svp" met een werkwoord staan:

```text
match (:node)-[:rel{rel:'svp'}]->(w2:node)
return w2
```

Omdat de query een node oplevert krijg je per default de zinnen te zien waarin een match voorkomt. Als je kiest voor "lemma's" levert dat dus een frequentieoverzicht op van de betreffende woordgroepen.

In sommige gevallen bevatten die woordgroepen gaten (discontinue woordgroepen). Een typisch voorbeeld hiervan levert de volgende query:

```text
match (:node)-[:rel{rel:'pc'}]->(w2:node)
return w2
```

Je ziet dat dan in het frequentieoverzicht van de woorden of de lemma's met "..." de gaten in de woordgroepen worden aangegeven.

In het receptenboek wordt uitgelegd hoe je queries kunt formuleren die op vergelijkbare wijze aggregeren over de waardes van attributen indien het andere attributen betreft dan "word" of "lemma".

## Geavanceerd zoeken met CYPHER

- 0..1* 

## Berekende attributen (of ergens anders)

- attributen zoals '_vorfeld', documenteer hoe die zijn gevonden

TODO
