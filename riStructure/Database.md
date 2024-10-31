https://datascience.mni.thm.de:7473/browser/

## Department and Volume Nodes from the Regest-Identifier

### Alles löschen
````cypher
MATCH (c1)-[r:IN_DEPARTMENT]-(c2:Collection)-[r2:IN_VOLUME]-(rr)
SET rr.department = null, rr.volume = null
DELETE c1,r,c2,r2;
`````

### RI III
```cypher
//RI III
MATCH (r:Regesta) WITH r, r.identifier AS identifier
WHERE identifier STARTS WITH "RI III,"
WITH
	r,
	"RI03" AS department,
	CASE
		WHEN identifier STARTS WITH "RI III,5" THEN "5"
		ELSE split(split(identifier, ' n. ')[0], 'RI III,')[1]
	END AS volume
SET r.volume = volume, r.department = department
MERGE (c1:Collection {type: "department", name: department})
MERGE (c2:Collection {type: "volume", name: volume})-[:IN_DEPARTMENT]->(c1)
MERGE (r)-[:IN_VOLUME]->(c2);
```


### RI VII
```cypher
//RI VII Department
MATCH (r:Regesta) WITH r, r.identifier AS identifier
WHERE identifier STARTS WITH "[RI VII]"
	WITH r,
	"RI07" AS department,
	split(split(identifier, ' n. ')[0], ' H. ')[1] AS volume
SET r.volume = volume, r.department = department
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
	"RI08" AS department,
	"1" AS volume
SET r.volume = volume, r.department = department
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
	"1" AS volume
SET r.volume = volume, r.department = department
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
	r,
	"RI11" AS department,
	"1" AS volume
SET r.volume = volume, r.department = department
MERGE (c1:Collection {type: "department", name: department})
MERGE (c2:Collection {type: "volume", name: volume})-[:IN_DEPARTMENT]->(c1)
MERGE (r)-[:IN_VOLUME]->(c2);

MATCH (r:Regesta) WITH r, r.identifier AS identifier
WHERE identifier STARTS WITH "RI XI Neubearb"
WITH
	r,
	"RI11" AS department,
	split(split(identifier, ' n. ')[0], 'RI XI ')[1] AS volume
SET r.volume = volume, r.department = department
MERGE (c1:Collection {type: "department", name: department})
MERGE (c2:Collection {type: "volume", name: volume})-[:IN_DEPARTMENT]->(c1)
MERGE (r)-[:IN_VOLUME]->(c2);
````


### RI XIII
```cypher
//RI XIII
MATCH (r:Regesta) WITH r, r.identifier AS identifier
WHERE identifier STARTS WITH "[RI XIII] "
WITH
	r,
	"RI13" AS department,
	CASE
		WHEN identifier STARTS WITH "[RI XIII] H." THEN split(split(identifier, ' n. ')[0], ' H. ')[1]
		ELSE split(split(identifier, ' n. ')[0], '[RI XIII] ')[1]
	END AS volume
SET r.volume = volume, r.department = department
MERGE (c1:Collection {type: "department", name: department})
MERGE (c2:Collection {type: "volume", name: volume})-[:IN_DEPARTMENT]->(c1)
MERGE (r)-[:IN_VOLUME]->(c2);
`````