# De interface van AlpinoGraph

De werking van de interface van AlpinoGraph spreekt grotendeels voor
zich. Maar er zijn een aantal dingen waar je rekening mee moet houden.

## Zinnen

Wanneer een query als resultaat een kolom met het label `sentid`
geeft, of een object met een property met de naam `sentid`, dan gaat
AlpinoGraph ervan uit dat het resultaat verwijst naar een zin met de
betreffende sentence-ID. Zijn er dan ook nog een of meer kolommen of
properties met de naam `id`, dan wordt aangenomen dat die verwijzen
naar nodes of woorden in de zin.

Met een zin als resultaat toont AlpinoGraph niet alleen een tabel,
maar ook de zin, met een link naar de grafische weergave van die zin,
en andere informatie zoals lijsten met woord-varianten en
lemma-varianten.

Dit alles raakt in de war als je een query als de volgende uitvoert:

```cypher
match (n1:word{lemma:'fiets'}),
      (n2:word{lemma:'trein'})
return n1, n2
```

In de resultaten zijn `n1` en `n2` meestal niet uit dezelfde zin. Je
krijgt dus twee verschillende sentence-IDs, waarvan AlpinoGraph er
eentje negeert.  En je krijgt twee IDs, waarvan eentje uit de
"verkeerde" zin.

Het gevolg: je krijgt dingen te zien die niet kloppen.

Om dit te voorkomen moet je ervoor zorgen, als je meerdere velden met
zins-informatie krijgt, dat dit van dezelfde zin komt.

Bovenstaand voorbeeld kun je zo verbeteren:

```cypher
match (n1:word{lemma:'fiets'}),
      (n2:word{lemma:'trein', sendtid: n1.sentid})
return n1, n2
```


## Macro's

In AlpinoGraph kun je simpele macro's gebruiken, een label in een
query die vervangen wordt door een voorgedefinieerde tekst. Er zijn
geen macro's met argumenten.

Macro's kun je definieren door op knop ^^Macro's^^ te klikken.
Definities van je macro's worden opgeslagen in je browser.

Een macro-definitie ziet er zo uit:

```python
kleur = """
    'rood','groen','blauw'
"""
```

Je kunt macro's in een query gebruiken door het label tussen
procent-tekens te zetten:

```cypher
match (w:word)
where w.lemma in [%kleur%]
return w
```
In de definite van macro's zelf kun je ook andere macro's gebruiken.

## Permalink

Door op de knop ^^Permalink^^ te klikken kun je een URL kopiëren
waarin de huidige query is opgenomen. Die URL kun je bijvoorbeeld in
een blog gebruiker. Wanneer iemand op die link klikt gaat de browser
naar AlpinoGraph, en wordt de betreffende query ingevoerd in het
zoekvak. Ook de keus voor het corpus wordt overgenomen.

Wil je dat de query direct wordt uitgevoerd als iemand op de link klikt,
voeg dan de tekst `,1` (komma één) aan de URL toe.

## Pagina's

Wanneer een query meer resultaten geeft dan er op een "pagina" passen,
dan staan er onderaan knoppen waarmee je naar een volgende pagina
kunt.

Dit werkt niet altijd zoals je zou verwachten. Tenzij je expliciet in
je query opgeeft dat de resultaten gesorteerd moeten worden, staat de
volgorde van de resultaten niet vast.

Wanneer je naar de tweede pagina gaat, dan voert de server dezelfde
query nog een keer uit, maar in de uitvoer slaat ie eerst een aantal
resultaten over, zoveel als er op de eerste pagina stonden. Maar die
overgeslagen resltaten zijn niet altijd dezelfde als die je net hebt
gezien. Door onder andere caching kan de server bij een tweede run de
resultaten in een andere volgorde genereren.

Vaak gaat het goed, en anders is het vaak geen probleem. Als je echt
alle resultaten langs wilt gaan, dan kun je aan de query een `order
by` toevoegen. Het nadeel hiervan is dat de server eerst alle hits
moet verzamelen en sorteren, voordat je een resultaat te zien krijgt.
Zonder sorteren zie je elke hit zodra die is gevonden.

Behalve `order by` zijn er andere instructies die ervoor zorgen dat de
resultaten gesorteerd worden, zoals bijvoorbeeld `count()`, `union`, en elke
functie die over de complete set van resultaten werkt.

## Ontdubbelen

Wanneer je onderstaande query
[uitvoert](https://urd2.let.rug.nl/~kleiweg/alpinograph/#match%20%28n%3Anode%7Bcat%3A'conj'%7D%29-%5B%3Arel*%5D-%3E%28%3Aword%7Blemma%3A'hij'%7D%29%0Areturn%20n%0Aorder%20by%20n.sentid,alpinotreebank,1)
op het corpus Alpino Treebank...

```cypher
match (n:node{cat:'conj'})-[:rel*]->(:word{lemma:'hij'})
return n
order by n.sentid
```

... dan krijg iets te zien dat zo begint:

```text
 1. Hij liep een shock op en kneusde enkele ribben .
 3. Hij lichtte de bal over Van den Lugt heen en gaf een diagonale
    voorzet naar Hilmstrom .
 5. Hij vertelt en windt zich op en moet door zijn vrouw tot kalmte
    worden gebracht .
10. Hij is geweken voor bedreiging en heeft zich onvoldoende achter
    de rector geplaatst en diens gezag onvoldoende hooggehouden .
```

In de nummering ontbreken heel wat getallen. Dat is omdat identieke
resultaten weggelaten worden. Dat weglaten gebeurt per pagina. Ga je
naar de tweede pagina dan zie je op nummer 41 dezelfde zin als die op
nummer 40 stond.

Nu [deze variant](https://urd2.let.rug.nl/~kleiweg/alpinograph/#match%20p%20%3D%20%28n%3Anode%7Bcat%3A'conj'%7D%29-%5B%3Arel*%5D-%3E%28%3Aword%7Blemma%3A'hij'%7D%29%0Areturn%20n%2C%20p%0Aorder%20by%20n.sentid%0Alimit%2010,alpinotreebank,1):

```cypher
match p = (n:node{cat:'conj'})-[:rel*]->(:word{lemma:'hij'})
return n, p
order by n.sentid
limit 10
```

Nu zie je wel tien resultaten, met identieke zinnen. Maar dat is omdat
de resultaten wel identiek lijken, maar het niet zijn. De eerdere
query gaf als resultaat steeds een node, vaak dezelfde omdat die node
op verschillende manieren kon worden gevonden. De laatste query geeft
ook het complete patroon terug, en dat is niet altijd gelijk. Dat kun
je zien door op de verschillende (zelfde) zinnen klikt.

Klik vervolgens op de knop ^^woorden^^ of ^^lemma's^^, en nu zie je
dat de vier verschillende zinnen elk maar één keer is geteld. Dat is
omdat nu alleen gekeken wordt naar de set van woorden per zin, en
identieke sets per zin worden weer weggefilterd.
