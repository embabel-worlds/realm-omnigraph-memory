# realm-omnigraph-memory

A second memory system under the shared `Memory` label: facts held in an
[Omnigraph](https://github.com/modernrelay/omnigraph) graph, recalled by BM25 through the graph's
own stored query, so "what do you remember about X" answers from the assistant's memory and from
Omnigraph in one Virtual Cypher query.

```cypher
MATCH (:Concept {value:'coffee'})-[r:RELEVANT_TO]->(m:Memory)          -- every memory system
RETURN m.source, m.text, r.score, r.mode ORDER BY m.source, r.rank

MATCH (:Concept {value:'coffee'})-[r:RELEVANT_TO]->(m:OmnigraphMemory) -- this one alone
RETURN m.text, r.score
```

Every `OmnigraphMemory` node carries `:Memory` and the realm label `:Omnigraph`.

## What it expects of the Omnigraph side

A graph named `memory` with a `Memory` node type carrying `mid` (key), `text` (indexed, so BM25
can read it), `confidence`, `status`, `created` and `source`, and a stored query `recall($q)` that
returns ACTIVE memories ordered by `bm25($m.text, $q)` with the score projected as `score`. The
`omnigraph/` directory holds the cluster bundle used to prove it (`cluster.yaml`, `memory.pg`,
`queries.gq`) and a sample `memories.jsonl`.

`apis/omnigraph-openapi.json` is the server's own `GET /openapi.json` with a `servers` entry added
(the served document has none) and the stored-query body made a plain object so its fields flatten
into the gateway operation. Point `servers[0].url` at your server. The dev bundle runs
`omnigraph-server --unauthenticated`; a deployed one needs `auth: bearer` in `apis/omnigraph.yml`.

## Contract

See `VIRTUAL_CYPHER.md` §6.8 in realm-spec: the `Memory` data contract (`id`, `text`,
`confidence`, `status`, `created`, `source`), the three modes, and the curation rule
(ACTIVE only; CONTRADICTED never answers). This realm meets it by projecting the stored query's
columns straight onto the node and by filtering status in the query itself.
