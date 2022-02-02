## 入口
绑定engine之后，将ast传给planner：
Plan::build -> Planner::build -> Planner::build_statement

针对不同的ast调用不同的分支，返回Node枚举。
以下我们直接进入query部分（ast::Statement::Select)

## Node
plan生成的目标，各种类型的节点

## Query Plan
Parser将Select的几个字句（where、groupby等）直接放在了Select statement的内部，  
这里planner做的事情是按照这些字句直接生成node节点的planner tree。

### From
- `build_from_clause`: 添加对应的scope，将table压入scope的kv中，并不断调用`build_from_item`产生Node树（向右增扩）。  
- -->`build_from_item`:1）若节点是table，叶子为`Node::Scan`节点；2）产生如果字句是Join的话，递归向下。获得left和right node后，生成一个`Node:: NestedLoopJoin`节点；
同时通过`build_expression`,自顶向下构造predicate

### Where
add predicate in where into node

### Select
Inject_hidden expr, add new columns and for planner tree.
