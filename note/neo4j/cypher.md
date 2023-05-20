## 是什么
Neo4j's graph query language

### 特点
* Uses patterns to describe graph data.
* Familiar SQL-like clauses.
* Declarative, describing what to find, not how to find it.

## 创建

### 创建节点

在 `neo4j$` 后的输入框中输入下面命令，创建一个节点

```
CREATE (ee:Person {name: 'Emil', from: 'Sweden', kloutScore: 99})
```

* CREATE creates the node
* () indicates the node
* ee:Person – ee is the node variable and Person is the node label
* {} contains the properties that describe the node

可以一次创建多个节点

```
CREATE (js:Person { name: 'Johan', from: 'Sweden', learn: 'surfing' }),
(ir:Person { name: 'Ian', from: 'England', title: 'author' }),
(rvb:Person { name: 'Rik', from: 'Belgium', pet: 'Orval' }),
(ally:Person { name: 'Allison', from: 'California', hobby: 'surfing' })
```

### 创建关系

```
CREATE (ee)-[:KNOWS {since: 2001}]->(js),(ee)-[:KNOWS {rating: 5}]->(ir),
(js)-[:KNOWS]->(ir),(js)-[:KNOWS]->(rvb),
(ir)-[:KNOWS]->(js),(ir)-[:KNOWS]->(ally),
(rvb)-[:KNOWS]->(ally)
```

### 创建约束
Create unique node property constraints to ensure that property values are unique for all nodes with a specific label. Adding the unique constraint, implicitly adds an index on that property.

```
CREATE CONSTRAINT FOR (n:Movie) REQUIRE (n.title) IS UNIQUE
CREATE CONSTRAINT FOR (n:Person) REQUIRE (n.name) IS UNIQUE
```

### 创建索引 Index nodes
Create indexes on one or more properties for all nodes that have a given label. Indexes are used to increase search performance.

`CREATE INDEX FOR (m:Movie) ON (m.released)`


## 匹配|查找

### 匹配节点

```
MATCH (ee:Person) WHERE ee.name = 'Emil' RETURN ee;
```

* MATCH specifies a pattern of nodes and relationships.
* (ee:Person) is a single node pattern with label Person. It assigns matches to the variable ee.
* WHERE filters the query.
* ee.name = 'Emil' compares name property to the value Emil.
* RETURN returns particular results.

### 匹配关系
```
MATCH (ee:Person)-[:KNOWS]-(friends)
WHERE ee.name = 'Emil' RETURN ee, friends
```

* MATCH describes what nodes will be retrieved based upon the pattern.
* (ee) is the node reference that will be returned based upon the WHERE clause.
* -[:KNOWS]- matches the KNOWS relationships (in either direction) from ee.
* (friends) represents the nodes that are Emil's friends.
* RETURN returns the node, referenced here by (ee), and the related (friends) nodes found.

### Find individual nodes
* Find the actor named "Tom Hanks":
`MATCH (tom:Person {name: "Tom Hanks"}) RETURN tom`

* Find the movie with title "Cloud Atlas"
`MATCH (cloudAtlas:Movie {title: "Cloud Atlas"}) RETURN cloudAtlas`

* Find 10 people and return their names
`MATCH (people:Person) RETURN people.name LIMIT 10`

* Find movies released in the 1990s and return their titles
`MATCH (nineties:Movie) WHERE nineties.released >= 1990 AND nineties.released < 2000 RETURN nineties.title`

### Query patterns
Use the type of the relationship to find patterns within the graph, for example, ACTED_IN or DIRECTED.

* What movies did Tom Hanks act in?
`MATCH (tom:Person {name: "Tom Hanks"})-[:ACTED_IN]->(tomHanksMovies) RETURN tom,tomHanksMovies`

* Who directed "Cloud Atlas"?
`MATCH (cloudAtlas:Movie {title: "Cloud Atlas"})<-[:DIRECTED]-(directors) RETURN directors.name`

* Who were Tom Hanks' co-actors?
`MATCH (tom:Person {name:"Tom Hanks"})-[:ACTED_IN]->(m)<-[:ACTED_IN]-(coActors) RETURN DISTINCT coActors.name`

* How people are related to "Cloud Atlas"?
`MATCH (people:Person)-[relatedTo]-(:Movie {title: "Cloud Atlas"}) RETURN people.name, Type(relatedTo), relatedTo.roles`

### Bacon Path
the shortest path between two nodes

* Use variable length patterns to find movies and actors up to 4 "hops" away from Kevin Bacon.
`MATCH (bacon:Person {name:"Kevin Bacon"})-[*1..4]-(hollywood)
RETURN DISTINCT hollywood`

* Use the built-in shortestPath() algorithm to find the "Bacon Path" to Meg Ryan.
`MATCH p=shortestPath(
(bacon:Person {name:"Kevin Bacon"})-[*]-(meg:Person {name:"Meg Ryan"})
)
RETURN p`

### Recommend
Recommend new co-actors
Let's recommend new co-actors for Tom Hanks. A basic recommendation approach is to find connections past an immediate neighborhood that are themselves well connected.
1. Extend Tom Hanks co-actors to find co-co-actors who have nоt worked with Tom Hanks.
```
MATCH (tom:Person {name:"Tom Hanks"})-[:ACTED_IN]->(m)<-[:ACTED_IN]-(coActors),
    (coActors)-[:ACTED_IN]->(m2)<-[:ACTED_IN]-(cocoActors)
  WHERE NOT (tom)-[:ACTED_IN]->()<-[:ACTED_IN]-(cocoActors) AND tom <> cocoActors
  RETURN cocoActors.name AS Recommended, count(*) AS Strength ORDER BY Strength DESC
```

2. Find someone who can introduce Tom Hanks to his potential co-actor, in this case Tom Cruise
```
MATCH (tom:Person {name:"Tom Hanks"})-[:ACTED_IN]->(m)<-[:ACTED_IN]-(coActors),
  (coActors)-[:ACTED_IN]->(m2)<-[:ACTED_IN]-(cruise:Person {name:"Tom Cruise"})
RETURN tom, m, coActors, m2, cruise
```


## 清空
NOTE: Nodes cannot be deleted if they have relationships, so you need to detach the nodes to delete them.
1. Delete all Movie and Person nodes, and their relationships.
`MATCH (n) DETACH DELETE n`
2. Verify that the Movie Graph has been removed.
`MATCH (n) RETURN n`
