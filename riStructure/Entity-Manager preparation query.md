This query must be executed to prepare the database for the Entity-Manager-Application.

```cypher


// alte collections löschen
MATCH (c1:Collection {type: "department"})<-[r:PART_OF]-(c2:Collection {type: "volume"})<-[r2:PART_OF]-(rr)
SET rr.department = null, rr.volume = null
DELETE c1,r,c2,r2;
// UUIDs vergeben
MATCH (n:Entity) 
WHERE n.uuid IS NULL
SET n.uuid = randomUUID();
// type übersetzen
MATCH (e:Entity)
WHERE toLower(e.type) IN ['ort', 'sache', 'ereignis']
SET e.type = CASE e.type
    WHEN 'ort' THEN 'place'
    WHEN 'sache' THEN 'thing'
    WHEN 'ereignis' THEN 'event'
END;
CREATE INDEX IF NOT EXISTS FOR (n:Collection) ON (n.type);
CREATE INDEX IF NOT EXISTS FOR (n:Collection) ON (n.department);
CREATE INDEX IF NOT EXISTS FOR (n:Collection) ON (n.uuid);
CREATE INDEX IF NOT EXISTS FOR (n:Entity) ON (n.parentId);
CREATE INDEX IF NOT EXISTS FOR (n:Entity) ON (n.department);
CREATE INDEX IF NOT EXISTS FOR (n:Entity) ON (n.volume);
CREATE FULLTEXT INDEX search IF NOT EXISTS FOR (n:Entity) ON EACH [n.label, n.origLabel];

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
MERGE (c1:Collection {type: "department", label: department})
MERGE (c2:Collection {type: "volume", label: volume})-[:PART_OF]->(c1)
MERGE (r)-[:PART_OF]->(c2);


//RI II DATENMODELL STIMMT NICHT ÜBEREIN
MATCH (r:Regesta) WITH r, r.identifier AS identifier
WHERE identifier STARTS WITH "RI II,"
WITH
	r,
	"RI02" AS department,
	CASE
		WHEN identifier STARTS WITH "RI II,5" THEN "Papstregesten"
		ELSE "1-4"
	END AS volume
SET r.volume = volume, r.department = department
MERGE (c1:Collection {type: "department", label: department})
MERGE (c2:Collection {type: "volume", label: volume})-[:PART_OF]->(c1)
MERGE (r)-[:PART_OF]->(c2);

 
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
MERGE (c1:Collection {type: "department", label: department})
MERGE (c2:Collection {type: "volume", label: volume})-[:PART_OF]->(c1)
MERGE (r)-[:PART_OF]->(c2);


//RI IV
MATCH (r:Regesta) WITH r, r.identifier AS identifier
WHERE identifier STARTS WITH "RI IV,"
WITH
	r,
	"RI04" AS department,
	CASE
		WHEN identifier STARTS WITH "RI IV,1,1" THEN "Lothar III."
		WHEN identifier STARTS WITH "RI IV,1,2" THEN "Konrad III."
		WHEN identifier STARTS WITH "RI IV,2," THEN "Friedrich I."
		WHEN identifier STARTS WITH "RI IV,3" THEN "Heinrich VI."
		WHEN identifier STARTS WITH "RI IV,4,4," THEN "Papstregesten"
		ELSE split(split(identifier, ' n. ')[0], 'RI IV,')[1]
	END AS volume
SET r.volume = volume, r.department = department
MERGE (c1:Collection {type: "department", label: department})
MERGE (c2:Collection {type: "volume", label: volume})-[:PART_OF]->(c1)
MERGE (r)-[:PART_OF]->(c2);


//RI V
MATCH (r:Regesta) WITH r, r.identifier AS identifier
WHERE identifier STARTS WITH "RI V,"
WITH
	r,
	"RI05" AS department,
	CASE
		WHEN identifier STARTS WITH "RI V,1," THEN "Staufer, Kaiser und Könige"
		WHEN identifier STARTS WITH "RI V,2," THEN "Staufer, Päpste und Reichssachen"
		WHEN identifier STARTS WITH "RI V,4,6" THEN "Staufer, Nachträge und Ergänzungen"
		WHEN identifier STARTS WITH "RI V,4,1" THEN "Staufer, Nachträge zu Friedrich II. und Konrad IV."
		ELSE split(split(identifier, ' n. ')[0], 'RI V,')[1]
	END AS volume
SET r.volume = volume, r.department = department
MERGE (c1:Collection {type: "department", label: department})
MERGE (c2:Collection {type: "volume", label: volume})-[:PART_OF]->(c1)
MERGE (r)-[:PART_OF]->(c2);


//RI VI
MATCH (r:Regesta) WITH r, r.identifier AS identifier
WHERE identifier STARTS WITH "RI VI,"
WITH
	r,
	"RI06" AS department,
	CASE
		WHEN identifier STARTS WITH "RI VI,4," THEN "4"
		WHEN identifier STARTS WITH "RI VI,1" THEN "1"
		WHEN identifier STARTS WITH "RI VI,2" THEN "2"
		ELSE split(split(identifier, ' n. ')[0], 'RI V,')[1]
	END AS volume
SET r.volume = volume, r.department = department
MERGE (c1:Collection {type: "department", label: department})
MERGE (c2:Collection {type: "volume", label: volume})-[:PART_OF]->(c1)
MERGE (r)-[:PART_OF]->(c2);
MATCH (r:Regesta)
WHERE r.identifier starts with "[RIplus] Regg. Heinrich VII. n. "
WITH r, r.identifier as identifier,
"RI06" AS department,
"[RIplus] Regg. Heinrich VII." AS volume
MATCH (c1:Collection {type: "department", label: "RI06"})
SET r.volume = volume, r.department = department
WITH r, volume, department, c1
//MERGE (c1:Collection {type: "department", label: department})
MERGE (c2:Collection {type: "volume", label: volume})
MERGE (c2)-[:PART_OF]->(c1)
MERGE (r)-[:PART_OF]->(c2);


  
// [Regesta Habsburgica 3]
MATCH (r:Regesta) WITH r, r.identifier AS identifier
WHERE identifier STARTS WITH "[Regesta Habsburgica 3]"
WITH
	r,
	"Regesta Habsburgica 3" AS department,
	CASE
		WHEN identifier STARTS WITH "[Regesta Habsburgica 3]" THEN "Friedrich der Schöne"
		ELSE split(split(identifier, ' n. ')[0], '[Regesta Habsburgica 3]')[1]
	END AS volume
SET r.volume = volume, r.department = department
MERGE (c1:Collection {type: "department", label: department})
MERGE (c2:Collection {type: "volume", label: volume})-[:PART_OF]->(c1)
MERGE (r)-[:PART_OF]->(c2);


//RI VII Department
MATCH (r:Regesta) WITH r, r.identifier AS identifier
WHERE identifier STARTS WITH "[RI VII]"
	WITH r,
	"RI07" AS department,
	split(split(identifier, ' n. ')[0], ' H. ')[1] AS volume
SET r.volume = volume, r.department = department
MERGE (c1:Collection {type: "department", label: department})

MERGE (c2:Collection {type: "volume", label: volume})-[:PART_OF]->(c1)

MERGE (r)-[:PART_OF]->(c2);



//RI VIII Department

MATCH (r:Regesta) WITH r, r.identifier AS identifier
WHERE identifier STARTS WITH "RI VIII"
	WITH
	r,
	"RI08" AS department,
	"1" AS volume
SET r.volume = volume, r.department = department
MERGE (c1:Collection {type: "department", label: department})

MERGE (c2:Collection {type: "volume", label: volume})-[:PART_OF]->(c1)

MERGE (r)-[:PART_OF]->(c2);

//Regg. Pfalzgrafen 2 Department
MATCH (r:Regesta) WITH r, r.identifier AS identifier
WHERE identifier STARTS WITH "[Regg. Pfalzgrafen 2]"
WITH
	r,
	"Pfalzgrafen 2" AS department,
	"1" AS volume
SET r.volume = volume, r.department = department
MERGE (c1:Collection {type: "department", label: department})

MERGE (c2:Collection {type: "volume", label: volume})-[:PART_OF]->(c1)

MERGE (r)-[:PART_OF]->(c2);

//RI XI Department
MATCH (r:Regesta) WITH r, r.identifier AS identifier
WHERE identifier STARTS WITH "RI XI,"
WITH
	r,
	"RI11" AS department,
	"1" AS volume
	SET r.volume = volume, r.department = department
MERGE (c1:Collection {type: "department", label: department})
MERGE (c2:Collection {type: "volume", label: volume})-[:PART_OF]->(c1)
MERGE (r)-[:PART_OF]->(c2);

MATCH (r:Regesta) WITH r, r.identifier AS identifier
WHERE identifier STARTS WITH "RI XI Neubearb"
WITH
	r,
	"RI11" AS department,
	split(split(identifier, ' n. ')[0], 'RI XI ')[1] AS volume
SET r.volume = volume, r.department = department
MERGE (c1:Collection {type: "department", label: department})
MERGE (c2:Collection {type: "volume", label: volume})-[:PART_OF]->(c1)
MERGE (r)-[:PART_OF]->(c2);


//RI XII Department
MATCH (r:Regesta) WITH r, r.identifier AS identifier
WHERE identifier STARTS WITH "RI XI "
WITH
	r,
	"RI12" AS department,
	"1" AS volume
	SET r.volume = volume, r.department = department
MERGE (c1:Collection {type: "department", label: department})
MERGE (c2:Collection {type: "volume", label: volume})-[:PART_OF]->(c1)
MERGE (r)-[:PART_OF]->(c2);



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
MERGE (c1:Collection {type: "department", label: department})
MERGE (c2:Collection {type: "volume", label: volume})-[:PART_OF]->(c1)
MERGE (r)-[:PART_OF]->(c2);
MATCH (r:Regesta) WITH r, r.identifier AS identifier
WHERE identifier STARTS WITH "Chmel"
WITH
	r,
	"RI13" AS department,
	"Chmel" AS volume
SET r.volume = volume, r.department = department
MERGE (c1:Collection {type: "department", label: department})
MERGE (c2:Collection {type: "volume", label: volume})-[:PART_OF]->(c1)
MERGE (r)-[:PART_OF]->(c2);

						 
						 
//RI XIV
MATCH (r:Regesta) WITH r, r.identifier AS identifier
WHERE identifier STARTS WITH "RI XIV,"
WITH
	r,
	"RI14" AS department,
	"Maximilian I." AS volume
SET r.volume = volume, r.department = department
MERGE (c1:Collection {type: "department", label: department})
MERGE (c2:Collection {type: "volume", label: volume})-[:PART_OF]->(c1)
MERGE (r)-[:PART_OF]->(c2);


// Collections uuids geben
MATCH (n:Collection) 
WHERE n.uuid IS NULL
SET n.uuid = randomUUID();

// Annotationsknoten erstellen aus APPEARS_IN-Kanten 
Match (r:Regesta)<-[rel:APPEARS_IN]-(e:Entity) 
create (a:Annotation {riType: "role", riRole: rel.type}) 
create (r)-[:HAS_ANNOTATION]->(a)-[:REFERS_TO]->(e) 
detach delete rel;

// Create property connection between Entities
MATCH (n:Entity)-[rel]->(n2:Entity)
MERGE (n)-[:HAS_PROPERTY]->(:Property { type: TYPE(rel), uuid: randomUUID(), namespace: "" })-[:REFERS_TO]->(n2)
DELETE rel;


// Entities bekommen department und volume property (old)
//MATCH (n:Entity)<-[:REFERS_TO]-(a:Annotation)<-[:HAS_ANNOTATION]-(r:Regesta)
//OPTIONAL MATCH (n)-[rel1]->(n2:Entity)
//OPTIONAL MATCH (n)-[:HAS_PROPERTY]->(:Property)-[:REFERS_TO]->(n3:Entity)
//OPTIONAL MATCH (n4:Entity) WHERE n4.parentId = n.xmlId
//SET n.volume = r.volume, n.department = r.department
//SET n2.volume = r.volume, n2.department = r.department
//SET n3.volume = r.volume, n3.department = r.department
//SET n4.volume = r.volume, n4.department = r.department;


// Entities bekommen department und volume property (beides listen)
MATCH (n:Entity)<-[:REFERS_TO]-(a:Annotation)<-[:HAS_ANNOTATION]-(r:Regesta)
OPTIONAL MATCH (n)-[rel1]->(n2:Entity)
OPTIONAL MATCH (n)-[:HAS_PROPERTY]->(:Property)-[:REFERS_TO]->(n3:Entity)
OPTIONAL MATCH (n4:Entity) WHERE n4.parentId = n.xmlId
WITH n, n2, n3, n4, r,
	apoc.coll.toSet(apoc.coll.removeAll(coalesce(n.volume, []) + [r.volume], [null])) AS newVolume,
	apoc.coll.toSet(apoc.coll.removeAll(coalesce(n.department, []) + [r.department], [null])) AS newDepartment
	
SET n.volume = CASE WHEN size(newVolume) > 0 THEN newVolume ELSE NULL END,
n.department = CASE WHEN size(newDepartment) > 0 THEN newDepartment ELSE NULL END
SET n2.volume = CASE WHEN size(newVolume) > 0 THEN newVolume ELSE NULL END,
n2.department = CASE WHEN size(newDepartment) > 0 THEN newDepartment ELSE NULL END
SET n3.volume = CASE WHEN size(newVolume) > 0 THEN newVolume ELSE NULL END,
n3.department = CASE WHEN size(newDepartment) > 0 THEN newDepartment ELSE NULL END
SET n4.volume = CASE WHEN size(newVolume) > 0 THEN newVolume ELSE NULL END,
n4.department = CASE WHEN size(newDepartment) > 0 THEN newDepartment ELSE NULL END;


// Entities bekommen uuids
MATCH (n:Entity) 
WHERE n.uuid IS NULL
SET n.uuid = randomUUID();


// Adds the Regesta to the collection model
MATCH (n:Regesta) SET n:Collection, n.type = "regesta";

// Deletes any direct connection between entities and regesta
MATCH (n:Entity)-[rel]->(r:Regesta) delete rel;

// numberOfChilds wird ermittelt
MATCH (n:Entity)
OPTIONAL MATCH (n)<-[:REFERS_TO]-(:Property {type: "IS_SUB_OF"})<-[:HAS_PROPERTY]-(sub:Entity)
WITH n, COLLECT(sub) AS directChildren
CALL {
	WITH n
	MATCH (n)<-[:REFERS_TO]-(:Property {type: "IS_SUB_OF"})<-[:HAS_PROPERTY]-(sub:Entity)
	CALL apoc.path.subgraphNodes(sub, {
		relationshipFilter: "HAS_PROPERTY>|<REFERS_TO",
		minLevel: 1
	}) YIELD node
	RETURN COUNT(DISTINCT node) AS totalSubEntities
}
SET n.numberOfChilds = totalSubEntities



MATCH (c:Collection)
WITH c, [
	"title",
	"identifier",
	"date",
	"placeOfIssue",
	"summary",
	"originalDate",
	"versoNote",
	"witnesses",
	"incipit",
	"clerk",
	"recipient",
	"chancellor",
	"archivalHistory",
	"literature",
	"commentary",
	"annotations",
	"footnotes"
] AS textProperties
UNWIND textProperties AS prop
WITH c, prop, c[prop] AS textValue

WHERE textValue IS NOT NULL
CREATE (t:Text { text: textValue, label: prop })
MERGE (t)-[:PART_OF]->(c)
WITH c, COLLECT(t) AS textNodes
WHERE SIZE(textNodes) > 1

UNWIND RANGE(0, SIZE(textNodes)-2) AS i
WITH textNodes[i] AS current, textNodes[i+1] AS next
MERGE (current)-[:NEXT]->(next);

// ATAG Chains
:auto MATCH (r:Text)
WHERE r.text is not null AND NOT((r)-[:NEXT_TOKEN]->())
CALL {
WITH r
CALL atag.chains.fullChain(r, "text")
} IN TRANSACTIONS OF 1 ROWS
RETURN count(*);

CREATE INDEX token_uuid IF NOT EXISTS
FOR (n:Token) ON (n.uuid);
CREATE INDEX character_uuid IF NOT EXISTS
FOR (n:Character) ON (n.uuid);
CREATE INDEX text_uuid IF NOT EXISTS
FOR (n:Text) ON (n.uuid);

:auto MATCH (t:Text)
WHERE t.uuid IS NULL
CALL {
WITH t
	SET t.uuid = randomUUID()
	} IN TRANSACTIONS OF 300 ROWS
RETURN count(*);

:auto MATCH (c:Character)
WHERE c.uuid IS NULL
CALL {
	WITH c
	SET c.uuid = randomUUID()
} IN TRANSACTIONS OF 300 ROWS
RETURN count(*);

:auto MATCH (tok:Token)
WHERE tok.uuid IS NULL
CALL {
	WITH tok
	SET tok.uuid = randomUUID()
} IN TRANSACTIONS OF 300 ROWS
RETURN count(*);





MATCH (t:Text)-[:NEXT_TOKEN]->(s:Token)
WITH t, s LIMIT 1 // temp limit
MATCH path = (s)-[:NEXT_TOKEN*]->(e:Token)
WITH t, s, e, nodes(path) AS tokens

// Berechnung der HTML/XML-Token-Sequenzen mit reduce()
WITH t, s, e, tokens,
range(0, size(tokens)-1) AS indices

WITH t, s, e, tokens,
reduce(htmlTokens = [], i IN indices |
CASE
	WHEN tokens[i].text = '<' AND i+1 < size(tokens)
		AND tokens[i+1].text IN ['a', 'b', 'sup', 'span', 'li', 'ol', 'em', 'link']
		AND i+2 < size(tokens) AND tokens[i+2].text = '>'
	THEN htmlTokens + [tokens[i], tokens[i+1], tokens[i+2]] // Öffnendes Tag <tag>
	WHEN tokens[i].text = '<' AND i+1 < size(tokens) AND tokens[i+1].text = '/'
		AND i+2 < size(tokens)
		AND tokens[i+2].text IN ['a', 'b', 'sup', 'span', 'li', 'ol', 'em', 'link']
		AND i+3 < size(tokens) AND tokens[i+3].text = '>'
	THEN htmlTokens + [tokens[i], tokens[i+1], tokens[i+2], tokens[i+3]] // Schließendes Tag </tag>
	ELSE htmlTokens
END
) AS htmlTokenSequences

// Erstelle die Liste der Tokens, die als replacements übernommen werden (keine HTML-Tags)
WITH t, s, e,
htmlTokenSequences,
[token IN tokens WHERE NOT token IN htmlTokenSequences] AS replacements

// Update die Chain ohne html/xml elemente
CALL atag.chains.update(t.uuid, s.uuid, e.uuid, replacements, {
textLabel: "Text",
elementLabel: "Token",
relationshipType: "NEXT_TOKEN"
}) YIELD path

// Finde die zugehörigen Character-Nodes für die ersetzten Tokens
WITH path, htmlTokenSequences, t
UNWIND htmlTokenSequences AS r
MATCH (tk:Token {uuid: r.uuid }),
(tk)-[:TOKEN_START]->(c1:Character),
(tk)-[:TOKEN_END]->(c2:Character)

// Aktualisiere auch die Characters entsprechend
CALL atag.chains.update(t.uuid, c1.uuid, c2.uuid, [], {
textLabel: "Text",
elementLabel: "Character",
relationshipType: "NEXT_CHARACTER"
}) YIELD path AS updatedCharacterPath

RETURN path, updatedCharacterPath;




// Alternative
  

MATCH (t:Text {uuid: "c2266482-05e0-43f7-96d2-040d3cdbec4b"})-[:NEXT_TOKEN]->(s:Token)

WITH t, s

MATCH path = (s)-[:NEXT_TOKEN*]->(e:Token)

WITH t, s, e, nodes(path) AS tokens

  

// Berechnung der HTML/XML-Token-Sequenzen mit reduce()

WITH t, s, e, tokens,

range(0, size(tokens)-1) AS indices

  

WITH t, s, e, tokens,

reduce(htmlTokens = [], i IN indices |

CASE

WHEN tokens[i].text = '<' AND i+1 < size(tokens)

AND tokens[i+1].text IN ['a', 'b', 'sup', 'span', 'li', 'ol', 'em', 'link']

AND i+2 < size(tokens) AND tokens[i+2].text = '>'

THEN htmlTokens + [tokens[i], tokens[i+1], tokens[i+2]] // Öffnendes Tag <tag>

WHEN tokens[i].text = '<' AND i+1 < size(tokens) AND tokens[i+1].text = '/'

AND i+2 < size(tokens)

AND tokens[i+2].text IN ['a', 'b', 'sup', 'span', 'li', 'ol', 'em', 'link']

AND i+3 < size(tokens) AND tokens[i+3].text = '>'

THEN htmlTokens + [tokens[i], tokens[i+1], tokens[i+2], tokens[i+3]] // Schließendes Tag </tag>

ELSE htmlTokens

END

) AS htmlTokenSequences

  

// **Fix: Flache Liste von `htmlTokenSequences` mit Maps erzeugen**

UNWIND htmlTokenSequences AS htmlToken

WITH t, s, e, COLLECT(DISTINCT { uuid: htmlToken.uuid }) AS htmlTokenSequences, tokens

  

// **Fix: Korrekte Map-Struktur für `replacements`**

WITH t, s, e, htmlTokenSequences,

[token IN tokens WHERE NOT token IN htmlTokenSequences | { uuid: token.uuid, text: token.text }] AS replacements

  

// Update die Chain ohne html/xml elemente

CALL atag.chains.update(t.uuid, s.uuid, e.uuid, replacements, {

textLabel: "Text",

elementLabel: "Token",

relationshipType: "NEXT_TOKEN"

}) YIELD path

  

// Finde die zugehörigen Character-Nodes für die ersetzten Tokens

WITH path, htmlTokenSequences, t

UNWIND htmlTokenSequences AS r

MATCH (tk:Token {uuid: r.uuid }),

(tk)-[:TOKEN_START]->(c1:Character),

(tk)-[:TOKEN_END]->(c2:Character)

  

// Aktualisiere auch die Characters entsprechend

CALL atag.chains.update(t.uuid, c1.uuid, c2.uuid, [], {

textLabel: "Text",

elementLabel: "Character",

relationshipType: "NEXT_CHARACTER"

}) YIELD path AS updatedCharacterPath

  

RETURN path, updatedCharacterPath;
```