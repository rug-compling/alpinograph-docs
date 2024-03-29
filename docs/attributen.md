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

```cypher
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

```cypher
match p = (:node{id: 0})-[:rel*0..{primary:true}]->(n:node)
where n.cat in ['smain','sv1','ssub']
with n, (select count(*)
    from jsonb_array_elements((to_json(p) -> 'vertices')::jsonb) as v
    where v -> 'properties' ->> 'cat' in ('smain','sv1','ssub')
) as c
set n._clause_lvl = c;
```

### `_cp`

!!! info "Compound parts"
    Items: `(:word)` <br>
    Type: lijst van strings <br>
    Waarde: lijst van lengte 1 of langer

Dit zijn de *compound parts* van een woord, afgeleid van het lemma.

Zie voorbeelden van [zoeken met compound parts](../recepten/#compound-parts).

^^Definitie^^

```cypher
match (w:word)
with w, regexp_split_to_array(w.lemma, '_') as cp
set w._cp = cp;
```

### `_deste`

!!! info "Een hulpattribuut voor het zoeken naar [correlatieve comparatieven](https://urd2.let.rug.nl/~kleiweg/alpinograph/#--%20SPOD%20%7C%20Correlatieve%20comparatieven%0A%0Amatch%20p1%20%3D%20%28n1%3Anode%7B_deste%3A%20true%7D%29%3C-%5B%3Arel*0..%5D-%28n0%3Anode%29%3C-%5B%3Arel*0..%5D-%28n%3Anode%7Bcat%3A%20'du'%7D%29%0Amatch%20p2%20%3D%20%28n1%29%3C-%5B%3Arel*0..%5D-%28n0%29-%5B%3Arel*0..%5D-%3E%28n2%3Anode%7B_deste%3A%20true%7D%29%0Aoptional%20match%20p%20%3D%20%28n0%29%3C-%5B%3Arel*0..%5D-%28%3Anode%7Bcat%3A'du'%7D%29%3C-%5B%3Arel*1..%5D-%28n%29%0Awith%20n1%2C%20n2%2C%20p%2C%20p1%2C%20p2%0Awhere%20n1.id%20%3C%20n2.id%0A%20%20and%20p%20is%20null%0Areturn%20p1%2C%20p2%0A%0A)"
    Items: `(:node)` <br>
    Type: bool <br>
    Waarde: `true` of niet aanwezig

Zonder dit hulpattribuut is het zoeken naar correlatieve comparatieven
zeer tijdrovend.

^^Definitie^^

```cypher
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

```cypher
match (n:nw)-[:rel*0..]->(w:word)
with distinct n.sentid as sentid, n.id as id, w.id as wid
with sentid, id, count(*) as c
match (n2:nw{sentid:sentid,id:id})
set n2._n_words = c;
```

### _nachfeld

!!! info "Is dit een nachfeld?"
    Items: `(:node)`, `(:word)`  <br>
    Type: bool <br>
    Waarde: `true` of niet aanwezig

Voor toelichting, zie beneden bij [_vorfeld](#_vorfeld)

^^Algoritme^^

```cypher
(n1)<-[:rel{rel:'hd'}]-(vp)-[:rel*0..]->()-[r:rel]->(n)
```

n is nachfeld, mits...

voorwaarden:

 * `vp.cat in ['inf','ti','ssub','oti','ppart']`
 * `not r.rel in ['hd','svp']`
 * `n.cat is null or not n.cat in ['inf','ppart']`

met:

```cypher
optional match (n)-[r2:rel]->(n2)
where r2.rel in ['hd','cmp','mwp','crd','rhd','whd','nucl','dp']
```

voorwaarden:

 * als n2 bestaat dan `n1.begin < n2.begin`
 * als n2 niet bestaat dan `n1.begin < n.begin`

meer voorwaarden:

 * geen andere vp tussen vp en n
 * geen nachfeld tussen vp en n

^^Definitie^^

De definitie maakt gebruik van een tijdelijk hulpattribuut op relaties:

```cypher
match ()-[r:rel]->()
where r.rel in ['hd','cmp','mwp','crd','rhd','whd','nucl','dp']
set r._head_rel = true;

match (n1)<-[:rel{_head_rel:true}]-()-[r2:rel{_head_rel:true}]->(n2)
where n1.begin < n2.begin
set r2._head_rel = NULL;
```

Definitie met behulp van tijdelijk attribuut:

```cypher
match (x:nw)
where x.sentid + ' ' + x.id in (
  select sid
  from (

    match (vp)
    where vp.cat in ['inf','ti','ssub','oti','ppart']
    match (n1)<-[:rel{rel:'hd'}]-(vp)-[:rel*0..]->()-[r:rel]->(n)
    where not r.rel in ['hd','svp']
      and ( n.cat is null or not n.cat in ['inf','ppart'] )
    optional match (n)-[:rel{_head_rel:true}]->(n2)
    with n, n1, n2, vp
    where n2 is not null and n1.begin < n2.begin
       or n2 is     null and n1.begin < n.begin
    return n.sentid + ' ' + n.id as sid, vp.id as vpid

    except

    match (vp)
    where vp.cat in ['inf','ti','ssub','oti','ppart']
    match (vp)-[:rel*1..]->(vp2)-[:rel*1..]->(n)
    where vp2.cat in ['inf','ti','ssub','oti','ppart']
    return n.sentid + ' ' + n.id as sid, vp.id as vpid

    except

    match (vp)
    where vp.cat in ['inf','ti','ssub','oti','ppart']
    match (n1)<-[:rel{rel:'hd'}]-(vp)-[:rel*0..]->()-[r:rel]->(n)
    where not r.rel in ['hd','svp']
      and ( n.cat is null or not n.cat in ['inf','ppart'] )
    optional match (n)-[:rel{_head_rel:true}]->(n2)
    with n, n1, n2, vp
    where n2 is not null and n1.begin < n2.begin
       or n2 is     null and n1.begin < n.begin
    match (n)-[:rel*1..]->(nn)
    return nn.sentid + ' ' + nn.id as sid, vp.id as vpid

  ) as foo
)
set x._nachfeld = true;
```

### `_np`

!!! info "Is dit een NP?"
    Items: `(:node)`, `(:word)` <br>
    Type: bool <br>
    Waarde: `true` of niet aanwezig

In Alpino heeft een NP lang niet altijd de category NP. Dit
hulpattribuut is aanwezig op elke NP, zowel node als word.


^^Definitie^^

```cypher
match ()-[r:rel]->(n1:nw)
where n1.cat = 'np'
   or ( n1.lcat = 'np' and r.rel != 'hd' and r.rel != 'mwp' )
   or ( n1.pt = 'n' and r.rel != 'hd' )
   or ( n1.pt = 'vnw' and n1.pdtype = 'pron' and r.rel != 'hd' )
   or ( n1.cat = 'mwu' and r.rel in ['su','obj1','obj2','app'] )
with n1
match (n1)<-[:rel*0..{rel:'cnj'}]-(n:nw)
set n._np = true;
```

### _vorfeld

!!! info "Is dit een vorfeld?"
    Items: `(:node)`, `(:word)`  <br>
    Type: bool <br>
    Waarde: `true` of niet aanwezig

Het ^^vorfeld^^ is het zinsdeel vóór het finiete werkwoord in een
zin met de *verb second*-volgorde.
Zie [Wikipedia](https://de.wikipedia.org/wiki/Feldermodell_des_deutschen_Satzes#Das_Vorfeld)
De terminologie gaat terug op een oude traditie in Germaanse syntax.

In hoofdzinnen zoals:

|vorfeld|persoonsvorm|mittelfeld|werkwoorden|nachfeld
|:-----|:----------|:--------|:---------|:-------|
|Ik     |heb         |de kinderen vandaag kadootjes|beloofd|omdat het feest is|
|Hij    |moet        |zijn jas in de trein | hebben laten liggen | met zijn dronken kop|
|De kinderen | roepen | scheldwoorden naar de stagiaire | | |

staat de persoonsvorm op de tweede plaats. Het zinsdeel dat daaraan voorafgaat wordt ^^vorfeld^^ genoemd.
De zinsdelen die volgen op de persoonsvorm en voorafgaan aan de eventuele overige werkwoorden staan in het ^^mittelfeld^^.
Het vijfde en laatste deel van de hoofdzin is dan het ^^nachfeld^^.

Voobeelden:

 * ^^In elk geval^^ kan men stellen dat er onzekerheid heerst over de vraag of de rente nog verder zal stijgen .
 * ^^Dit effect^^ is nu bezig te verdwijnen .
 * ^^Aan het slot^^ was de markt op het beste niveau van de dag en liep de ticker drie minuten achter .
 * ^^In een plaats hier vlakbij^^ werd onlangs een voorstel voor de gemeenteraad gebracht .
 * ^^Wie ironisch is^^ zegt het tegendeel van hetgeen hij meent .
 * ^^Wie^^ loopt daar ?

^^Definitie^^

```cypher
match (x:nw)
where x.sentid + ' ' + x.id in (
    select sid
    from (
        match (n:node)-[r:rel*0..1{rel:'body'}]->(nn:node)-[:rel{rel:'hd'}]->(fin:word)
        where ( n.cat = 'smain' and length(r) = 0 )
           or ( n.cat = 'whq' and length(r) = 1 and nn.cat = 'sv1' )
        match (n)-[r2:rel*1..{primary:true}]->(topic:nw)-[rel:rel*0..1]->(htopic:nw)
        where ( n.cat = 'smain' or r2[0].rel = 'whd' )
          and ( ( htopic.lemma is not null
                  and htopic.begin < fin.begin
                  and ( length(rel) = 0
                        or rel[0].rel in ['hd','cmp','crd']
                      )
                )
                or
                ( topic.begin < fin.begin
                  and
                  topic.end <= fin.begin
                )
              )
        return topic.sentid + ' ' + topic.id as sid, n.id as nid

        except

        match (n:node)-[r:rel*0..1{rel:'body'}]->(nn:node)-[:rel{rel:'hd'}]->(fin:word)
        where ( n.cat = 'smain' and length(r) = 0 )
           or ( n.cat = 'whq' and length(r) = 1 and nn.cat = 'sv1' )
        match (n)-[r2:rel*1..{primary:true}]->(topic:nw)-[rel:rel*0..1]->(htopic:nw)
        where ( n.cat = 'smain' or r2[0].rel = 'whd' )
          and ( ( htopic.lemma is not null
                  and htopic.begin < fin.begin
                  and ( length(rel) = 0
                        or rel[0].rel in ['hd','cmp','crd']
                      )
                )
                or
                ( topic.begin < fin.begin
                  and
                  topic.end <= fin.begin
                )
              )
        match (topic)<-[:rel*1..]-(nt:node)<-[:rel*1..]-(n)
        match (nt)-[relt:rel*0..1]->(hnt:nw)
        where ( hnt.lemma is not null
                and hnt.begin < fin.begin
                and ( length(relt) = 0
                      or relt[0].rel in ['hd','cmp','crd']
                    )
              )
              or
              ( nt.begin < fin.begin
                and
                nt.end <= fin.begin
              )
        return topic.sentid + ' ' + topic.id as sid, n.id as nid

    ) as foo
)
set x._vorfeld = true;
```

## Relaties

Extra attributen op relaties van het type `rel`.

De attributen `id` en `primary` hebben betrekking op relaties waarbij
in Alpino een index-node werd gebruikt.

Voorbeeld, dit in Alpino:

```xml
<node begin="7" end="9" id="17" index="3" rel="su"  cat="np"> ... </node>
...
<node begin="7" end="9" id="21" index="3" rel="obj1"/>
```

... wordt dit in AlpinoGraph:

```cypher
(:node)-[:rel{rel: 'su',   primary: true}]->
   (:node{begin: 7, end: 9, id: 17, cat: 'np'})
      <-[:rel{rel: 'obj1', primary: false, id: 21}]-(:node)
```

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
niet primair. Alle overige relaties zijn primair, ook daar waar geen
index-node was.
