# Inleiding

AlpinoGraph is een tool om syntactisch geannoteerde corpora te doorzoeken. De tool maakt gebruik van AgensGraph. AgensGraph combineert database technologie (PostgresSQL) en Cypher, de standaard zoektaal voor grafen. De zoek-queries die je in AlpinoGraph kunt gebruiken zijn daarom een mix van SQL en Cypher. Daar voegt AlpinoGraph nog enkele extra uitbreidingen aan toe, zoals een eenvoudig maar handig systeem van macro's.

De combinatie van databasetechnologie en graaf-gebaseerde zoektechnologie zorgt voor een erg flexibele en relatief efficiënte zoekmachine. Deze flexibiliteit betekent bijvoorbeeld dat je patronen kunt definiëren om zinnen die aan het patroon te voldoen terug te vinden. Maar je kunt ook woorden of woordgroepen terugvinden en aggregeren over de informatie van die woorden of woordgroepen (bijvoorbeeld voor het maken van frequentieoverzichten) door middel van de database-primitieven van SQL. 

Niet alleen is de zoekmachne veel flexibeler dan bijvoorbeeld XPath (zoals beschikbaar in PaQu), ook is het relatief makkelijk om verschillende annotatielagen te combineren. In AlpinoGraph zijn drie van zulke lagen beschikbaar:

- de CGN/Lassy/Alpino dependentiestructuren
- de hieruit automatisch afgeleide Universal Dependency structuren (zowel de standaard als de "enhanced" variant)
- de woord-paren relaties, zoals die in de eenvoudige zoektab in PaQu beschikbaar zijn

Deze drie lagen kunnen in queries indien gewenst eenvoudig gecombineerd worden.

In AlpinoGraph zijn een aantal syntactisch geannoteerde corpora beschikbaar. AlpinoGraph ondersteunt handmatig geverifieerde treebanks zoals CGN, Lassy Klein en de Alpino Treebank. Daarnaast worden ook automatisch door Alpino geanalyseerde corpora ondersteund, zoals Lassy Groot.

