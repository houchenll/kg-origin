### 创建一个node
`CREATE (ee: Person {name: 'Emil', from: 'Sweden', kloutScore: 99})`
* CREATE: create a node
* (): indicate the node
* ee: Person - ee is the node variable and Person is the node label
* {}: contains the properties that describe the node.

### 一次创建多个node
```
MATCH (ee:Person) WHERE ee.name = 'Emil'
CREATE (js:Person { name: 'Johan', from: 'Sweden', learn: 'surfing' }),
(ir:Person { name: 'Ian', from: 'England', title: 'author' }),
(rvb:Person { name: 'Rik', from: 'Belgium', pet: 'Orval' }),
(ally:Person { name: 'Allison', from: 'California', hobby: 'surfing' }),
(ee)-[:KNOWS {since: 2001}]->(js),(ee)-[:KNOWS {rating: 5}]->(ir),
(js)-[:KNOWS]->(ir),(js)-[:KNOWS]->(rvb),
(ir)-[:KNOWS]->(js),(ir)-[:KNOWS]->(ally),
(rvb)-[:KNOWS]->(ally)
```
* CREATE - 除了创建node外，还可以创建关系
* 多个node和关系之间用 , 分隔
* 一个创建语句中，node 的 variable 不能重复
* 关系有方向 () -[:RelationType {properties}]-> ()
* 关系中的节点使用 ()，内部使用 node variable
* 关系类型前加冒号 :
* 关系可有0到多个属性
* CREATE 中使用已有节点 ee 之前，需要先查询出来，否则ee相关创建无效
* 重复执行创建语句，会重复创建节点和它们之间的关系

