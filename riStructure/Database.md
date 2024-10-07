https://datascience.mni.thm.de:7473/browser/

## Department and Volume Nodes from the Regest-Identifier


### RI VII
```cypher
//RI VII Department
MATCH (r:Regesta) WITH r, r.identifier AS identifier
WHERE identifier STARTS WITH "[RI VII]"
	WITH r,
	"RI VII" AS department,
	toInteger(split(split(identifier, ' n. ')[0], ' H. ')[1]) AS volume

MERGE (c1:Collection {type: "department", name: department})

MERGE (c2:Collection {type: "volume", name: volume})-[:IN_DEPARTMENT]->(c1)

MERGE (r)-[:IN_VOLUME]->(c2);
```

### RI VIII
```cypher
//RI VIII Department

MATCH (r:Regesta) WITH r, r.identifier AS identifier
WHERE identifier STARTS WITH "RI VIII"
	WITH
	r,
	"RI VIII" AS department,
	1 AS volume

MERGE (c1:Collection {type: "department", name: department})

MERGE (c2:Collection {type: "volume", name: volume})-[:IN_DEPARTMENT]->(c1)

MERGE (r)-[:IN_VOLUME]->(c2);
```

### Pfalzgrafen 2
````cypher
//Regg. Pfalzgrafen 2 Department

MATCH (r:Regesta) WITH r, r.identifier AS identifier
WHERE identifier STARTS WITH "[Regg. Pfalzgrafen 2]"
WITH
	r,
	"Pfalzgrafen 2" AS department,
	1 AS volume
MERGE (c1:Collection {type: "department", name: department})

MERGE (c2:Collection {type: "volume", name: volume})-[:IN_DEPARTMENT]->(c1)

MERGE (r)-[:IN_VOLUME]->(c2);
````

### RI XI
```cypher
//RI XI Department
MATCH (r:Regesta) WITH r, r.identifier AS identifier
WHERE identifier STARTS WITH "RI XI,"
WITH
	"RI XI" AS department,
	1 AS volume
MERGE (c1:Collection {type: "department", name: department})
MERGE (c2:Collection {type: "volume", name: volume})-[:IN_DEPARTMENT]->(c1)
MERGE (r)-[:IN_VOLUME]->(c2);

MATCH (r:Regesta) WITH r, r.identifier AS identifier
WHERE identifier STARTS WITH "RI XI Neubearb"
WITH
	"RI XI" AS department,
	split(split(identifier, ' n. ')[0], 'RI XI ')[1] AS volume
MERGE (c1:Collection {type: "department", name: department})
MERGE (c2:Collection {type: "volume", name: volume})-[:IN_DEPARTMENT]->(c1)
MERGE (r)-[:IN_VOLUME]->(c2);
````