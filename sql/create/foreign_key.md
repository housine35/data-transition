SQL Revision Notes: Foreign Keys, Joins, and Example Database
This document summarizes key SQL concepts, including foreign keys, ON DELETE/ON UPDATE actions, and different types of joins, followed by example queries for a recipe database. It serves as a reference for future work and study.
Definitions and Concepts
1. Foreign Keys

Definition: A foreign key is a column (or set of columns) in one table that refers to the primary key in another table. It enforces referential integrity, ensuring that relationships between tables remain consistent.
Purpose: Maintains data consistency and enables relationships between tables (e.g., linking a recipe to its category).
Syntax:FOREIGN KEY (column_name) REFERENCES parent_table(parent_column)


Enabling Foreign Keys: Use PRAGMA foreign_keys = ON; to enforce foreign key constraints in SQLite.

2. ON DELETE and ON UPDATE Actions
Foreign keys can define what happens when a referenced row in the parent table is deleted or updated. The possible actions are:

RESTRICT: Prevents deletion or update of the parent row if it is referenced by any child row.
SET NULL: Sets the foreign key column in the child table to NULL if the parent row is deleted or updated.
SET DEFAULT: Sets the foreign key column to its default value (if defined) when the parent row is deleted or updated.
CASCADE: Automatically deletes or updates the child rows when the parent row is deleted or updated.
Example:FOREIGN KEY (category_id) REFERENCES categories(id) ON DELETE CASCADE ON UPDATE RESTRICT



3. SQL Joins
Joins combine rows from two or more tables based on a related column. Common types include:

INNER JOIN: Returns only the rows where there is a match in both tables.
LEFT JOIN (LEFT OUTER JOIN): Returns all rows from the left table, and matched rows from the right table (unmatched rows from the right table return NULL).
RIGHT JOIN (RIGHT OUTER JOIN): Returns all rows from the right table, and matched rows from the left table (unmatched rows from the left table return NULL).
FULL JOIN (FULL OUTER JOIN): Returns all rows when there is a match in either table (not supported in SQLite).
Syntax Example:SELECT columns
FROM table1
[INNER | LEFT | RIGHT] JOIN table2
ON table1.column = table2.column;



Example Database: Recipes and Categories
Below is a practical example of a database with two tables: categories and recipes, demonstrating the use of foreign keys and joins.
1. Enable Foreign Key Support
PRAGMA foreign_keys = ON;


Explanation: Ensures that foreign key constraints are enforced in SQLite, preventing invalid data relationships.

2. Create Categories Table
CREATE TABLE IF NOT EXISTS categories (
    id INTEGER PRIMARY KEY AUTOINCREMENT,
    title VARCHAR(150) NOT NULL,
    description VARCHAR(255) NOT NULL
);


Explanation:
id: Auto-incrementing primary key.
title: Category name (e.g., "plat", "dessert").
description: Brief description of the category.
NOT NULL: Ensures these columns cannot be empty.



3. Create Recipes Table
CREATE TABLE IF NOT EXISTS recipes (
    id INTEGER PRIMARY KEY AUTOINCREMENT NOT NULL,
    title VARCHAR(150) NOT NULL,
    description VARCHAR(255) NOT NULL,
    content TEXT NOT NULL,
    category_id INTEGER,
    FOREIGN KEY (category_id) REFERENCES categories(id)
);


Explanation:
id: Auto-incrementing primary key.
title: Recipe name.
description: Short description of the recipe.
content: Detailed recipe instructions.
category_id: Foreign key linking to the id column in the categories table.
No ON DELETE or ON UPDATE actions are specified, so the default is RESTRICT.



4. Insert Sample Data (Commented)
/*
INSERT INTO categories (title, description) VALUES
    ('plat', 'A dish served as the main course'),
    ('dessert', 'A sweet course served at the end of a meal'),
    ('appetizer', 'A small dish served before the main course');

INSERT INTO recipes (title, description, category_id, content) VALUES
    ('Spaghetti Carbonara', 'A classic Italian pasta dish made with eggs, cheese, pancetta, and pepper.', 1, 'Cook spaghetti. In a bowl, mix eggs and cheese. Fry pancetta. Combine all with pepper. Serve hot.'),
    ('Chocolate Lava Cake', 'A rich chocolate cake with a gooey molten center.', 2, 'Melt chocolate and butter. Mix with sugar, eggs, and flour. Bake until edges are firm. Serve warm.'),
    ('Bruschetta', 'Grilled bread topped with diced tomatoes, garlic, and basil.', 1, 'Toast bread. Top with a mixture of tomatoes, garlic, and basil. Drizzle with olive oil. Serve fresh.');
*/


Explanation: These commented inserts populate the tables with sample data for testing. Uncomment to use.

5. Query All Categories
SELECT * FROM categories;


Explanation: Retrieves all columns and rows from the categories table.

6. Query All Recipes
SELECT * FROM recipes;


Explanation: Retrieves all columns and rows from the recipes table.

7. Example Join Query
SELECT r.description, r.content AS recipe_content
FROM recipes r
JOIN categories c
ON r.category_id = c.id;


Explanation:
Uses an INNER JOIN to combine


