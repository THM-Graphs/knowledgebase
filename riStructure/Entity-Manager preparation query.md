This query must be executed to prepare the database for the Entity-Manager-Application.

```cypher
// alte collections löschen
MATCH (c1)-[r:IN_DEPARTMENT]-(c2:Collection)-[r2:IN_VOLUME]-(rr)
SET rr.department = null, rr.volume = null
DELETE c1,r,c2,r2;
// UUIDs vergeben
MATCH (n:Entity) SET n.uuid = randomUUID();
// type übersetzen
MATCH (e:Entity)
WHERE toLower(e.type) IN ['ort', 'sache', 'ereignis']
SET e.type = CASE e.type
    WHEN 'ort' THEN 'place'
    WHEN 'sache' THEN 'thing'
    WHEN 'ereignis' THEN 'event'
END;

//RI III
MATCH (r:Regesta) WITH r, r.identifier AS identifier
WHERE identifier STARTS WITH "RI III,"
WITH
	r,
	"RI III" AS department,
	CASE
		WHEN identifier STARTS WITH "RI III,5" THEN "5"
		ELSE split(split(identifier, ' n. ')[0], 'RI III,')[1]
	END AS volume
SET r.volume = volume, r.department = department
MERGE (c1:Collection {type: "department", name: department})
MERGE (c2:Collection {type: "volume", name: volume})-[:IN_DEPARTMENT]->(c1)
MERGE (r)-[:IN_VOLUME]->(c2);

//RI VII Department
MATCH (r:Regesta) WITH r, r.identifier AS identifier
WHERE identifier STARTS WITH "[RI VII]"
	WITH r,
	"RI VII" AS department,
	split(split(identifier, ' n. ')[0], ' H. ')[1] AS volume
SET r.volume = volume, r.department = department
MERGE (c1:Collection {type: "department", name: department})

MERGE (c2:Collection {type: "volume", name: volume})-[:IN_DEPARTMENT]->(c1)

MERGE (r)-[:IN_VOLUME]->(c2);


//RI VII Department
MATCH (r:Regesta) WITH r, r.identifier AS identifier
WHERE identifier STARTS WITH "[RI VII]"
	WITH r,
	"RI VII" AS department,
	split(split(identifier, ' n. ')[0], ' H. ')[1] AS volume
SET r.volume = volume, r.department = department
MERGE (c1:Collection {type: "department", name: department})

MERGE (c2:Collection {type: "volume", name: volume})-[:IN_DEPARTMENT]->(c1)

MERGE (r)-[:IN_VOLUME]->(c2);

//RI VIII Department

MATCH (r:Regesta) WITH r, r.identifier AS identifier
WHERE identifier STARTS WITH "RI VIII"
	WITH
	r,
	"RI VIII" AS department,
	"1" AS volume
SET r.volume = volume, r.department = department
MERGE (c1:Collection {type: "department", name: department})

MERGE (c2:Collection {type: "volume", name: volume})-[:IN_DEPARTMENT]->(c1)

MERGE (r)-[:IN_VOLUME]->(c2);

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

//RI XI Department
MATCH (r:Regesta) WITH r, r.identifier AS identifier
WHERE identifier STARTS WITH "RI XI,"
WITH
	r,
	"RI XI" AS department,
	"1" AS volume
SET r.volume = volume, r.department = department
MERGE (c1:Collection {type: "department", name: department})
MERGE (c2:Collection {type: "volume", name: volume})-[:IN_DEPARTMENT]->(c1)
MERGE (r)-[:IN_VOLUME]->(c2);

MATCH (r:Regesta) WITH r, r.identifier AS identifier
WHERE identifier STARTS WITH "RI XI Neubearb"
WITH
	r,
	"RI XI" AS department,
	split(split(identifier, ' n. ')[0], 'RI XI ')[1] AS volume
SET r.volume = volume, r.department = department
MERGE (c1:Collection {type: "department", name: department})
MERGE (c2:Collection {type: "volume", name: volume})-[:IN_DEPARTMENT]->(c1)
MERGE (r)-[:IN_VOLUME]->(c2);

//RI XIII
MATCH (r:Regesta) WITH r, r.identifier AS identifier
WHERE identifier STARTS WITH "[RI XIII] "
WITH
	r,
	"RI XIII" AS department,
	CASE
		WHEN identifier STARTS WITH "[RI XIII] H." THEN split(split(identifier, ' n. ')[0], ' H. ')[1]
		ELSE split(split(identifier, ' n. ')[0], '[RI XIII] ')[1]
	END AS volume
SET r.volume = volume, r.department = department
MERGE (c1:Collection {type: "department", name: department})
MERGE (c2:Collection {type: "volume", name: volume})-[:IN_DEPARTMENT]->(c1)
MERGE (r)-[:IN_VOLUME]->(c2);


// Collections uuids geben
MATCH (n:Collection) SET n.uuid = randomUUID();
```