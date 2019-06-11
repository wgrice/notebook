# Mybatis入门

## JDBC操作

### 1. JDBC操作流程

```mermaid
graph LR
  start[开始] --> step1[编写SQL]
  step1 --> step2[预编译]
  step2 --> step3[设置参数]
  step3 --> step4[执行SQL]
  step4 --> step5[封装结果]
  step5 --> stop[结束]
```

### 2. JDBC解决方案

1. 工具：JdbcTemplate（Spring） - 考虑不周全，功能不完善
2. 框架：Hibernate、 Mybatis - 整体解决方案 

### 3. 框架比较

1. Hibernate：全自动、全映射ORM ( Obeject realtion mapping ) 框架，旨在消除SQL

   - 长难复杂SQL对于Hibernate不是易事

   - SQL透明，SQL难以优化，对于定制SQL需要额外学习HQL
   - 全映射，Java对象的所有字段都对应表中的列，部分映射相对困难

2. Mybatis：半自动、轻量级ORM框架，SQL编写交给开发人员
   - 解决了Hibernate的SQL难以定制的问题，将SQL语句编写交给开发人员
   - SQL语句写在配置文件中，与Java代码隔离解耦，可以通过修改配置文件的SQL语句进行SQL优化