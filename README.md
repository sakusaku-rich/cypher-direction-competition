# cypher-direction-competition

Algorithm to correct directions in a Cypher query based on the specified schema.

I organized this repository to apply to [this competition](https://github.com/tomasonjo/cypher-direction-competition).

## Requirements

There's no dependency packages.
This algorithm works with standard bundled libraries in Python(>3.10).

## Usage

```python
from QueryCorrector import QueryCorrector, load_schemas

schema = '(Person, KNOWS, Person), (Person, WORKS_AT, Organization)'
schema = load_schemas(schema)
query_corrector = QueryCorrector(schema)

query = 'MATCH (p:Person {id:"Foo"})<-[:WORKS_AT]-(o:Organization) RETURN o.name AS name'
corrected = query_corrector(query)
print(corrected)
```

Output:
```
MATCH (p:Person {id:"Foo"})-[:WORKS_AT]->(o:Organization) RETURN o.name AS name
```

For more details, check out the example notebook.

## Overview of Algorithm

### Sample Data to Show the Algorithm

Query:

```cypher
MATCH
    (p:Person)<-[:WORKS_AT]-(o:Organization)-[:MANAGES]->(c:Country)
WHERE
    c.region = ‘Asia’
    and p.name = ‘Ryosuke Sakurai’
RETURN
  o.name
```

Schema:

| from         | relation   | to           |
| ------------ | ---------- | ------------ |
| Person       | WORKS_AT   | Organization |
| Organization | LOCATED_IN | Country      |
| Person       | MANAGES    | Person       |

### Abstracted Main Processing Steps

1. Extract node variables and detect each labels.
    | Variable | Label            |
    | -------- | ---------------- |
    | p        | ["Person"]       |
    | o        | ["Organization"] |
    | c        | ["Country"]      |

2. Extract paths from the query.[^*]
   - (p:Person)<-[:WORKS_AT]-(o:Organization)-[:MANAGES]->(c:Country)

   [^*]: This example has only one path in a query, but it can be possible that multiple paths be extracted depending on the query.

3. Decompose each extracted paths into paths with Node-Relation-Node pattern.

    - (p:Person)<-[:WORKS_AT]-(o:Organization)-[:MANAGES]->(c:Country)

        - (p:Person)<-[:WORKS_AT]-(o:Organization)
        - (o:Organization)-[:MANAGES]->(c:Country)

4. Verify and correct each Node-Relation-Node patterns with reference to schemas.

    - (p:Person)<-[:WORKS_AT]-(o:Organization)

        There’s no schema that express “Organization WORKS_AT Person” but a schema that express “Person WORKS_AT Organization”.
Then, reverse the direction of relation and make it correct.

    - (o:Organization)-[:MANAGES]->(c:Country)

        There’s no schema that express “Organization MANAGES Country” and vice versa.
Then, the algorithm determines this query as illegal and return empty string.
