---
layout: post
title: "MySQL 使用笔记(三)"
date: 2020-12-18 08:46:08 +0800
comments: true
categories: [Archives, Web Development]
keywords: MySQL, notes
description: Mysql notes.
---

###1.How do I see all foreign keys to a table or column?
A:For a Table:  

{% codeblock %}
SELECT 
  TABLE_NAME,COLUMN_NAME,CONSTRAINT_NAME, REFERENCED_TABLE_NAME,REFERENCED_COLUMN_NAME
FROM
  INFORMATION_SCHEMA.KEY_COLUMN_USAGE
WHERE
  REFERENCED_TABLE_SCHEMA = '<database>' AND
  REFERENCED_TABLE_NAME = '<table>';
{% endcodeblock %}

For a Column:

{% codeblock %}
SELECT 
  TABLE_NAME,COLUMN_NAME,CONSTRAINT_NAME, REFERENCED_TABLE_NAME,REFERENCED_COLUMN_NAME
FROM
  INFORMATION_SCHEMA.KEY_COLUMN_USAGE
WHERE
  REFERENCED_TABLE_SCHEMA = '<database>' AND
  REFERENCED_TABLE_NAME = '<table>' AND
  REFERENCED_COLUMN_NAME = '<column>';
{% endcodeblock %}

Reference:  

[How do I see all foreign keys to a table or column?](https://stackoverflow.com/questions/201621/how-do-i-see-all-foreign-keys-to-a-table-or-column)  

