# Forschungsfragen

## Wanderstationen, Martin Ruarus
Insbesondere: Datum, Ort, Personen

```cypher
// Wanderstationen, Martin Ruarus
MATCH (ruarus:Entity {guid: 'zrl_x1x_12b'})

// Traverse to the sent letter, origin place, metadata and receiver
MATCH (ruarus)-[:SENT_BY]-(sent:Sent)-[:SENT_FROM]-(place:Entity {type: "place"})
MATCH (sent)<-[:HAS_ANNOTATION]-(metadata:Metadata)
MATCH (metadata)-[:HAS_ANNOTATION]->(received:Received)-[:RECEIVED_BY]->(receiver:Entity)

WITH sent.dateStart AS date, place, metadata, receiver
ORDER BY date

// Collect letters and prepare index pairs
WITH collect({date: date, place: place, metadata: metadata, receiver: receiver}) AS letters
WITH letters, range(0, size(letters) - 2) AS idxs
UNWIND idxs AS i

// Only connect if place changed
WITH letters[i] AS from, letters[i+1] AS to
WHERE from.place <> to.place

// Create virtual relationships for visualization
WITH from, to,
apoc.create.vRelationship(
  from.place,
  'NEXT',
  { lastDateBeforeNext: from.date },
  to.place
) AS rel,

apoc.create.vRelationship(
  from.place,
  'HAS_SENT',
  { date: from.date },
  from.metadata
) AS fromMetaRel,

apoc.create.vRelationship(
  from.metadata,
  'RECEIVED_BY',
  {},
  from.receiver
) AS receiverRel

RETURN
from.place AS from,
to.place AS to,
from.metadata AS fromMetadata,
from.receiver AS receiver,
rel,
fromMetaRel,
receiverRel
```

## Zusammenhänge mit Vacuum

Wo und in welchen Zusammenhängen erscheint „Vacuum“ in Verbindung z.B. mit „spatium imaginarium“, „Descartes, René“, „Aristoteles“, „Lehre, epikureische“, „Epikur“, „Lucretius Carus , Titus“.

```cypher
WITH 
  ['Vakuum', 'Raum, leerer', 'Raum, luftleerer', 'Luft-Vakuum-Grenze', 'Vakuumpumpe'] AS vacuumTerms,
  ['Descartes, René', 'Aristoteles', 'Spatium imaginarium', 'Lehre, epikureische', 
   'Lucretius Carus, Titus', 'Epicurus', 'Epikur'] AS philosophyTerms

// Find relevant entity mentions in texts via annotations
MATCH (text:Text)-[:HAS_ANNOTATION]->(annot:Spo {teiType: "rs"})-[:REFERS_TO]->(entity:Entity)
WHERE entity.label IN vacuumTerms OR entity.label IN philosophyTerms

WITH text, entity, vacuumTerms, philosophyTerms

// Create virtual relationship from Entity to Text (skips annotation nodes in the graph view)
CALL apoc.create.vRelationship(entity, 'MENTIONED_IN', {}, text) YIELD rel

RETURN DISTINCT entity, rel, text
```

## 