# 1. Creating a database and table

## Create a database

From your Linux user and home directory:

`psql`

Create the database:

`CREATE DATABASE testdb;`

Connect to it:

`\connect testdb;`

The psql prompt changes to show the database that you are connected to.

## Create a basic table

***Note: If you don't specify a schema when creating a table, it will be created within the default 'public' schema.***

Create a table named 'books' with two text columns, author and title:

`CREATE TABLE books (author text, title text);`

Insert a single entry:

`INSERT INTO books VALUES ('Albert Camus', 'La peste');`

Get everything from the table:

`SELECT * FROM books`
