
[ Ga naar AlpinoGraph](https://urd2.let.rug.nl/~kleiweg/alpinograph/){: .md-button }

test 1, 2, 3

# Inleiding

AlpinoGraph is een tool om syntactisch geannoteerde corpora te doorzoeken. De tool maakt gebruik van [AgensGraph](https://bitnine.net/agensgraph/). AgensGraph combineert databasetechnologie ([PostgreSQL](https://www.postgresql.org/)) en [Cypher](https://en.wikipedia.org/wiki/Cypher_(query_language)), de standaard zoektaal voor grafen. De zoek-queries die je in AlpinoGraph kunt gebruiken zijn daarom een mix van SQL en Cypher. Daar voegt AlpinoGraph nog enkele extra uitbreidingen aan toe, zoals een eenvoudig maar handig systeem van macro's, en visualisatie van de resultaten.

!!! note
    Als je Docker hebt kun je AlpinoGraph lokaal draaien, zodat je zelf
    corpora toe kunt voegen.
    Zie [AlpinoGraph in Docker](https://github.com/rug-compling/alpinograph-docker).

## Overzicht van documentatie

In [Overzicht van AlpinoGraph](interface/) leer je de interface
kennen.

In [Zoeken met AlpinoGraph](zoeken/) vind je een uitgebreide
inleiding in de zoektaal Cypher en hoe je die in AlpinoGraph kunt
toepassen.

In het [Receptenboek](recepten/) vind je een verzameling
voorbeelden van verschillende soorten zoekopdrachten.

In [AlpinoGraph in AgensGraph](alpinoagens/) is gedocumenteerd hoe
het XML-formaat van Alpino is vertaald naar AgensGraph.

In [Hulpattributen](attributen/) vind je informatie over de
attributen die niet voorkomen in Alpino, maar door AlpinoGraph zijn
toegevoegd om het zoeken krachtiger te maken.

In [Cypher in AgensGraph](agensgraph/) lees je hoe Cypher zoals dat
in AgensGraph is geïmplementeerd afwijkt van de standaard openCypher.

In [Corpora](corpora/) vind je informatie over alle corpora die
beschikbaar zijn in de officiële versie van AlpinoGraph.

In [Links](links/) vind je nuttige externe informatie.
