### 查询一个节点
`MATCH (aa: Person) WHERE ee.name = 'Emil' RETURN aa;`
* MATCH - specifies a pettern of nodes and relationshipts.
* (aa: Person) - is a single node pattern with label Person. it assigns match to the variable aa. aa can be any variable name.
* WHERE - filters the query.
* ee.name = 'Emil' - compares name property to the value Emil.
* RETURN - returns paticular results.

### 查询一个节点的一级关系节点
```
MATCH (ee:Person)-[:KNOWS]-(friends)
WHERE ee.name = 'Emil' RETURN ee, friends
```
* MATCH - describes what nodes will be retrieved based upon the pattern.
* (ee) - s the node reference that will be returned based upon the WHERE clause.
* `-[:KNOWS]-` - matches the KNOWS relationships (in either direction) from ee.
* `(friends)` - represents the nodes that are Emil's friends.
* `RETURN` - returns the node, referenced here by (ee), and the related (friends) nodes found.

### 查询全部
`MATCH (n) RETURN n`
* (n) - 不加限制

### 使用别名
```
WITH TomH as a
MATCH (a)-[:ACTED_IN]->(m)<-[:DIRECTED]-(d) RETURN a,m,d LIMIT 10;
```
* `WITH TomH as a` - 使用别名
* `LIMIT` - 限定数量

### 查询指定属性
查询指定标签下节点的指定属性，并限定数量
`MATCH (people:Person) RETURN people.name LIMIT 10`

### 使用条件限定
`MATCH (nineties:Movie) WHERE nineties.released >= 1990 AND nineties.released < 2000 RETURN nineties.title`

### 使用关系
`MATCH (tom:Person {name: "Tom Hanks"})-[:ACTED_IN]->(tomHanksMovies) RETURN tom,tomHanksMovies`
`MATCH (cloudAtlas:Movie {title: "Cloud Atlas"})<-[:DIRECTED]-(directors) RETURN directors.name`
#### DISTINCT
`MATCH (tom:Person {name:"Tom Hanks"})-[:ACTED_IN]->(m)<-[:ACTED_IN]-(coActors) RETURN DISTINCT coActors.name`
#### relatedTo
`MATCH (people:Person)-[relatedTo]-(:Movie {title: "Cloud Atlas"}) RETURN people.name, Type(relatedTo), relatedTo.roles`

### 查询n-hops范围内节点
Use variable length patterns to find movies and actors up to 4 "hops" away from Kevin Bacon.
```
MATCH (bacon:Person {name:"Kevin Bacon"})-[*1..4]-(hollywood)
RETURN DISTINCT hollywood
```
### 查找节点间最短距离
Use the built-in shortestPath() algorithm to find the "Bacon Path" to Meg Ryan.
```
MATCH p=shortestPath(
(bacon:Person {name:"Kevin Bacon"})-[*]-(meg:Person {name:"Meg Ryan"})
)
RETURN p
```

### 推荐
Extend Tom Hanks co-actors to find co-co-actors who have nоt worked with Tom Hanks.
```
MATCH (tom:Person {name:"Tom Hanks"})-[:ACTED_IN]->(m)<-[:ACTED_IN]-(coActors),
    (coActors)-[:ACTED_IN]->(m2)<-[:ACTED_IN]-(cocoActors)
  WHERE NOT (tom)-[:ACTED_IN]->()<-[:ACTED_IN]-(cocoActors) AND tom <> cocoActors
  RETURN cocoActors.name AS Recommended, count(*) AS Strength ORDER BY Strength DESC
```

Find someone who can introduce Tom Hanks to his potential co-actor, in this case Tom Cruise.
```
MATCH (tom:Person {name:"Tom Hanks"})-[:ACTED_IN]->(m)<-[:ACTED_IN]-(coActors),
  (coActors)-[:ACTED_IN]->(m2)<-[:ACTED_IN]-(cruise:Person {name:"Tom Cruise"})
RETURN tom, m, coActors, m2, cruise
```

### 查询一对多关系
What categories of food does each supplier supply?
```
MATCH (s:Supplier)-->(:Product)-->(c:Category)
RETURN s.companyName as Company, collect(distinct c.categoryName) as Categories
```
* `()-->()-->()` - 查询匹配的所有关系对
* `as Company` - 为选择属性生成输出列表中的字段名
* `distinct xxx` - 去除
* `collect()` - 汇总在一个列表中

### 查询产品供应商
```
MATCH (c:Category {categoryName:"Produce"})<--(:Product)<--(s:Supplier)
RETURN DISTINCT s.companyName as ProduceSuppliers
```

### 查询消费者购买商品总量
查询购买了指定类型商品的消费者列表，并计算每个消费者订单总量
```
MATCH (cust:Customer)-[:PURCHASED]->(:Order)-[o:ORDERS]->(p:Product),
  (p)-[:PART_OF]->(c:Category {categoryName:"Produce"})
RETURN DISTINCT cust.contactName as CustomerName, SUM(o.quantity) AS TotalProductsPurchased
```
