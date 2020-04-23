# Receptenboek

## Tellen van reeksen

Stap 1: zoek iets

```text
match (n:node{_deste: true})
return distinct n.sentid as sid, n.id
order by sid, id
```

Stap 2: zoek de pt van woorden onder iets

<pre><code class="text">
match (n:node{_deste: true})
<span class="prediff">match (n)-[:rel*]->(w:word)</span>
return distinct n.sentid as sid, n.id<span class="prediff">, w.end as positie, w.pt as pt</span>
order by sid, id, positie
</code></pre>

Stap 3: voeg de pt van woorden per iets samen

<pre><code class="text">
<span class="prediff">select sid, id, json_agg(pt) as pt_list
from (</span>
  match (n:node{_deste: true})
  match (n)-[:rel*]->(w:word)
  return distinct n.sentid as sid, n.id, w.end as positie, w.pt as pt
  order by sid, id, positie
<span class="prediff">) as foo
group by sid, id
order by sid, id</span>
</code></pre>

Stap 3a: maak het wat leesbaarder

<pre><code class="text">
select sid, id, <span class="prediff">string_agg(trim(both '"' from pt::text), ' ')</span> as pt_list
from (
  match (n:node{_deste: true})
  match (n)-[:rel*]->(w:word)
  return distinct n.sentid as sid, n.id, w.end as positie, w.pt as pt
  order by sid, id, positie
) as foo
group by sid, id
order by sid, id
</code></pre>

Stap 3b: variant om te kijken of de woorden wel direct achter elkaar staan

<pre><code class="text">
select sid, id, string_agg(trim(both '"' from pt::text), ' ') as pt_list
from (
  match (n:node{_deste: true})
  match (n)-[:rel*]->(w:word)
  return distinct n.sentid as sid, n.id, w.end as positie, <span class="prediff">w.end + ' ' + w.pt</span> as pt
  order by sid, id, positie
) as foo
group by sid, id
order by sid, id
</code></pre>

Stap 4: tel de frequenties van pt van woorden onder iets

<pre><code class="text">
<span class="prediff">select count(pt_list) as aantal, pt_list
from (</span>
  select string_agg(trim(both '"' from pt::text), ' ') as pt_list
  from (
    match (n:node{_deste: true})
    match (n)-[:rel*]->(w:word)
    return distinct n.sentid as sid, n.id as id, w.end as nummer, w.pt as pt
    order by sid, id, nummer
  ) as foo
  group by sid, id
<span class="prediff">) as bar
group by pt_list
order by aantal desc, pt_list</span>
</code></pre>
