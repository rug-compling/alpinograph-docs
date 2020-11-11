# Zoeken met AlpinoGraph

De combinatie van databasetechnologie en graaf-gebaseerde zoektechnologie zorgt voor een erg flexibele en relatief efficiënte zoekmachine. Deze flexibiliteit betekent bijvoorbeeld dat je patronen kunt definiëren om zinnen die aan het patroon te voldoen terug te vinden. Maar je kunt ook woorden of woordgroepen terugvinden en aggregeren over de informatie van die woorden of woordgroepen (bijvoorbeeld voor het maken van frequentieoverzichten) door middel van de database-primitieven van SQL.

Niet alleen is de zoekmachine veel flexibeler dan bijvoorbeeld XPath (zoals beschikbaar in [PaQu](https://www.let.rug.nl/alfa/paqu)), ook is het relatief makkelijk om verschillende annotatielagen te combineren. In AlpinoGraph zijn meerdere annotatielagen beschikbaar:

- de CGN/Lassy/Alpino dependentiestructuren
- de hieruit automatisch afgeleide Universal Dependency structuren (zowel de standaard als de "enhanced" variant)
- de woord-paren relaties, zoals die in de eenvoudige zoektab in PaQu beschikbaar zijn

Deze drie lagen kunnen in queries indien gewenst eenvoudig gecombineerd worden.

In AlpinoGraph zijn een aantal syntactisch geannoteerde corpora beschikbaar. AlpinoGraph ondersteunt handmatig geverifieerde treebanks zoals CGN, Lassy Klein en de Alpino Treebank. Daarnaast worden ook automatisch door Alpino geanalyseerde corpora ondersteund, zoals Lassy Groot.

Hieronder volgt een beknopte inleiding, toegespitst op zoeken in
corpora met AlpinoGraph. Voor een complete handleiding van AgensGraph,
zie de
[AgensGraph Developer Manual](https://bitnine.net/documentations/manual/agens_graph_developer_manual_en.html).


## Syntactische analyse als graaf

### Woorden

De syntactische analyse van een zin is beschikbaar, zoals hierboven al genoemd, in meerdere lagen: de dependentiestructuur zoals bekend van CGN/Lassy/Alpino, de Universal Dependency-structuur (zowel standaard als enhanced), en de woord-paren structuur van PaQu. Deze lagen zijn allemaal gebaseerd op de woorden van de zin. Deze woorden zijn de bouwstenen van de graaf die al deze annotatielagen combineert. De volgende query zoekt alle woorden waarvoor het lemma de waarde *fiets* heeft:

```cypher
match (n:word{lemma: 'fiets'})
return n
```

Een woord in de graafstructuur is een knoop van het type `word`. Een knoop ziet eruit als

```cypher
(:word)

(:word{...})
```

waarbij tussen de accolades dan attributen en waardes kunnen worden gespecificeerd. In het voorbeeld fungeert `n` als een variabele.

### Andere knopen

Naast woorden zijn er ook nog de andere knopen voor bijvoorbeeld NP, PP, SMAIN. Deze knopen zien eruit als

```cypher
(:node)

(:node{...})
```

Je kunt dus bijvoorbeeld zoeken naar alle niet-lexicale knopen met als categorie PP:

```cypher
match (n:node{cat: 'pp'})
return n
```

Indien je wilt zoeken naar een knoop maar die knoop mag zowel een woord zijn als een hogere knoop, dan gebruik je de notatie

```cypher
(:nw)

(:nw{...})
```

### Verbindingen tussen knopen

Je kunt dus direct naar woorden en woordgroepen zoeken, maar het wordt pas echt een klein beetje interessant wanneer je ook relaties tussen woorden en woordgroepen kunt specificeren. In Cypher ziet zo'n relatie tussen twee knopen (woorden of woordgroepen) er zo uit, naar keuze:

```cypher
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

Om te zoeken naar een PP die een dochter heeft met het *hdf* relatie attribuut kun je de volgende query formuleren:

```cypher
match (n:node{cat:'pp'})-[:rel{rel:'hdf'}]->(:nw)
return n
```

We specificeren hier dus eerst via `:rel` dat het gaat om de Alpino relaties. En daarbinnen geven we aan dat de waarde van het `rel`-attribuut de waarde *hdf* heeft. Een edge met als type :rel vertrekt altijd vanuit een :node en eindigt bij een :node of een :word.

#### :ud

Een universal dependency relatie wordt gerepresenteert met het :ud type. Zulke relaties bestaan alleen tussen woorden. De volgende query vindt alle lijdend voorwerpen van 'drinken' waarbij het attribuut UPOS de waarde 'NOUN' heeft:

```cypher
match (:word{lemma:'drinken'})-[:ud{main:'obj'}]->(o:word{upos:'NOUN'})
return o
```

#### :next

Een woord heeft altijd een relatie met als type :next naar het volgende woord. Het is dus erg eenvoudig om te zoeken naar een bigram, bijvoorbeeld: *in geval*:

```cypher
match (w1:word{lemma:'in'})-[:next]->(w2:word{lemma:'geval'})
return w1,w2
```

Dit voorbeeld toont ook aan dat het heel wel mogelijk is om als resultaat van een query meerdere knopen terug te geven: in dit geval de variabelen w1 en w2.

Het is ook mogelijk om patronen te maken waarbij meerdere knopen en relaties tegelijk voorkomen. De volgende query identificeert woorden die direct volgen op *in geval*:

```cypher
match (:word{lemma:'in'})-[:next]->(:word{lemma:'geval'})-[:next]->(w:word)
return w
```

### Wat geeft de query als resultaat terug?

In veel gevallen wordt een query gedefinieerd waarbij een knoop wordt gespecificeerd die je als resultaat terug wilt krijgen.  Zo'n typisch geval is de volgende query, waarbij we via de Alpino relaties zoeken naar lijdend voorwerpen van een werkwoord:

```cypher
match (:word{pt:'ww'})<-[:rel{rel:'hd'}]-(:node)-[:rel{rel:'obj1'}]->(w:node)
return w
```

Een voordeel in vergelijking met XPath is, dat je indien je dezelfde hierarchische relatie wilt beschrijven, maar een andere knoop als resultaat terug wilt krijgen, slechts de variabele op een andere plek in het patroon kunt plaatsen. In XPath moet je in zulke gevallen de query herschrijven. Hier wordt het:

```cypher
match (w:word{pt:'ww'})<-[:rel{rel:'hd'}]-(:node)-[:rel{rel:'obj1'}]->(:node)
return w
```

En het is dus ook mogelijk om beide knopen als resultaat terug te geven, indien gewenst:

```cypher
match (v:word{pt:'ww'})<-[:rel{rel:'hd'}]-(:node)-[:rel{rel:'obj1'}]->(w:node)
return v, w
```

Een ander typisch voorbeeld is de optie om als resultaat van een query de waarde(s) van een of meer attributen te specificeren. Bijvoorbeeld:

```cypher
match (v:word{pt:'ww'})<-[:rel{rel:'hd'}]-(:node)-[:rel{rel:'obj2'}]->(w:node)
return v.lemma, w.cat
```

Er zijn nog allerlei andere mogelijkheden, waarvan een aantal in de volgende sectie wordt geillustreerd. Een speciaal geval is de optie om een sub-graaf als resultaat terug te geven. Dat kan bijvoorbeeld als volgt:

```cypher
match p=(:word{pt:'ww'})<-[:rel{rel:'hd'}]-(:node)-[:rel{rel:'obj1'}]->(:node)
return p
```

## Flexibel zoeken met Cypher

Hierboven waren de voorbeelden steeds van het type: match een bepaald
patroon, en geef een of meerdere delen van de match terug als
resultaat. Met Cypher is veel meer mogelijk. Hieronder tonen we een
paar veelgebruikte technieken, zonder de ambitie een tutorial voor
Cypher te verzorgen.

### meerdere patronen

Het argument van *match* is tot nu toe steeds één patroon. Je kunt ook meerdere patronen als argument van `match` gebruiken, gescheiden door een comma. Het volgende patroon zoekt naar knopen van category `sv1` waarbij in dezelfde zin een vraagteken voorkomt:

```cypher
match (w:word{word: '?'}),
      (n:node{sentid:w.sentid,cat: 'sv1'})
return n
```

Een gerelateerde techniek is het gebruik van meerdere `match` statements. In dat geval wordt de eerste match gebruikt als een eerste filter, en kan met variabelen geïnstantieerd in het eerste match patroon gezocht worden naar het tweede match patroon. Het vorige voorbeeld zou ook als volgt kunnen worden aangepakt:

```cypher
match (w:word{word: '?'})
match (n:node{sentid:w.sentid,cat: 'sv1'})
return n
```



### optional match

Het is lastig om direct in de graafpatronen te eisen dat een bepaalde edge *niet* bestaat. Een manier om dat toch te bewerkstelligen is het gebruik van de optional match. Hierbij wordt geprobeerd een patroon te matchen, maar indien dat niet lukt slaagt de query als geheel. De eventuele variabele die gebonden had moet worden door het optionele patroon krijgt de waarde null. Dit gebruiken we in het volgende voorbeeld om te eisen dat een knoop geen subject relatie mag hebben:

```cypher
match (n:node{cat: 'sv1'})
optional match (n)-[:rel{rel: 'su'}]->(x1)
with n, x1
where x1 is null
return n
```

!!! warning "Vuistregel"
    Gebruik na `optional match` altijd een `with`, anders krijg je
    resultaten die je waarschijnlijk niet wilt.

Je kunt een optional match ook gebruiken om te zoeken naar
alternatieve structuren. Met onderstaande query zoek je naar
ja/nee-vragen. Hierin moet het resultaat van de optional match of null
zijn, of aan bepaalde voorwaarden voldoen.

```cypher
match (w:word{word: '?'}) ,
      (n:node{sentid:w.sentid,cat: 'sv1'})<-[r:rel]-()
optional match (n)-[:rel{rel: 'hd'}]->(n1)
with w, r, n, n1
where not r.rel = 'body'
  and n.end < w.end
  and (n1 is null or n1.stype is null or n1.stype = 'ynquestion')
return n.sentid, n.id
```

### exists

In sommige gevallen is het handiger om in plaats van een optional
match een `exists` of `not exists` na een `with` te gebruiken. Hierin
is `exists` een functie met als argument een path-expressie.

Dit is een voorbeeld waarin gezocht wordt naar ingebedde vraagzinnen:

```cypher
match (n:node{cat: 'whsub'})-[:rel*1..2]->(n2:node{cat: 'ssub'})
where exists( (n)-[:rel{rel: 'body'}]->(n2) )
   or exists( (n)-[:rel{rel: 'body'}]->(:node{cat: 'conj'})-[:rel{rel: 'cnj'}]->(n2) )
   or exists( (n)-[:rel{rel: 'body'}]->(:node{cat: 'du'})-[:rel]->(n2) )
return n.sentid, n.id, n2.id
```

Er gelden twee beperkingen voor exists:

 1. Je kunt geen nieuwe variabelen introduceren.
 1. Gebruik dit niet voor paden met een variabele lengte, zoals
    `()-[:rel*]->()` want daarmee komt de server in de
    [problemen](https://github.com/bitnine-oss/agensgraph/issues/524).

De functie `exists` is specifiek voor AgensGraph, en een van de dingen
waarmee AgensGraph afwijkt van standaard openCypher.
Zie [Cypher in AgensGraph](../agensgraph/).

### distinct

Bij sommige queries kan dezelfde knoop op meerdere manieren gevonden worden. Meestal geeft dat geen extra informatie. Met `distinct` kunnen zulke dubbele hits verwijderd worden.

```cypher
match (n:node{cat:'mwu'}) -[:rel{rel:'mwp'}]-> (p:word{postag:'SPEC(deeleigen)'})
return n
```

In bovenstaand voorbeeld zoek je mwu's waarvoor geldt dat één van de delen de postag *SPEC(deeleigen)* heeft. Indien een mwu nu meerdere van zulke delen heeft, levert dat ook meerdere hits op. Die dubbelen verwijder je met het keyword `distinct` als volgt:

```cypher
match (n:node{cat:'mwu'}) -[:rel{rel:'mwp'}]-> (p:word{postag:'SPEC(deeleigen)'})
return distinct n
```

Als je in de output kijkt naar de zinnen kan het nog steeds zo zijn dat een zin meerdere keren voorkomt, indien die zin meerdere mwu's bevat die aan deze eisen voldoet. Als je elke zin slechts één keer terug wilt krijgen kun je expliciet naar zinnen zoeken. Dat gaat dan als volgt:

```cypher
match (:node{cat:'mwu'}) -[:rel{rel:'mwp'}]-> (p:word{postag:'SPEC(deeleigen)'})
return distinct p.sentid
```

### with distinct

Hierboven bespraken we hoe je kunt zorgen dat je met behulp van distinct unieke resultaten kunt krijgen. Soms is de situatie ingewikkelder. Stel je voor, dat je op zoek bent naar subjecten waarvoor geldt dat (net als hierboven) het een meerwoorduitdrukking moet zijn met minstens één dochter die de postag *SPEC(deeleigen)* heeft. Je wilt dan niet hetzelfde onderwerp meerdere keren terugkrijgen omdat er meerdere dochters de postag *SPEC(deeleigen)* hebben, maar je wilt wel, eventueel, meerdere onderwerpen uit dezelfde zin terugkrijgen als die er mochten zijn. Dan werkt het niet om `dinstinct` voor het hele patroon te gebruiken. In zulke gevallen kun je `with` gebruiken:

```cypher
match (n:node{cat:'mwu'}) -[:rel{rel:'mwp'}]-> (:word{postag:'SPEC(deeleigen)'})
with distinct n
match (:node{cat:'smain'})-[:rel{rel:'su'}]->(n)
return n
```

Keyword `with` fungeert dus als een soort `return` statement, waarna je vervolgens verdere eisen aan de opgeleverde resultaten kunt stellen. Deze techniek lijkt een beetje op het gebruik van een existentiële quantifier.

Een ingewikkelder voorbeeld van deze techniek vind je bij de volgende query om cross-serial verb clusters terug te vinden. In deze definitie wordt eerst een knoop `n1` gevonden die met relatie `vc` knoop `n1` in een werkwoordcluster domineert. Die definitie refereert aan knopen `x` en `w`, maar er kunnen meerdere knopen `x` zijn. In die verschillende `x` zijn we niet geinteresseerd - het bestaan van een `x` is voldoende om vast te stellen dat het hier om een werkwoordcluster gaat. Op basis van de unieke combinatie `n` en `n1` wordt dan vervolgens de eis gesteld dat het onderwerp van de één fungeert als het lijdend voorwerp van de ander:

```cypher
match (x)<-[:rel]-(n:node{cat: 'inf'})<-[:rel{rel: 'vc'}]-(n1)-[:rel{rel: 'hd'}]->(w:word{pt: 'ww'})
where not n1.cat in ['smain','sv1']
  and x.begin < w.begin
with distinct n, n1

match (n)-[:rel{rel: 'su'}]->()<-[:rel{rel: 'obj1'}]-(n1)
return n
```

### where clause

Verdere condities aan een patroon kun je (ook) specificeren met behulp van een `where`-clause. Bijvoorbeeld, de query die naar lijdend voorwerpen van *drinken* zoekt kun je uitbreiden door ook varianten van *drinken* toe te staan:

```cypher
match (w1:word)-[:ud{main:'obj'}]->(o:word{upos:'NOUN'})
where w1.lemma in ['eten','op_eten','drinken','op_drinken']
return o
```

Zo'n where clause is ook een makkelijke manier om uit te drukken dat een bepaalde waarde nu juist niet mag voorkomen. In het volgende voorbeeld zoeken we werkwoorden met lijdend voorwerpen waarbij dat lijdend voorwerp geen voornaamwoord mag zijn:

```cypher
match (w1:word{upos:'VERB'})-[:ud{main:'obj'}]->(w2:word)
where w2.upos != 'PRON'
return w1, w2
```

### tellen en sorteren

Vaak is het interessant om te aggregeren over de relevante delen van een match. Dat is natuurlijk makkelijk in een database te doen.

```cypher
match (w:word)-[:ud{main:'obj'}]->(w2:word{upos:'NOUN'})
where w.lemma in ['eten','op_eten','drinken','op_drinken']
return w2.lemma, count(*)
```

Omdat in zulke gevallen het resultaat geen node of word is, zal in de AlpinoGraph interface de resulterende tabel worden getoond (in plaats van de gevonden zinnen met de matchende delen). De namen van de kolommen van de tabel kun je eventueel expliciet aangeven met `as Name`, zoals in dit voorbeeld:

```cypher
match (w:word)-[:ud{main:'obj'}]->(w2:word{upos:'NOUN'})
where w.lemma in ['eten','op_eten','drinken','op_drinken']
return w2.lemma as woord, count(*) as aantal
```

De namen van de kolommen zijn ook relevant indien je wilt sorteren. Je verwijst dan naar de kolom op basis waarvan je wilt sorteren.

```cypher
match (w:word)-[:ud{main:'obj'}]->(w2:word{upos:'NOUN'})
where w.lemma in ['eten','op_eten','drinken','op_drinken']
return w2.lemma as woord, count(*) as aantal
order by aantal
```

Je kunt natuurlijk ook omgekeerd sorteren, dat gaat als volgt:

```cypher
match (w:word)-[:ud{main:'obj'}]->(w2:word{upos:'NOUN'})
where w.lemma in ['eten','op_eten','drinken','op_drinken']
return w2.lemma as woord, count(*) as aantal
order by aantal desc
```

En ten slotte kun je nog op basis van het woord sorteren indien de tellingen gelijk zijn, door een volgende kolomnaam toe te voegen bij het `order` regel:

```cypher
match (w:word)-[:ud{main:'obj'}]->(w2:word{upos:'NOUN'})
where w.lemma in ['eten','op_eten','drinken','op_drinken']
return w2.lemma as woord, count(*) as aantal
order by aantal desc, woord
```

### except, union, intersect

Het is mogelijk om tabellen met resultaten te combineren met `except`, `union`, en `intersect` clauses. Een onbenullig voorbeeld ziet er zo uit:

```cypher
match (n:node{cat: 'pp'})
return n
union
match (n:node{cat: 'np'})
return n
```

Bij het gebruik van deze operatoren moeten de beide delen van de union een tabel opleveren met evenveel kolommen, en corresponderende kolommen moeten van hetzelfde type zijn. Een bij-effect van het gebruik van deze set-operatoren is, dat de tabellen ook als set worden geïnterpreteerd: elke rij is uniek. De volgende query levert daarom slechts een tabel met twee waardes: *np* en *pp*.

```cypher
match (n:node{cat: 'pp'})
return n.cat
union
match (n:node{cat: 'np'})
return n.cat
```

We kunnen bijvoorbeeld intersectie gebruiken om te ontdekken welke enkelvoudige zelfstandige naamwoorden (die niet als verkleinwoord zijn gebruikt) zowel met *de* als *het* combineren in het corpus:

```cypher
match (n:word{num:'sg'}) -[:ud{rel:'det'}]-> (:word{lemma:'de'})
return n.lemma
intersect
match (n:word{graad:'basis'}) -[:ud{rel:'det'}]-> (:word{lemma:'het'})
return n.lemma
```

De `except` clause werkt op een vergelijkbare manier en trekt de resultaten van de tweede match af van de eerste. Dus om de zelfstandige naamwoorden te vinden die alleen met "het" combineren, kun je een variant van de vorige query toepassen:

```cypher
match (n:word{graad:'basis'}) -[:ud{rel:'det'}]-> (:word{lemma:'het'})
return n.lemma
except
match (n:word{num:'sg'}) -[:ud{rel:'det'}]-> (:word{lemma:'de'})
return n.lemma
```

### all, any, none, single

Stel, je hebt een query met een path van onbekende lengte, zoals:

```cypher
match p = (n1)-[r:rel*1..3]->(n2)
return p
```

Hieraan wil je de voorwaarde toevoegen dat alle edges in `r` geen
attribuut `id` hebben. Dit kun je doen met de functie `all`.
Maar dit werkt niet:

```cypher
match p = (n1)-[r:rel*1..3]->(n2)
where all(x in r where x.id is null)
return p
```

De juiste syntax
is ingewikkeld omdat je het argument eerst in jsonb moet omzetten,
waarna de attributen beschikbaar zijn via properties:

```cypher
match p = (n1)-[r:rel*1..3]->(n2)
where all(x in to_jsonb(r) where x.properties.id is null)
return p
```

Zie ook:
[Predicates functions](https://bitnine.net/documentations/manual/agens_graph_developer_manual_en.html#predicates-functions)
in de
[AgensGraph Developer Manual](https://bitnine.net/documentations/manual/agens_graph_developer_manual_en.html).
(TODO: is deze link stabiel?)

Bovenstaand voorbeeld is slechts bedoeld als illustratie. Voor dit geval, als een `:rel` geen
`id` heeft, dan heeft het `primary:true`, dus bovenstaande query kan
sneller op deze manier:

```cypher
match p = (n1)-[r:rel*1..3{primary:true}]->(n2)
return p
```

### select

Alhoewel je met Cypher een eind kunt komen, loop je soms tegen
beperkingen aan die in Cypher niet op te lossen zijn, maar wel in SQL.
Omdat AgensGraph geïmplementeerd is in PostgreSQL kun je Cypher
imbedden in SQL (en andersom).

Het algemene patroon is dit:

```cypher
select ...
from (
   match ...
   return ...
) as foo
where ...
```

In het volgende voorbeeld worden nevenschikkingen van NP's geteld:

```cypher
select count(distinct(sentid)) as zinnen, count(*) as items
from (
  match (n:node{cat: 'conj'}) -[:rel{rel: 'cnj'}]-> (:nw{_np: true})
  return n.sentid, n.id
  except
  match (n:node{cat: 'conj'}) -[:rel{rel: 'cnj'}]-> (n2:nw)
  where n2._np is null
  return n.sentid, n.id
) as foo
```

## Resultaten van een query in AlpinoGraph

In het algemeen levert elke query een tabel op. Maar AlpinoGraph behandelt niet elke tabel op dezelfde wijze. In de gevallen waarbij de query een knoop in de graaf of een deelgraaf oplevert, kiest AlpinoGraph ervoor om de zin die hoort bij die knoop of deelgraaf als resultaat te presenteren. Als je vervolgens op een zin klikt krijg je de visualizatie van de graaf te zien, in meerdere varianten.

Indien gewenst kun je alsnog de resultaten in tabelvorm bekijken door op de betreffende button te klikken. Twee andere opties zijn "woorden" en "lemma's". In die laatste gevallen wordt per hit de woordgroep (in woorden dan wel in lemma's) opgehaald die door de matchende knoop wordt gedomineerd. Van al die woordgroepen wordt vervolgens een frequentieoverzicht gemaakt.

Dus voor deze query:

```cypher
match (w:word{lemma:'zijn'})
return w.word
```

krijg je alleen een tabel te zien, terwijl je voor de volgende query de zinnen terug krijgt:


```cypher
match (w:word{lemma:'zijn'})
return w
```

Je kunt hier dus nu kiezen voor de andere opties. Als je nu voor "tabel" kiest, krijg je erg veel informatie: alle attributen van elke matchende knoop. Kies je voor "woorden", dan krijg je een mooi frequentie-overzicht van alle vormen van het werkwoord 'zijn'. In dit specifieke geval is het niet erg inzichtelijk om te kiezen voor "lemma's".

Het volgende voorbeeld suggereert wanneer het nuttig kan zijn om voor "lemma's" te kiezen:

```cypher
match (:word{lemma:'eten'})-[:ud{main:'obj'}]->(w2:word)
return w2
```

Dit levert dus alle zinnen op waarin *eten* een lijdend voorwerp heeft. En als je op "lemma's" klikt krijg je een mooi frequentieoverzicht van al de (hoofden van) die lijdend voorwerpen.

In bovenstaande gevallen zijn de matchende knopen steeds woorden, maar dit werkt dus op vergelijkbare wijze voor hogere knopen. De volgende query vindt alle hogere knopen die in relatie *svp* met een werkwoord staan:

```cypher
match (:node)-[:rel{rel:'svp'}]->(w2:node)
return w2
```

Omdat de query een node oplevert krijg je per default de zinnen te zien waarin een match voorkomt. Als je kiest voor "lemma's" levert dat dus een frequentieoverzicht op van de betreffende woordgroepen.

In sommige gevallen bevatten die woordgroepen gaten (discontinue woordgroepen). Een typisch voorbeeld hiervan levert de volgende query:

```cypher
match (:node)-[:rel{rel:'pc'}]->(w2:node)
return w2
```

Je ziet dat dan in het frequentieoverzicht van de woorden of de lemma's met *[...]* de gaten in de woordgroepen worden aangegeven.

In het receptenboek wordt uitgelegd hoe je queries kunt formuleren die
op vergelijkbare wijze aggregeren over de waardes van attributen
indien het andere attributen betreft dan `word` of `lemma`.
Maar als je de resultaten wilt downloaden heb je ook de mogelijkheid
voor een ander attribuut dan `word` of `lemma` te kiezen.

## Geavanceerd zoeken met CYPHER

### paden met `*`

Een sequentie van edges (een pad) kan soms compact worden genoteerd met behulp van de `*`-operator. Om te zoeken naar een conjunct binnen een conjunct binnen een conjunct kun je formuleren:

```cypher
match (n)-[:rel{rel: 'cnj'}]->()-[:rel{rel: 'cnj'}]->()-[:rel{rel: 'cnj'}]->()
return n
```

Je kunt zo'n sequentie afkorten als volgt:
```cypher
match (n)-[:rel*3{rel: 'cnj'}]->()
return n
```

Niet alleen kun je in zo'n patroon het preciese aantal stappen aangeven dat vereist is, je kunt ook een interval specificeren `i..j`, waarbij `i` en `j` integers zijn. `*3..8` geeft dan een pad aan van tussen de 3 en de 8 edges. Je kunt `i` of `j` ook weglaten. Met `*..5` geef je een pad aan van hoogstens 5 edges, terwijl `*3..` alle paden aangeeft met minstens drie edges.

In het volgende voorbeeld zoeken we een knoop met daarin een woord dat je via minstens drie conjunct-relaties kunt bereiken:

```cypher
match (n)-[:rel*3..{rel: 'cnj'}]->(:word)
return n
```

Een typisch geval is het gebruik van dit mechanisme met het interval `*0..1`. In het volgende voorbeeld worden knopen geïdentificeerd die als subject optreden. In geval de knoop een lexicaal hoofd heeft, wordt dat hoofd als resultaat gevonden. Als de knoop zelf lexicaal is (en dus ook geen hoofd kan hebben), wordt de knoop zelf teruggegeven. Subjecten die niet lexicaal zijn, maar geen hoofd hebben (conjuncties bijvoorbeeld) worden dus overgeslagen.

```cypher
match ()-[:rel{rel: 'su'}]->()-[:rel*0..1{rel: 'hd'}]->(n:word)
return n
```

### Eisen aan aantal

Soms is er behoefte om te kunnen zeggen dat een bepaalde knoop of bepaalde relatie een specifiek aantal keer aanwezig moet zijn. In het volgende voorbeeld willen we nevenschikking vinden waarbij er precies één coordinator is:

```cypher
match (n:node{cat: 'conj'})-[r:rel{rel: 'crd'}]->()
with n, count(*) as cnt
where cnt = 1
return n
```

Je kunt dus iets vergelijkbaars doen om een nevenschikking te vinden met precies 5 conjuncten:
```cypher
match (n:node{cat: 'conj'})-[:rel{rel: 'cnj'}]->(r)
with n, count(*) as cnt
where cnt = 5
return n
```

## Berekende attributen

Sommige complexere concepten zijn "voorberekend" en als
[hulpattribuut](../attributen) aanwezig. Hierdoor wordt het zoeken
naar sommige ingewikkelde dingen een stuk vereenvoudigd.

Een voorbeeld is het zoeken naar een NP-topic dat het onderwerp is.
Hierin speelt het zinsdeel dat met *vorfeld* aangeduid wordt een rol.
Omdat dit iets is dat als voorberekend attribuut is opgenomen
wordt de query vrij eenvoudig, en vooral veel sneller dan wanneer je
de structuur van een vorfeld zou moeten uitschrijven in je query:

```cypher
match (n:nw{_vorfeld: true, _np: true})<-[r:rel]-()
where r.rel in ['su','sup'] and r.id is null
return n.sentid, n.id
```

De definitie van het attribuut `_vorfeld` zie je [hier](../attributen/#_vorfeld).
