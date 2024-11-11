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


//RI I DATENMODELL STIMMT NICHT ÜBEREIN
MATCH (r:Regesta) WITH r, r.identifier AS identifier
WHERE identifier STARTS WITH "RI I n." OR identifier STARTS WITH "RI I,"
WITH
	r,
	"RI01" AS department,
	CASE
		WHEN identifier STARTS WITH "RI I n." THEN "1"
		WHEN identifier STARTS WITH "RI I,2" THEN "2"
		WHEN identifier STARTS WITH "RI I,3" THEN "3"
		WHEN identifier STARTS WITH "RI I,4" THEN "4"
		ELSE "no volume"
	END AS volume
SET r.volume = volume, r.department = department
MERGE (c1:Collection {type: "department", name: department})
MERGE (c2:Collection {type: "volume", name: volume})-[:IN_DEPARTMENT]->(c1)
MERGE (r)-[:IN_VOLUME]->(c2);


//RI II DATENMODELL STIMMT NICHT ÜBEREIN
MATCH (r:Regesta) WITH r, r.identifier AS identifier
WHERE identifier STARTS WITH "RI II,"
WITH
	r,
	"RI02" AS department,
	split(split(identifier, ' n.')[0], 'RI II,')[1] AS volume
SET r.volume = volume, r.department = department
MERGE (c1:Collection {type: "department", name: department})
MERGE (c2:Collection {type: "volume", name: volume})-[:IN_DEPARTMENT]->(c1)
MERGE (r)-[:IN_VOLUME]->(c2);

 
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

//RI VI
MATCH (r:Regesta)
WHERE r.identifier STARTS WITH "RI VI,4,"
WITH r, r.identifier as identifier,
"RI06" AS department,
"4" AS volume
SET r.volume = volume, r.department = department
WITH r, volume, department
MERGE (c1:Collection {type: "department", name: department})
MERGE (c2:Collection {type: "volume", name: volume})
MERGE (c2)-[:IN_DEPARTMENT]->(c1)
MERGE (r)-[:IN_VOLUME]->(c2);
MATCH (r:Regesta)
WHERE r.identifier starts with "[RIplus] Regg. Heinrich VII. n. "
WITH r, r.identifier as identifier,
"RI06" AS department,
"[RIplus] Regg. Heinrich VII." AS volume
MATCH (c1:Collection {type: "department", name: "RI06"})
SET r.volume = volume, r.department = department
WITH r, volume, department, c1
//MERGE (c1:Collection {type: "department", name: department})
MERGE (c2:Collection {type: "volume", name: volume})
MERGE (c2)-[:IN_DEPARTMENT]->(c1)
MERGE (r)-[:IN_VOLUME]->(c2);

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


// Collections uuids geben
MATCH (n:Collection) SET n.uuid = randomUUID();

// Annotationsknoten erstellen aus APPEARS_IN-Kanten 
Match (r:Regesta)<-[rel:APPEARS_IN]-(e:Entity) create (a:Annotation {riType: "role", riRole: rel.type}) create (r)-[:HAS_ANNOTATION]->(a)-[:REFERS_TO]->(e) detach delete rel;

// Entities bekommen department und volume property
MATCH (n:Entity)<-[:REFERS_TO]-(a:Annotation)<-[:HAS_ANNOTATION]-(r:Regesta)
SET n.volume = r.volume, n.department = r.department;

```