# Forschungsfragen

## Wanderstationen, Martin Ruarus
Insbesondere: Datum, Ort, Personen

```cypher
// Wanderstationen, Martin Ruarus
// Params: guid/uuid/uid
MATCH (ruarus:Entity {guid: 'zrl_x1x_12b'})

MATCH (ruarus)-[:SENT_BY]-(sent:Sent)-[:SENT_FROM]-(place:Entity {type: "place"})
MATCH (sent)<-[:HAS_ANNOTATION]-(metadata:Metadata)
MATCH (metadata)-[:HAS_ANNOTATION]->(received:Received)-[:RECEIVED_BY]->(receiver:Entity)

WITH sent.dateStart AS date, place, metadata, receiver
ORDER BY date

WITH collect({date: date, place: place, metadata: metadata, receiver: receiver}) AS letters
WITH letters, range(0, size(letters) - 2) AS idxs
UNWIND idxs AS i

WITH letters[i] AS from, letters[i+1] AS to
WHERE from.place <> to.place

// vRels: NEXT, HAS_SENT, RECEIVED_BY
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

## Zusammenhänge mit Vakuum

Wo und in welchen Zusammenhängen erscheint „Vakuum“ in Verbindung z.B. mit „spatium imaginarium“, „Descartes, René“, „Aristoteles“, „Lehre, epikureische“, „Epikur“, „Lucretius Carus , Titus“.

```cypher
// Zusammenhänge mit Vakuum
// Params: entityTerms, contextTerms
WITH 
  ['Vakuum', 'Raum, leerer', 'Raum, luftleerer', 'Luft-Vakuum-Grenze', 'Vakuumpumpe'] AS entityTerms,
  ['Descartes, René', 'Aristoteles', 'Spatium imaginarium', 'Lehre, epikureische', 
   'Lucretius Carus, Titus', 'Epicurus', 'Epikur'] AS contextTerms

MATCH (text:Text)-[:HAS_ANNOTATION]->(:Spo {teiType: "rs"})-[:REFERS_TO]->(entity:Entity)
WHERE entity.label IN entityTerms OR entity.label IN contextTerms
MATCH (metadata:Metadata)-[:HAS_TEXT]->(text)

OPTIONAL MATCH (metadata)-[:HAS_ANNOTATION]->(sent:Sent)
OPTIONAL MATCH (sent)-[:SENT_BY]->(sender:Entity {type: "person"})
OPTIONAL MATCH (sent)-[:SENT_FROM]->(place:Entity {type: "place"})

// Duplicate rels, because of (e1)->(m)<-(e2)
// vRels: MENTIONED_IN, SENT_BY, SENT_FROM
WITH DISTINCT entity, metadata, sent, sender, place
CALL apoc.create.vRelationship(entity, 'MENTIONED_IN', {}, metadata) YIELD rel AS rel1
WITH entity, rel1, metadata, sent, sender, place

WITH entity, rel1, metadata, sent, sender, place
WHERE sender IS NOT NULL
CALL apoc.create.vRelationship(metadata, 'SENT_BY', {date: sent.dateStart}, sender) YIELD rel AS rel2
WITH entity, rel1, rel2, metadata, sent, sender, place

WHERE place IS NOT NULL
CALL apoc.create.vRelationship(metadata, 'SENT_FROM', {date: sent.dateStart}, place) YIELD rel AS rel3

RETURN DISTINCT entity, rel1, metadata, rel2, sender, rel3, place
```

## Zusammenhänge mit Astrologie, Einfluss ‚mala malis, bona bonis

Wo erscheint der Registereintrag „Astrologie, Einfluss ‚mala malis, bona bonis‘“, und in welchen Verbindungen, z.B. mit „Komet, Unglückszeichen“, „Aberglaube“, „Idololatrie“, „Astrologie, ‚astra inclinant, non necessitant‘“, „Astrologie, ‚sapiens dominabitur astris‘“, „Kometen, Theorie des Aristoteles“, „Kometen, sublunar“.

```cypher
// Zusammenhänge mit Astrologie, Einfluss, mala malis, bona bonis
// Params: entityTerms, contextTerms
WITH 
  ['Astrologie, Einfluss ‚mala malis, bona bonis‘'] AS entityTerms,
  ['Komet, Unglückszeichen', 'Aberglaube', 'Idololatrie', 
   'Astrologie, ‚astra inclinant, non necessitant‘', 
   'Astrologie, ‚sapiens dominabitur astris‘', 
   'Kometen, Theorie des Aristoteles', 'Kometen, sublunar'] AS contextTerms

MATCH (text:Text)-[:HAS_ANNOTATION]->(:Spo {teiType: "rs"})-[:REFERS_TO]->(entity:Entity)
WHERE entity.label IN entityTerms OR entity.label IN contextTerms
MATCH (metadata:Metadata)-[:HAS_TEXT]->(text)

OPTIONAL MATCH (metadata)-[:HAS_ANNOTATION]->(sent:Sent)
OPTIONAL MATCH (sent)-[:SENT_BY]->(sender:Entity {type: "person"})
OPTIONAL MATCH (sent)-[:SENT_FROM]->(place:Entity {type: "place"})

// Duplicate rels, because of (e1)->(m)<-(e2)
// vRels: MENTIONED_IN, SENT_BY, SENT_FROM
WITH DISTINCT entity, metadata, sent, sender, place
CALL apoc.create.vRelationship(entity, 'MENTIONED_IN', {}, metadata) YIELD rel AS rel1
WITH entity, rel1, metadata, sent, sender, place

WITH entity, rel1, metadata, sent, sender, place
WHERE sender IS NOT NULL
CALL apoc.create.vRelationship(metadata, 'SENT_BY', {date: sent.dateStart}, sender) YIELD rel AS rel2
WITH entity, rel1, rel2, metadata, sent, sender, place

WHERE place IS NOT NULL
CALL apoc.create.vRelationship(metadata, 'SENT_FROM', {date: sent.dateStart}, place) YIELD rel AS rel3

RETURN DISTINCT entity, rel1, metadata, rel2, sender, rel3, place
```

## Konstellation mit Entität: relig., phil. Freiheit

Konstellationen der Registereinträge „Freiheit“, „Freiheit, philosophische“ und „Freiheit, religiöse“ in Verbindung mit Personen und Orten

```cypher
// Konstellation mit Entitäten bzgl. Freiheit
// Params: entity labels
MATCH (e:Entity)
WHERE e.label IN ['Freiheit', 'Freiheit, philosophische', 'Freiheit, religiöse']

MATCH (e)<-[:REFERS_TO]-(s:Spo)-[:HAS_ANNOTATION|COMMENT_ON]-(t:Text)<-[:HAS_TEXT]-(m:Metadata)

OPTIONAL MATCH (m)-[:HAS_ANNOTATION]->(sent:Sent)
OPTIONAL MATCH (sent)-[:SENT_BY]->(author:Entity)
WHERE author.type = 'person'

OPTIONAL MATCH (sent)-[:SENT_FROM]->(origin:Entity)
OPTIONAL MATCH (m)-[:HAS_ANNOTATION]->(received:Received)
OPTIONAL MATCH (received)-[:SENT_TO]->(recipient:Entity)
OPTIONAL MATCH (received)-[:RECEIVED_BY]->(receiver:Entity)

WITH DISTINCT e, m, author, origin, recipient, receiver

CALL {
  WITH e, m
  RETURN apoc.create.vRelationship(e, 'MENTIONED_IN', {}, m) AS virtualEdge
}

CALL {
  WITH author, m
  RETURN CASE 
    WHEN author IS NOT NULL 
    THEN apoc.create.vRelationship(m, 'SENT_BY', {}, author) 
    ELSE NULL 
  END AS vSentBy
}

CALL {
  WITH origin, m
  RETURN CASE 
    WHEN origin IS NOT NULL 
    THEN apoc.create.vRelationship(origin, 'SENT_FROM', {}, m) 
    ELSE NULL 
  END AS vSentFrom
}

CALL {
  WITH recipient, m
  RETURN CASE 
    WHEN recipient IS NOT NULL 
    THEN apoc.create.vRelationship(recipient, 'SENT_TO', {}, m) 
    ELSE NULL 
  END AS vSentTo
}

CALL {
  WITH receiver, m
  RETURN CASE 
    WHEN receiver IS NOT NULL 
    THEN apoc.create.vRelationship(m, 'RECEIVED_BY', {}, receiver) 
    ELSE NULL 
  END AS vReceivedBy
}

CALL {
  WITH author, e
  RETURN CASE 
    WHEN author IS NOT NULL 
    THEN apoc.create.vRelationship(e, 'MENTIONED_BY', {}, author) 
    ELSE NULL 
  END AS vMentionedBy
}

RETURN e, m, virtualEdge, vSentBy, vSentFrom, vSentTo, vReceivedBy, vMentionedBy,
       author, origin, recipient, receiver
```
