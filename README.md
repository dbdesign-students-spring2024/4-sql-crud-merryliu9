## Write a report

Update the `README.md` file to include the solutions to all of these instructions. For each part, be sure to include:

- the SQL code to create each of the required tables
- a link to each of the practice CSV data files in the `data` directory.
- the SQLite code to import the practice CSV data files into the tables.
- the SQL queries that solve each of the tasks you were asked to do. Make it clear which task each query is intended to solve - include the task number and text on the line above the SQL code solution.

Make the document well-formatted using Markdown code. Use clear headings and sub-headings and use Markdown's ability to embed code snippets into a document.

# SQL CRUD
## Part 1: Restaurant finder
### SQL Code to create required tables
#### restaurant
```
CREATE TABLE restaurants (
    id INTEGER PRIMARY KEY,
    name TEXT,
    category TEXT,
    price_tier TEXT,
    neighborhood TEXT,
    opening_hours TEXT,
    average_rating REAL,
    good_for_kids BOOLEAN
);
```
#### reviews
```
CREATE TABLE reviews (
    id INTEGER PRIMARY KEY,
    restaurant_id INTEGER,
    rating INTEGER,
    comment TEXT,
    FOREIGN KEY (restaurant_id) REFERENCES restaurants(id)
);
```
### link to practice CSV data
- [restaurant](data/restaurants.csv)
### SQLite code to import the practice CSV data files into the tables
```
.mode csv
.import data/restaurants.csv restaurants
```
### SQL queries
> Replace the terms in square brackets with the real values (and delete the square brackets).
#### 1. Find all cheap restaurants in a particular neighborhood (pick any neighborhood as an example).
```
SELECT * FROM restaurants WHERE price_tier = 'cheap' AND neighborhood = '[Picked Neighborhood]';
```
#### 2. Find all restaurants in a particular genre (pick any genre as an example) with 3 stars or more, ordered by the number of stars in descending order.
```
SELECT * FROM restaurants WHERE category = '[Desired Category]' AND average_rating >= 3 ORDER BY average_rating DESC;
```
#### 3. Find all restaurants that are open now (see hint below).
```
SELECT * FROM restaurants WHERE strftime('%H:%M', '[time_now]') BETWEEN substr(opening_hours, 1, 5) AND substr(opening_hours, 7, 5);
```
#### 4. Leave a review for a restaurant (pick any restaurant as an example; note that leaving a review has no automatic effect on the average rating of the restaurant).
```
INSERT INTO reviews (restaurant_id, rating, comment) VALUES ([Desired_Restaurant_ID], [Desired_Rating], '[Desired Comment]');
```
#### 5. Delete all restaurants that are not good for kids.
```
DELETE FROM restaurants WHERE good_for_kids = 0;
```
#### 6. Find the number of restaurants in each NYC neighborhood.
```
SELECT neighborhood, COUNT(*) FROM restaurants GROUP BY neighborhood;
```
## Part 2: Social media app
### SQL Code to create required tables
#### users
```
CREATE TABLE users (
    id INTEGER PRIMARY KEY,
    email TEXT,
    password TEXT,
    handle TEXT
);
```
#### posts
```
CREATE TABLE posts (
    id INTEGER PRIMARY KEY,
    sender_id INTEGER,
    receiver_id INTEGER,
    type TEXT CHECK(type IN ('message', 'story')),
    text TEXT,
    created_at TEXT,
    visible BOOLEAN,
    FOREIGN KEY (sender_id) REFERENCES users(id),
    FOREIGN KEY (receiver_id) REFERENCES users(id)
);
```
### link to practice CSV data
- [users](data/users.csv)
- [posts](data/posts.csv)
### SQLite code to import the practice CSV data files into the tables
```
.mode csv
.import data/users.csv users
.import data/posts.csv posts
```
### SQL queries
> Replace the terms in square brackets with the real values (and delete the square brackets).
#### 1. Register a new User.
```
INSERT INTO users (email, password, handle) VALUES ('[new_user@example.com]', '[new_password]', '[new_handle]');
```
#### 2. Create a new Message sent by a particular User to a particular User (pick any two Users for example).
```
INSERT INTO posts (sender_id, receiver_id, type, text, created_at) 
VALUES ([desired_sender_id], [desired_receiver_id], 'message', 'Message text', DATETIME('[time_now]'));
```
#### 3. Create a new Story by a particular User (pick any User for example).
```
INSERT INTO posts (sender_id, type, text, created_at)
VALUES ([desired_sender_id], 'story', [story_created], DATETIME('[time_now]'));
```
#### 4. Show the 10 most recent visible Messages and Stories, in order of recency.
```
SELECT * FROM posts WHERE visible = 1 ORDER BY created_at DESC LIMIT 10;
```
#### 5. Show the 10 most recent visible Messages sent by a particular User to a particular User (pick any two Users for example), in order of recency.
```
SELECT * FROM posts WHERE sender_id = [desired_sender_id] AND receiver_id = [desired_receiver_id] AND visible = 1 AND type = 'message' ORDER BY created_at DESC LIMIT 10;
```
#### 6. Make all Stories that are more than 24 hours old invisible.
```
UPDATE posts SET visible = 0 WHERE type = 'story' AND DATETIME('[time_now]') - DATETIME(created_at) > 24;
```
#### 7. Show all invisible Messages and Stories, in order of recency.
```
SELECT * FROM posts WHERE visible = 0 ORDER BY created_at DESC;
```
#### 8. Show the number of posts by each User.
```
SELECT users.handle, COUNT(posts.id) AS post_count
FROM users LEFT JOIN posts ON users.id = posts.sender_id
GROUP BY users.id;
```
#### 9. Show the post text and email address of all posts and the User who made them within the last 24 hours.
```
SELECT users.email, posts.text
FROM users JOIN posts ON users.id = posts.sender_id
WHERE DATETIME('[time_now]') - DATETIME(posts.created_at) <= 24;
```
#### 10. Show the email addresses of all Users who have not posted anything yet.
```
SELECT email FROM users WHERE id NOT IN (SELECT DISTINCT sender_id FROM posts);
```