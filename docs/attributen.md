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

^^Definitie:^^

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

^^Definitie:^^

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

!!! info "Een hulpattribuut voor het zoeken naar *correlatieve comparatieven*"
    Items: `(:node)` <br>
    Type: bool <br>
    Waarde: `true` of niet aanwezig

Het attibuut `_deste` is `true` voor nodes die overeenkomen met deze xpath-expressie:

TODO: in dit geval **nadat** indexnodes worden geëxpandeerd (klopt dat?)

```xpath
//node[ node[ @graad="comp"]
        and
        node[ @lemma=("hoe", "deste")
              or
              ( node[@lemma="des"]
                and
                node[@lemma="te"]
              )
        ]
]
```

### _n_words

!!! info "Het aantal woorden dat deze node bestrijkt"
    Items: `(:node)`, `(:word)` <br>
    Type: int <br>
    Waarde: 1 of groter

Voor woorden is dit altijd 1.

Voor nodes is dit het aantal woorden onder de node die zowel via
[primaire](#primary) als via niet-primaire relaties kan worden bereikt.

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


Het attribuut `_vorfeld` is `true` voor nodes en woorden die overeenkomen met
deze xpath-expressie:

TODO: in dit geval **voordat** indexnodes worden geëxpandeerd

TODO: definitie hieronder updaten

```xpath
//node[( (  ancestor::node[@cat="smain"]/node[@rel="hd"]/number(@begin) > node[@rel=("hd","cmp","mwp","crd","rhd","whd","nucl","dp")]/number(@begin)
            or
            (  ancestor::node[@cat="smain"]/node[@rel="hd"]/number(@begin) > number(@begin)
               and
               not(node[@rel=("hd","cmp","mwp","crd","rhd","whd","nucl","dp")])
            )
         )
         and
         not(parent::node[(  ancestor::node[@cat="smain"]/node[@rel="hd"]/number(@begin) > node[@rel=("hd","cmp","mwp","crd","rhd","whd","nucl","dp")]/number(@begin)
                             or
                             (  ancestor::node[@cat="smain"]/node[@rel="hd"]/number(@begin) > number(@begin)
                                and
                                not(node[@rel=("hd","cmp","mwp","crd","rhd","whd","nucl","dp")])
                             )
                          )])
         and
         (@cat or @pt)
       )]
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
