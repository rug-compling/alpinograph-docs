# Alpino-corpora in AgensGraph

Een `<node>` in alpino\_ds is een `(:node)` of een `(:word)` in AgensGraph

`(:nw)` is een alias voor `(:node)` of `(:word)`

Dit in alpino\_ds:

```xml
<node id="11" ... >
  <node id="12" rel="hd" ... />
</node>
```

... is dit in AgensGraph:

```cypher
(:node{id: '11', ... })-[:rel{rel: 'hd'}]->(:node{id: '12', ... })
```

... of, als de binnenste node een woord is:

```cypher
(:node{id: '11', ... })-[:rel{rel: 'hd'}]->(:word{id: '12', ... })
```


Dit:

```cypher
match (n:node{cat: 'np'}) return n;
```

... is sneller dan dit:

```cypher
match (n{cat: 'np'}) return n;
```

... omdat agens in het tweede geval ook gaat zoeken in de type items
zonder `cat`, en omdat daarvoor geen index voor `cat` is moeten alle items
bekeken worden.

Items `(:node)` en `(:word)` en dus `(:nw)` hebben dezelfde indexen, ook al
zijn die voor `(:node)` of `(:word)` leeg.


## Patronen

```cypher
-- Alpino-relaties
(:sentence)-[:rel{rel: 'top'}]->(:node{cat: 'top'})
(:node)-[:rel]->(:node)
(:node)-[:rel]->(:word)
(:node)-[:rel]->(:nw)           -- link naar :node of :word

-- relaties tussen woordparen
(:sentence)-[:pair]->(:word)    -- enkelzijdige relatie, zoals hd/-
(:word)-[:pair]->(:word)
(:word)-[:pair]->(:node{cat: 'mwu'})
(:node{cat: 'mwu'})-[:pair]->(:word)
(:node{cat: 'mwu'})-[:pair]->(:node{cat: 'mwu'})
(:nw)-[:pair]->(:nw)            -- link van :node of :word naar :node of :word

-- basic universal dependencies
(:sentence)-[:ud{rel: 'root', main: 'root'}]->(:word)
(:word)-[:ud]->(:word)

-- enhanced universal dependencies
(:sentence)-[:eud{rel: 'root', main: 'root'}]->(:word)
(:word)-[:eud]->(:word)

-- alle universal dependencies: dep is een alias voor ud of eud
(:sentence)-[:dep{rel: 'root', main: 'root'}]->(:word)
(:word)-[:dep]->(:word)

-- opeenvolgende tokens
(:word)-[:next]->(:word)
(:word{end: 1})-[:next*]->(:word{last: true})    -- de hele zin

-- metadata
(:meta)

-- documentatie over het corpus
(:doc)

-- tellingen van attributen en metadata
(:feature)

-- voor intern gebruik
(:data)
```

## Attributen van items

### `(:sentence)`

attribuut       | type   | opmerkingen
----------------|--------|------------
`sentid`        | string |
`text`          | string |
`tokens`        | string | getokeniseerde tekst
`len`           | int    | aantal tokens
`cats`          | int    | parser-succes
`skips`         | int    | parser-succes
`build`         | string | versie van Alpino
`date`          | string | datum en tijd van parsen door Alpino
`conllu_status` | string | `OK`, `error`
`conllu_error`  | string |  als `conllu_status` != `OK`

### `(:node)`

attribuut | type   | opmerkingen
----------|--------|------------
`sentid`  | string |
`id`      | int    |
`begin`   | int    |
`end`     | int    |
...       | string | alle overige attributen uit de Alpino-node behalve `rel` en `index`
`_clause`  | bool   | zie [hier](../attributen/#_clause)
`_clause_lvl` | int | zie [hier](../attributen/#_clause_lvl)
`_deste`   | bool   | zie [hier](../attributen/#_deste)
`_n_words` | int    | zie [hier](../attributen/#_n_words)
`_np`      | bool   | zie [hier](../attributen/#_np)
`_vorfeld` | bool   | zie [hier](../attributen/#_vorfeld)

voor `cat` == `mwu` ook:

attribuut   | type   | opmerkingen
------------|--------|------------
`pt`        | string | `mwu`
`word`      | string |
`lemma`     | string |

### `(:word)`

attribuut   | type   | opmerkingen
------------|--------|------------
`sentid`    | string |
`last`      | bool   | `true` --- alleen voor laatste token in de zin
`id`        | int    |
`begin`     | int    |
`end`       | int    |
`getal_n`   | string | in plaats van `getal-n`
...         | string | alle overige attributen uit de Alpino-node behalve `rel` en `index`
`upos`      | string | het veld `UPOS` van CoNLL-U
`nospaceafter` | bool | `true` als CoNLL-U het extra attribuut `SpaceAfter=No` heeft
...         | string | alle features uit het veld `FEATS` van CoNLL-U, met hoofdletters
`_n_words` | int    | zie [hier](../attributen/#_n_words)
`_np`      | bool   | zie [hier](../attributen/#_np)
`_vorfeld` | bool   | zie [hier](../attributen/#_vorfeld)

Bij het zoeken naar CoNLL-U-features dubbele aanhalingstekens
gebruiken, vanwege de hoofdletters:

```cypher
match (w:word{"Gender": 'Com'}) return w;
```

### `(:meta)`

attribuut   | type   | opmerkingen
------------|--------|------------
`sentid`    | string |
`type`      | string | `text`, `int`, `float`, `date`, `datetime`
`name`      | string |
`value`     | string/number | number voor `int` en `float`

### `(:doc)`

attribuut   | type   | opmerkingen
------------|--------|------------
`alud_version` | string | versie van de automatische afleiding van Universal Dependencies


### `(:feature)`

attribuut   | type   | opmerkingen
------------|--------|------------
`v`         | string | `meta`, `node`, `word`, `rel`
`name`      | string |
`count`     | int    |

### `(:data)`

attribuut   | type   | opmerkingen
------------|--------|------------
`class`     | string | `node`, `rel`
`name`      | string | naam in AlpinoGraph: `x_`..., met `[^a-zA-Z0-9]+` → `_`
`type`      | string | `text`, `int`, `float`, `bool`, `date`, `datetime`
`oriname`   | string | oorspronkelijke naam in Alpino

Voor intern gebruik, voor het reconstrueren van alpino_ds met door gebruiker toegevoegde attributen.

## Attributen van relaties

### `[:rel]`

attribuut   | type   | opmerkingen
------------|--------|------------
`rel`       | string |
`primary`  | bool   | zie [hier](../attributen/#relaties)
`id`        | int    | zie [hier](../attributen/#relaties)

### `[:pair]`

attribuut   | type   | opmerkingen
------------|--------|------------
`rel`       | string |

### `[:ud]`

attribuut   | type   | opmerkingen
------------|--------|------------
`rel`       | string | *main*, *main*`:`*aux*
`main`      | string |
`aux`       | string |

### `[:eud]`

attribuut   | type   | opmerkingen
------------|--------|------------
`rel`       | string | *main*, *main*`:`*aux*
`main`      | string |
`aux`       | string |
`from`      | string | indien niet gelijk aan waarde van `end`
`to`        | string | indien niet gelijk aan waarde van `end`

### `[:next]`

geen attributen

