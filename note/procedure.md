## 入口
- 跳过之前的客户端服务端通信，第一条query执行语句：src/sql/engine/mod.rs::line 74
  ```rust
    Parser::new(query).parse()?
  ```
  于是进一步到parser内：

## Lexer
parser直接调用了Lexer产生的迭代器，将query str传入Lexer中，转换为内部变量（token）。
Lexer是一个peekable的Char迭代器。（生命周期？）.
值得注意的是aggregate function没有被作为keyword，而是直接转化成了function expr。
lexer核心是scan方法，peek match之后调用子scan方法，内部就是一般的dfs。


## Parser
struct Parser with lifetime?
```rust
pub struct Parser<'a> {
    lexer: std::iter::Peekable<Lexer<'a>>,
}
```
new with query string, 一个peekable的迭代器（todo） and call parse to generate ast;  
main function: `parse_statement`, peek 并指向不同的parse方法。  
Parser是top-down的，每一个字句都是一个 statement + ';'
- 几个基本的工具方法
  - `next`：通过lexer的next方法获取下一个token
  - `next_expect`：比较期望的token是否和下一个lexer返回的token一致
  - `next_if`, `next_if_xxx`, `next_ident`: 下一个token/ident是否是期望的，是则返回。

### CREATE
以最简单的create为例  
  STATEMENT:: CREATE TABLE ident '(' 'name_ident' TYPE CONSTRAINT ')',  
- parse_statement: peek and match `CREATE`
- -> parse_ddl(match `TABLE`)
- ---> next_ident: consume ident and save name
- ---> consume OpenParen
- ---> loop till comma, parse column and push into column vec
- -----> column 直接就是ast了，保存名称、类型，通过布尔值或者设置基本属性
```sql
CREATE TABLE genres (
        id INTEGER PRIMARY KEY,
        name STRING NOT NULL
        );
```
被转换成了
```json
"CreateTable":{
  "name": "genres",
  "columns": [
    "Column": { "name": "id", "datatype": Integer, primary_key: true, nullable: None, default: None, unique: false, index: false, references: None },
    "Column": { "name": "name", "datatype": String, primary_key: false, nullable: Some(false), default: None, unique: false, index: false, references: None }
  ]
}
```

## 入口2
回到一开始的入口，对于事务操作（begin，commit，rollback等），调用对应的事务方法；对于读写操作，绑定已有事务或新建事务并调用plan.optimize.execute。
