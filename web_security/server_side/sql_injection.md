# SQLi (SQL injection)


![SQLi illustration](/assets/sql-injection.jpg)

## What is
SQL injection (SQLi) is a web security vulnerability that allows an attacker to interfere with the queries that an application makes to its database. It generally allows an attacker to view data that they are not normally able to retrieve. This might include data belonging to other users, or any other data that the application itself is able to access. In many cases, an attacker can modify or delete this data, causing persistent changes to the application's content or behavior.

In some situations, an attacker can escalate an SQL injection attack to compromise the underlying server or other back-end infrastructure, or perform a denial-of-service attack.


## Attacks
### 1. Retrieving hidden data
Find a way to interfere and modify the SQL query to retrieve hidden information excluded by the query.

For example, onsider a shopping application that displays products in different categories. When the user clicks on the Gifts category, their browser requests the URL:

`https://insecure-website.com/products?category=Gifts`

We can suppose that after the `category=Gift` WHERE clause, could be something else to get only published products, like:

`SELECT * FROM products WHERE category = 'Gifts' AND released = 1`

Our work is find a way to exclude the where clause or modify it to get hidden informations.

| SQLi url                                                       | SQL Query                                                                  |
|----------------------------------------------------------------|----------------------------------------------------------------------------|
| https://insecure-website.com/products?category=Gifts'--        | SELECT * FROM products WHERE category = 'Gifts'--' AND released = 1        |
| https://insecure-website.com/products?category=Gifts'+OR+1=1-- | SELECT * FROM products WHERE category = 'Gifts' OR 1=1--' AND released = 1 |

The key thing here is that the double-dash sequence <b>-- is a comment indicator in SQL</b>, and means that the rest of the query is interpreted as a comment.

In the first case, we close the category field with ' and then comment the following characters.
In the second case, we force a "always true" where clause by adding `OR 1=1` instruction.

### 2. Twist application logic
Consider an application that lets users log in with a username and password. If a user submits the username admin and the password bluecheese, the application checks the credentials by performing the following SQL query:
`SELECT * FROM users WHERE username = 'admin' AND password = 'bluecheese'`

We can use the same techniques seen above to remove the password check and gain access:
`SELECT * FROM users WHERE username = 'administrator'--' AND password = ''`
