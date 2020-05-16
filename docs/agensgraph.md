# Cypher in AgensGraph

AgensGraph is gebouwd op PostgreSQL. Hierdoor kun je Cypher combineren
met SQL. Maar door de eigenaardigheden van SQL (zoals geïmplementeerd
in PostgreSQL) wijkt de syntax voor Cypher op een paar punten af van
de standaard [openCypher](https://www.opencypher.org/).

Hoofdletters in labels spelen geen rol. Alle hoofdletters worden omgezet naar
kleine letters, tenzij het label tussen **dubbele**
aanhalingstekens staat.

Strings dienen tussen **enkele** aanhalingstekens te staan.

Dit is correct in openCypher:
```text
MATCH (tobias {name: 'Tobias'}), (others)
WHERE others.name IN ['Andres', 'Peter'] AND (tobias)<-[]-(others)
RETURN others.name, others.age;
```
In AgensGraph moet je dat zo doen:
```text
MATCH (tobias {name: 'Tobias'}), (others)
WHERE others.name IN ['Andres', 'Peter'] AND EXISTS((tobias)<-[]-(others))
RETURN others.name, others.age;
```

openCypher:
```text
MATCH (persons), (peter {name: 'Peter'})
WHERE NOT (persons)-[]->(peter)
RETURN persons.name, persons.age;
```
AgensGraph:
```text
MATCH (persons), (peter {name:'Peter'})
WHERE NOT EXISTS((persons)-[]->(peter))
RETURN persons.name, persons.age;
```

Andere verschillen...

Voor niet nader gespecifieerde relatie mag dit in openCypher:

```text
()-->()
()<--()
```

In AgensGraph moet je hiervoor deze notatie gebruiken:

```text
()-[]->()
()<-[]-()
```

Relaties kunnen maar van één soort zijn. Wel kun je soorten
definiëren met een of meer basissoorten. Bijvoorbeeld, als je
*vader* en *moeder* definieert als afgeleid van *ouder*, dan kun je
door te zoeken naar *ouder* zowel relaties van *vader* en *moeder*
vinden. Direct zoeken op meer dan één relatie kan niet. Dit werkt
niet in AgensGraph: `match ()-[:vader|moeder]->()`

Dit werkt in Neo4j:

```text
match (x) where "Node" in labels(x) or "Word" in labels(x) return x;
```

In AgensGraph moet je het zo doen:

```text
match (x) where any(i in labels(x) where i='node' or i='word') return x;
```

Als je alle variabelen wilt hebben, dan werkt dit ook:
```text
match (w1:word{lemma:'fietsen'})<-[n:next*]-(w2:word{lemma:'gaan'}) return *;
```

Let op:
```text
# drop graph if exists test cascade;
# create graph test;
# set graph_path='test';
# create (:foo);
# match (n:foo) where not (n.bar='test') return n;
 n
---
(0 rows)

# match (n:foo) where not (n.bar is not null and n.bar='test') return n;
     n
------------
 foo[3.1]{}
(1 row)

# drop graph test cascade;
```
