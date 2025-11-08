---
layout:     post
title:      "Outer Join 转 Inner Join：理解 NULL 拒绝与查询优化"
date:       2025-11-08 10:00:00
author:     "Hawkingrei"
header-img: "img/post-bg-2015.jpg"
catalog: true
tag: [Database, SQL Optimization] 
---

# 基础知识介绍

在关系型数据库中，Join 操作用于将多个表中的数据根据关联条件组合在一起。常见的 Join 类型包括：

- Inner Join：只返回两个表中匹配的行
- Left Outer Join：返回左表所有行，右表不匹配的显示为 NULL
- Right Outer Join：返回右表所有行，左表不匹配的显示为 NULL
- Full Outer Join：返回两个表的所有行

让我们先创建一些示例数据来演示后续的概念：

```sql
-- 创建部门表
CREATE TABLE departments (
    dept_id INT PRIMARY KEY,
    dept_name VARCHAR(50)
);

-- 创建员工表
CREATE TABLE employees (
    emp_id INT PRIMARY KEY,
    emp_name VARCHAR(50),
    dept_id INT,
    salary DECIMAL(10,2)
);

-- 插入数据
INSERT INTO departments VALUES 
(1, '技术部'),
(2, '销售部'),
(3, '市场部');

INSERT INTO employees VALUES 
(1, '张三', 1, 5000),
(2, '李四', 1, 6000),
(3, '王五', 2, 5500),
(4, '赵六', NULL, 4800);  -- 注意：赵六没有部门
```

# Outer Join 转 Inner Join 与 NULL 拒绝