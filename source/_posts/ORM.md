---
title: ORM框架
date: 2021-3-26 23:30:01
categories: 春招复习
tags:
- 2021春招
---

<!--more-->

# ORM框架
* 将Java Bean转换成行记录
* 将行记录转换为Java Bean

# Hibernate
* 通过`Entity`标注实体类，可用`@Table(name="...")`来标识对应的表名。通过`@Column(...)`建立Java Bean字段和数据库表的列的对应关系
* 实体类之间可以有继承关系: 父类使用`@MappedSuperClass`标识
* 提供自动生成的插入/删除/修改(`update`)/查询（`findByExample`/`findByCriteria`/`HQL`，使用类名和属性名，由Hibernate转换成实际的表名和列名）
* ORM如何将对Java Bean的修改反应到数据库中（调用`update`时如何确定哪些字段更新了）
  * 使用代理模式，从ORM读出的对象是代理对象，原对象的每个属性都附加一个标志变量说明这个对象是否被修改。

# MyBatis
* 半自动ORM：可以完成Java Bean到数据行和数据行到Java Bean的双向转换，但是SQL需要用户自行编写。
* 属性名与列名不一致？：添加`ResultMap`（注解或XML）