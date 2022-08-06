## Polymorphic Relationships: A database relationship your college probably won't teach you

## Introduction

Most of the colleges with computer science course have a subject called **Database Management system** or **DBMS** where you are taught topics like **Entity Relationships**, **Normalization**, **Relational Algebra**, **SQL**, etc. Recently, I came across a new type of database relationship called **Polymorphic Relationships** which is very useful in optimizing database schema, but I never knew about it before last week. Lets learn about it in this blog.

## A refresher on Database Relationships

A relationship in database is basically association between two database tables established using a **foreign key**. There are mainly 3 types of database relationships used very heavily:-

### One-to-One

One-to-One relationship is used to to associate two tables in a way such that one record from table 1 is related to only one record in table 2. For example, one user can have only one profile.

| ![o2o.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1651412791257/tWIvgWnTH.png align="center") |
|:--:| 
| *One-to-One relationship* |

### One-to-Many

One-to-Many relationship is used to associate two tables in a way such that one one record from table 1 is related to multiple records in table two. For example, one user can have multiple posts.


| ![o2m.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1651413597714/FYb0w_ux6.png align="center") |
|:--:| 
| *One-to-Many relationship* |

### Many-to-Many

In Many-to-Many relationships, each record of the first table can relate to any records (or no records) in the second table and each record of the second table can also relate to more than one record of the first table. To establish a many-to-many relationship, we create a new table that store the related records together. For example, one employee can be a part of multiple projects and a project can also have multiple employees.


| ![m2m.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1651414716729/v9RNIQ56Z.png align="left") |
|:--:| 
| *Many-to-Many relationship* |

## What are polymorphic relationships

A polymorphic relationship is an association where a single model/entity can be related to multiple other tables. Lets understand with an example:-

Lets say we need to create a database for an e-commerce website that sells multiple products. Each for these products have different properties. Lets take a book and a t-shirt for example:-


![poly.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1651421007501/ojinsk1zh.png align="center")

As we can see from the above diagram, both t-shirts and books are products, but have different properties. So, how do we create a relationship between these tables?

Well, we can use polymorphic relationship here, since both t-shirt and book can belong to products. As stated before, in polymorphic relationship, an entity can belong to multiple other entities. So, both *books* and *t-shirts* can belong to products table as follows:-


![poly 24.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1651423211114/WxdbjVDAF.png align="center")

### How does it work

In the polymorphic table (products table in this case), the type field contains the name of the table in which that type id belongs. So, if the *product type* is *tshirt*, we have to look for the corresponding *product_id* in the *tshirt table*.

## Pros and Cons

The best thing about polymorphic relationships is that they are very flexible. Using these, you can relate anything quite easily. Also they are pretty easy to implement and great for **ad-hoc systems**.

On the downside, they introduce a whole new level of complexity with the indexes. They could be hard when used without any **ORM** and a lot of **database interpreting tools** can not identify them.

 