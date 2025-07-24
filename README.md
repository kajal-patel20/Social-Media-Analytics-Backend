# Social-Media-Analytics-Backend
# 📊 Social Media Analytics Backend

A SQL Server–based backend system designed to analyze user engagement on social media posts, including likes, comments, and post performance rankings.

This project is part of my learning and internship submission, aimed at demonstrating my understanding of database design concepts.

## 🎯 Objective

Create a SQL system to:
- Track post engagement via likes and comments
- Maintain like counts with triggers
- Generate rankings using window functions
- Provide views for easy analytics and reporting

## 🛠 Tools & Technologies

- **SQL Server**
- **T-SQL** (Views, Triggers, Window Functions)
- **SQL Server Management Studio (SSMS)**


## 🧱 Database Schema

- **Users**: Stores user info (`user_id`, `username`, `email`)
- **Posts**: User-created content with engagement data
- **Likes**: Tracks who liked which post and when
- **Comments**: Stores comment content per post

### 🔗 Relationships

- `Posts.user_id` → `Users.user_id`
- `Likes.user_id` & `Likes.post_id` → `Users` & `Posts`
- `Comments.user_id` & `Comments.post_id` → `Users` & `Posts`


## 🧪 Features & Logic

### 1. Schema Design  
Created 4 normalized tables with foreign key constraints.

### 2. Sample Data  
Inserted realistic users, posts, likes, and comments.

### 3. Views  
- `top_posts`: Posts with highest like counts and comment totals  
- `engagement_scores`: Calculates engagement (likes + comments)  
- `ranked_posts`: Ranks posts using `RANK()` window function  

### 4. Triggers  
- `trg_LikeInsert`: Updates `Posts.like_count` when a like is added  
- `trg_LikeDelete`: Decreases `like_count` when a like is removed

### 5. Reporting  
Query `ranked_posts` and export result grid from SSMS as CSV.

# 💻 SQL Script

Creating the data

```
CREATE TABLE Users (
    user_id INT IDENTITY(1,1) PRIMARY KEY,
    username VARCHAR(50) NOT NULL UNIQUE,
    email VARCHAR(100) NOT NULL UNIQUE,
    created_at DATETIME DEFAULT GETDATE()
);

CREATE TABLE Posts (
    post_id INT IDENTITY(1,1) PRIMARY KEY,
    user_id INT FOREIGN KEY REFERENCES Users(user_id),
    content VARCHAR(MAX) NOT NULL,
    like_count INT DEFAULT 0,
    created_at DATETIME DEFAULT GETDATE()
);

CREATE TABLE Likes (
    like_id INT IDENTITY(1,1) PRIMARY KEY,
    user_id INT FOREIGN KEY REFERENCES Users(user_id),
    post_id INT FOREIGN KEY REFERENCES Posts(post_id),
    liked_at DATETIME DEFAULT GETDATE()
);

CREATE TABLE Comments (
    comment_id INT IDENTITY(1,1) PRIMARY KEY,
    user_id INT FOREIGN KEY REFERENCES Users(user_id),
    post_id INT FOREIGN KEY REFERENCES Posts(post_id),
    comment_text VARCHAR(MAX) NOT NULL,
    commented_at DATETIME DEFAULT GETDATE()
);
```

Inserting the data

```
INSERT INTO Users (username, email) VALUES 
('Kajal', 'kajal4636@gmail.com'),
('Naved', 'naved7562@gmail.com'),
('Zanib', 'zanib9371@gmail.com');

INSERT INTO Posts (user_id, content) VALUES 
(1, 'Life is like a mirror. We get the best results when we smile.'),
(2, 'Life is better when you’re laughing.'),
(3, 'A smile can change the world.');

INSERT INTO Likes (user_id, post_id) VALUES 
(1, 2), (2, 1), (3, 1), (1, 3);

INSERT INTO Comments (user_id, post_id, comment_text) VALUES 
(2, 1, 'Nice post!'),
(3, 1, 'Welcome!'),
(1, 2, 'Interesting read.');
```

Views for Top Posts and Engagement Scores

```
CREATE VIEW top_posts AS
SELECT p.post_id, p.content, p.like_count, COUNT(c.comment_id) AS comment_count
FROM Posts p
LEFT JOIN Comments c ON p.post_id = c.post_id
GROUP BY p.post_id, p.content, p.like_count;

CREATE VIEW engagement_scores AS
SELECT 
    p.post_id, 
    p.content, 
    p.like_count, 
    COUNT(c.comment_id) AS comment_count,
    (p.like_count + COUNT(c.comment_id)) AS engagement_score
FROM Posts p
LEFT JOIN Comments c ON p.post_id = c.post_id
GROUP BY p.post_id, p.content, p.like_count;
```

Window Functions for Ranking

```
CREATE VIEW ranked_posts AS
SELECT *,
       RANK() OVER (ORDER BY (like_count + comment_count) DESC) AS rank
FROM (
    SELECT 
        p.post_id, 
        p.content, 
        p.like_count, 
        COUNT(c.comment_id) AS comment_count
    FROM Posts p
    LEFT JOIN Comments c ON p.post_id = c.post_id
    GROUP BY p.post_id, p.content, p.like_count
) AS engagement_data;
```

Triggers to Update Like Count
```
CREATE TRIGGER trg_LikeInsert
ON Likes
AFTER INSERT
AS
BEGIN
    UPDATE Posts
    SET like_count = like_count + 1
    WHERE post_id IN (SELECT post_id FROM inserted);
END;
```

Delete Trigger
```
CREATE TRIGGER trg_LikeDelete
ON Likes
AFTER DELETE
AS
BEGIN
    UPDATE Posts
    SET like_count = like_count - 1
    WHERE post_id IN (SELECT post_id FROM deleted);
END;
```

Showing the data
```
SELECT * FROM ranked_posts ORDER BY rank;
```

Output:-

<img width="659" height="132" alt="image" src="https://github.com/user-attachments/assets/35cc3107-f233-49fe-b26d-73311562210f" />





