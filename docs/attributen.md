# Hulpattributen

In AlpinoGraph worden extra attributen gebruikt die niet in Alpino
voorkomen. Deze attributen zijn er om sommige query's te
vereenvoudigen en het zoeken te versnellen.

## Items

Hieronder een lijst van extra attributen op items van de types `(:node)`
en `(:word)`.

### `_clause`

!!! info "Is deze node een *clause*?"
    Items: `(:node)` <br>
    Type: bool <br>
    Waarde: `true` of niet aanwezig

De waarde is `true` voor een node waarvan `cat` de waarde `smain`,
`sv1` of `ssub` heeft.

^^Definitie^^

```text
match (n:node)
where n.cat in ['smain','sv1','ssub']
set n._clause = true;
```

### `_clause_lvl`

!!! info "Als deze node een *clause* is, hoe diep is het dan genest?"
    Items: `(:node)` <br>
    Type: int <br>
    Waarde: 1 of hoger, of niet aanwezig

Dit attribuut is aanwezig op nodes met de waarde `true` voor het
attribuut `_clause`. Het geeft aan hoe diep de clause-node zit genest
onder andere clause-nodes.

Het hoogste niveau heeft de waarde 1.

De bepaling van niveau wordt afgeleid via [primaire relaties](#primary).

^^Definitie^^

```text
match p = (:sentence)-[:rel*{primary:true}]->(n:node)
where n.cat in ['smain','sv1','ssub']
set n._clause_lvl = count_lvl(to_json(p).vertices);
```

Hierbij is `count_lvl` een hulpfunctie met deze definitie:

```text
CREATE FUNCTION count_lvl(vertices json) RETURNS integer AS $$
DECLARE
  count integer := 0;
  cat text;
BEGIN
  FOR i IN 0..(json_array_length(vertices) - 1)
  LOOP
    cat := vertices -> i #>> '{properties, cat}';
    IF cat = 'smain' OR cat = 'sv1' OR cat = 'ssub' THEN
      count := count + 1;
    END IF;
  END LOOP;
  RETURN count;
END;
$$ LANGUAGE plpgsql;
```


### `_deste`

!!! info "Een hulpattribuut voor het zoeken naar [correlatieve comparatieven](https://urd2.let.rug.nl/~kleiweg/alpinograph/#--%20SPOD%20%7C%20Correlatieve%20comparatieven%0A%0Amatch%20p1%20%3D%20%28n1%3Anode%7B_deste%3A%20true%7D%29%3C-%5B%3Arel*0..%5D-%28n0%3Anode%29%3C-%5B%3Arel*0..%5D-%28n%3Anode%7Bcat%3A%20'du'%7D%29%0Amatch%20p2%20%3D%20%28n1%29%3C-%5B%3Arel*0..%5D-%28n0%29-%5B%3Arel*0..%5D-%3E%28n2%3Anode%7B_deste%3A%20true%7D%29%0Aoptional%20match%20p%20%3D%20%28n0%29%3C-%5B%3Arel*0..%5D-%28%3Anode%7Bcat%3A'du'%7D%29%3C-%5B%3Arel*1..%5D-%28n%29%0Awith%20n1%2C%20n2%2C%20p%2C%20p1%2C%20p2%0Awhere%20n1.id%20%3C%20n2.id%0A%20%20and%20p%20is%20null%0Areturn%20p1%2C%20p2%0A%0A)"
    Items: `(:node)` <br>
    Type: bool <br>
    Waarde: `true` of niet aanwezig

TODO: korte uitleg

^^Definitie^^

```text
match (:word{graad:'comp'})<-[:rel{primary:true}]-(n:node)
                          -[:rel{primary:true}]->(n2:nw)
optional match p = (:word{lemma:'des'})<-[:rel{primary:true}]-(n2)
                      -[:rel{primary:true}]->(:word{lemma:'te'})
with n, n2, p
where n2.lemma in ['hoe','deste']
   or p is not null
set n._deste = true;
```


### _n_words

!!! info "Het aantal woorden dat deze node bestrijkt"
    Items: `(:node)`, `(:word)` <br>
    Type: int <br>
    Waarde: 1 of groter

Voor woorden is dit altijd 1.

Voor nodes is dit het aantal woorden onder de node die zowel via
[primaire](#primary) als via niet-primaire relaties kan worden bereikt.

^^Definitie^^

```text
match (n:nw)-[:rel*0..]->(w:word)
with distinct n.sentid as sentid, n.id as id, w.id as wid
with sentid, id, count(wid) as c
match (n2:nw{sentid:sentid,id:id})
set n2._n_words = c;
```

### `_np`

!!! info "Is dit een NP?"
    Items: `(:node)`, `(:word)` <br>
    Type: bool <br>
    Waarde: `true` of niet aanwezig

Het attribuut `_np` is `true` voor nodes en woorden die overeenkomen met deze xpath-expressie:

TODO: een conj van een conj (etc) van een np is ook een np

TODO: in dit geval **nadat** indexnodes worden geëxpandeerd (klopt dat?)

TODO: definitie hieronder updaten

```xpath
//node[( ( @cat="np"
           or
           ( @lcat="np"
             and
             not(@rel=("hd","mwp"))
           )
           or
           ( @pt="n"
             and not(@rel="hd")
           )
           or
           ( @pt="vnw"
             and
             @pdtype="pron"
             and
             not(@rel="hd")
           )
           or
           ( @cat="mwu"
             and
             not(@rel="hd")
             and
             @rel=("su","obj1","obj2","app")
           )
         )
         or
         ( @cat="conj"
           and node[( @cat="np"
                      or
                      ( @lcat="np"
                        and
                        not(@rel=("hd","mwp"))
                      )
                      or
                      ( @pt="n"
                        and not(@rel="hd")
                      )
                      or
                      ( @pt="vnw"
                        and
                        @pdtype="pron"
                        and
                        not(@rel="hd")
                      )
                      or
                      ( @cat="mwu"
                        and
                        not(@rel="hd")
                        and
                        @rel=("su","obj1","obj2","app")
                      )
                    )]
         )
       )]
```


### _vorfeld

!!! info "Is dit een vorfeld?"
    Items: `(:node)`, `(:word)`  <br>
    Type: bool <br>
    Waarde: `true` of niet aanwezig

Het ^^vorfeld^^ is het zinsdeel vóór het finiete werkwoord in een
zin met de *verb second*-volgorde. TODO: klopt deze omschrijving?

TODO: schematische weergave (plaatje)?

^^Definitie^^

```
match (x:nw)
where x.sentid + ' ' + x.id in (
    select id
    from (

        match (n:node{cat:'smain'})-[:rel{rel:'hd'}]->(fin:word)
        match (n)-[:rel*{primary:true}]->(topic:nw)-[rel:rel*0..1]->(htopic:nw)
        where (( not htopic.lemma is null)
                  and htopic.begin < fin.begin
                  and   (  length(rel) = 0
                        or rel[0].rel in ['hd','cmp','crd']
                        )
               ) or
               (  topic.begin < fin.begin
                  and
                  topic.end <= fin.begin
              )
        return topic.sentid + ' ' + topic.id as id, n.id as nid

        except

        match (n:node{cat:'smain'})-[:rel{rel:'hd'}]->(fin:word)
        match (n)-[:rel*{primary:true}]->(topic:nw)-[rel:rel*0..1]->(htopic:nw)
        where (( not htopic.lemma is null)
                  and htopic.begin < fin.begin
                  and   (  length(rel) = 0
                        or rel[0].rel in ['hd','cmp','crd']
                        )
               ) or
               (  topic.begin < fin.begin
                  and
                  topic.end <= fin.begin
              )
        match (topic)<-[:rel*1..]-(nt:node)<-[:rel*]-(n)
        match (nt)-[relt:rel*0..1]->(hnt:nw)
        where (( not hnt.lemma is null)
                  and hnt.begin < fin.begin
                  and   (  length(relt) = 0
                        or relt[0].rel in ['hd','cmp','crd']
                        )
               ) or
               (  nt.begin < fin.begin
                  and
                  nt.end <= fin.begin
              )
        return topic.sentid + ' ' + topic.id as id, n.id as nid

    ) as foo
)
set x._vorfeld = true
```

## Relaties

Extra attributen op relaties van het type `rel`.


### `id`

!!! info "Het originele ID van niet-primaire relaties"
    Relaties: `[:rel]` <br>
    Type: int <br>
    Waarde: 1 of groter, of niet aanwezig

Het ID van de lege index-node waarnaar deze relatie verwees in de
originele boom in Alpino. Dit ID wordt gebruikt om uit de graaf die
boom te reconstrueren.

### `primary`

!!! info "Is dit een primaire relatie?"
    Relaties: `[:rel]` <br>
    Type: bool <br>
    Waarde: `true`, `false`

Relaties in de originele boom in Alpino naar een lege index-node zijn
niet primair. Alle overige relaties zijn primair.
