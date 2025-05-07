### ATAG JSON Schema

Das sind die möglichen Datentypen für Knoten-Properties (z. B. Annotationen, Collections). Die Tabelle orientiert sich grob an der Data types-Tabelle von JSON-Schema: https://json-schema.org/understanding-json-schema/reference/type. 

Für das Mapping zwischen JavaScript- und Cypher-Datentypen siehe https://neo4j.com/docs/javascript-manual/current/data-types/

| Type        | Specific Keywords                                            |
| ----------- | ------------------------------------------------------------ |
| `array`     | `items`, `minItems`, `maxItems`                              |
| `boolean`   |                                                              |
| `date`      |                                                              |
| `date-time` |                                                              |
| `integer`   | `minimum`, `maximum`, `exclusiveMinimum`, `exclusiveMaximum` |
| `number`    | `minimum`, `maximum`, `exclusiveMinimum`, `exclusiveMaximum` |
| `string`    | `minLength`, `maxLength`, `options`, `template`              |
| `time`      |                                                              |
### Verwendete Cypher-Datentypen in Projekten

| Knotenlabel | Hildegard                          | Regesta Imperii                            |
| ----------- | ---------------------------------- | ------------------------------------------ |
| Annotation  | - BOOLEAN<br>- INTEGER<br>- STRING | - INTEGER<br>- STRING                      |
| Collection  | - STRING                           | - STRING<br>- DATE<br>- POINT<br>- INTEGER |
| Text        | - STRING                           | - STRING                                   |
| Character   | - STRING                           | - STRING                                   |

### Komponenten

Jeder Typ bekommt eine eigene Input-Komponente. Falls der Typ `array` ist, wird die Komponente für jeden Eintrag gerendert.

| Type        | Component                                        |
| ----------- | ------------------------------------------------ |
| `boolean`   | [Checkbox](https://primevue.org/checkbox/)       |
| `date`      | [DatePicker](https://primevue.org/datepicker/)   |
| `date-time` | [DatePicker](https://primevue.org/datepicker/)   |
| `integer`   | [InputNumber](https://primevue.org/inputnumber/) |
| `number`    | [InputNumber](https://primevue.org/inputnumber/) |
| `string`    | [InputText](https://primevue.org/inputtext/)     |
| `time`      | [DatePicker](https://primevue.org/datepicker/)   |

### Links
- Datentypen in JSON Schema: https://json-schema.org/understanding-json-schema/reference/type
- Datentypen OpenAPI: https://swagger.io/docs/specification/v3_0/data-models/data-types/
- Mapping Datentypen Neo4j/JavaScript: https://neo4j.com/docs/javascript-manual/current/data-types/
- Mapping Datentypen Neo4j/JavaScript in GraphAcademy-Kurs: https://graphacademy.neo4j.com/courses/app-nodejs/2-interacting/3-type-system/