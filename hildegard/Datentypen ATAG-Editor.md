
### ATAG JSON Schema

Das sind die möglichen Datentypen für Knoten-Properties (z. B. Annotationen, Collections). Die Tabelle orientiert sich an der Data types-Tabelle von JSON-Schema: https://json-schema.org/understanding-json-schema/reference/type. Die Typen `object`, `regex` und `null` werden erstmal rausgelassen. Dafür kommt ein neuer `point`-Datentyp für Punkte in einem Koordinatensystem dazu. Außerdem werden manche specific keywords weggelassen, die nicht verwendet werden sollen (z. B. `additionalItems` bei `array`) 

| Type                                                                                                                                                       | Specific Keywords                                                          | JavaScript Type | Cypher Data type | Description                                                                                                 |
| ---------------------------------------------------------------------------------------------------------------------------------------------------------- | -------------------------------------------------------------------------- | --------------- | ---------------- | ----------------------------------------------------------------------------------------------------------- |
| [`array`](https://json-schema.org/understanding-json-schema/reference/array)                                                                               | `items`, `minItems`, `maxItems`, `uniqueItems`                             | `Array<T>`      | `LIST<T>`        | Define item schemas, additional item handling, item count constraints, and uniqueness.                      |
| [`number`](https://json-schema.org/understanding-json-schema/reference/numeric)                                                                            | `minimum`, `maximum`, `exclusiveMinimum`, `exclusiveMaximum`, `multipleOf` |                 |                  | Define numeric ranges, including exclusive bounds and divisibility.                                         |
| [`string`](https://json-schema.org/un[string](https://json-schema.org/understanding-json-schema/reference/string)derstanding-json-schema/reference/string) | `minLength`, `maxLength`, `pattern`, `format`                              |                 |                  | Restrict string length, pattern matching, and format validation (e.g., email, date).                        |
| `point`                                                                                                                                                    |                                                                            | `Object`        | `POINT`          | Standard Spatial Type von Cypher. Enthält x-, y- und ggf. z-Koordinate sowie Referenz auf Koordinatensystem |
| `boolean`                                                                                                                                                  |                                                                            | `Boolean`       | `BOOLEAN`        | True/false-Wert                                                                                             |
### Datentypen - Ausprägungen 

#### `string`
Einträgen vom typ `string` kann ein zusätzliches `format`-Property mitgegeben werden, das den String näher spezifiziert
Hier gehts fast ausschließlich um Date/Time-Daten, siehe https://neo4j.com/docs/javascript-manual/current/data-types/#_temporal_types dazu.

| `format`-Wert | JavaScript Type                                                                                | Cypher Type      |
| ------------- | ---------------------------------------------------------------------------------------------- | ---------------- |
| `date-time`   | [`LocalDateTime`](https://neo4j.com/docs/javascript-manual/current/data-types/#_localdatetime) | `LOCAL DATETIME` |
| `time`        | [`Time`](https://neo4j.com/docs/javascript-manual/current/data-types/#_time)                   | `LOCAL TIME`     |
| `date`        | [`Date`](https://neo4j.com/docs/javascript-manual/current/data-types/#_date)                   | `DATE`           |
| `duration`    | [`Duration`](https://neo4j.com/docs/javascript-manual/current/data-types/#_duration)           | `DURATION`       |
#### `number`
In der Tabelle oben ist nur von "numeric types" die Rede, aber hier gibt es zwei Ausprägungen: `integer` (für Integer) und `number` (für Floats).

| `type`    | JavaScript Type | Cypher Type |
| --------- | --------------- | ----------- |
| `integer` | `Integer`       | `INTEGER`   |
| `number`  | `Number`        | `FLOAT`     |

#### `array`
Gibt die Möglichkeit an, Listen in Properties zu speichern. In Knoten-Properties können aber nur homogene Listen gespeichert werden, also Listen, in denen alle den gleichen Datentyp haben (https://neo4j.com/docs/cypher-manual/current/values-and-types/property-structural-constructed/). Den Datentyp der Einträge gibt man im `items`-Property an. Macht es ein bisschen einfacher

| Erlaubte `type`'s im `items`-Property | Cypher Data type | Anmerkung                   |
| ------------------------------------- | ---------------- | --------------------------- |
| `integer`                             | `LIST<INTEGER>`  |                             |
| `number`                              | `LIST<FLOAT>`    |                             |
| `string`                              | `LIST<STRING>`   | Siehe `string`-Ausprägungen |
| `point`                               | `LIST<POINT>`    |                             |

### Verwendete Cypher-Datentypen in Projekten

| Knotenlabel | Hildegard                          | Regesta Imperii                            |
| ----------- | ---------------------------------- | ------------------------------------------ |
| Annotation  | - BOOLEAN<br>- INTEGER<br>- STRING | - INTEGER<br>- STRING                      |
| Collection  | - STRING                           | - STRING<br>- DATE<br>- POINT<br>- INTEGER |
| Text        | - STRING                           | - STRING                                   |
| Character   | - STRING                           | - STRING                                   |

### Links
- Datentypen in JSON Schema: https://json-schema.org/understanding-json-schema/reference/type
- Datentypen OpenAPI: https://swagger.io/docs/specification/v3_0/data-models/data-types/
- Mapping Datentypen Neo4j/JavaScript: https://neo4j.com/docs/javascript-manual/current/data-types/