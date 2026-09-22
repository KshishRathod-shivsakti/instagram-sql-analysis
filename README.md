# instagram-sql-analysis

Driven by curiosity and a desire to test my SQL analysis skills, I built a mini-project analyzing Instagram user engagement and content performance in MySQL

Concept used : DDL &amp; DML , Aggregations &amp; Filtering: GROUP BY, HAVING, and conditional logic (CASE WHEN),joins ,cte, window function 





-- ==========================================================
-- PHASE 1: DDL - Schema Creation (4 Relational Tables)
-- ==========================================================

DROP TABLE IF EXISTS comments CASCADE;
DROP TABLE IF EXISTS posts CASCADE;
DROP TABLE IF EXISTS followers CASCADE;
DROP TABLE IF EXISTS users CASCADE;

CREATE TABLE users (
    user_id INT PRIMARY KEY,
    username VARCHAR(50) UNIQUE NOT NULL,
    country VARCHAR(50) NOT NULL,
    created_at TIMESTAMP NOT NULL,
    account_type VARCHAR(20) CHECK (account_type IN ('Personal', 'Creator', 'Business'))
);

CREATE TABLE followers (
    follower_id INT,
    following_id INT,
    created_at TIMESTAMP NOT NULL,
    PRIMARY KEY (follower_id, following_id),
    FOREIGN KEY (follower_id) REFERENCES users(user_id),
    FOREIGN KEY (following_id) REFERENCES users(user_id)
);

CREATE TABLE posts (
    post_id INT PRIMARY KEY,
    user_id INT REFERENCES users(user_id),
    caption TEXT,
    post_type VARCHAR(20) CHECK (post_type IN ('Reel', 'Carousel', 'Image')),
    likes INT DEFAULT 0,
    created_at TIMESTAMP NOT NULL
);

CREATE TABLE comments (
    comment_id INT PRIMARY KEY,
    post_id INT REFERENCES posts(post_id),
    user_id INT REFERENCES users(user_id),
    comment_text TEXT NOT NULL,
    created_at TIMESTAMP NOT NULL
);

-- ==========================================================
-- PHASE 2: DML - Seed Data Insertion (100+ Records)
-- ==========================================================

-- 1. Insert Users (12 Users)
INSERT INTO users (user_id, username, country, created_at, account_type) VALUES
(1, 'tech_guru', 'USA', '2023-01-10 08:30:00', 'Creator'),
(2, 'fit_life', 'India', '2023-02-15 09:15:00', 'Creator'),
(3, 'foodie_delight', 'India', '2023-03-01 12:00:00', 'Business'),
(4, 'wanderlust_sam', 'UK', '2023-03-20 14:45:00', 'Personal'),
(5, 'style_icon', 'USA', '2023-04-05 18:20:00', 'Creator'),
(6, 'code_master', 'India', '2023-05-12 11:10:00', 'Personal'),
(7, 'baking_heaven', 'Canada', '2023-06-01 07:00:00', 'Business'),
(8, 'daily_memes', 'USA', '2023-06-18 20:00:00', 'Creator'),
(9, 'travel_diaries', 'UK', '2023-07-22 16:30:00', 'Creator'),
(10, 'fitness_freak', 'India', '2023-08-10 06:45:00', 'Personal'),
(11, 'art_gallery', 'Germany', '2023-09-01 13:15:00', 'Business'),
(12, 'photo_dump', 'USA', '2023-10-10 19:50:00', 'Personal');

-- 2. Insert Follower Relationships (25 Relationships)
INSERT INTO followers (follower_id, following_id, created_at) VALUES
(2, 1, '2023-02-16 10:00:00'), (3, 1, '2023-03-02 11:30:00'), (4, 1, '2023-03-21 15:00:00'),
(5, 1, '2023-04-06 09:12:00'), (6, 1, '2023-05-13 14:20:00'), (1, 2, '2023-02-16 12:00:00'),
(3, 2, '2023-03-05 08:00:00'), (10, 2, '2023-08-11 07:30:00'), (1, 3, '2023-03-02 10:00:00'),
(7, 3, '2023-06-02 18:00:00'), (2, 5, '2023-04-10 11:00:00'), (8, 5, '2023-06-19 21:00:00'),
(9, 5, '2023-07-23 10:10:00'), (12, 5, '2023-10-11 15:40:00'), (1, 6, '2023-05-15 08:30:00'),
(2, 6, '2023-05-16 09:20:00'), (4, 9, '2023-07-25 12:00:00'), (11, 9, '2023-09-02 16:00:00'),
(5, 8, '2023-06-20 10:00:00'), (6, 8, '2023-06-21 11:30:00'), (10, 8, '2023-08-15 13:00:00'),
(12, 8, '2023-10-12 18:20:00'), (3, 11, '2023-09-05 09:00:00'), (7, 11, '2023-09-06 10:00:00'),
(1, 12, '2023-10-15 22:00:00');

-- 3. Insert Posts (35 Posts)
INSERT INTO posts (post_id, user_id, caption, post_type, likes, created_at) VALUES
(101, 1, 'Top 5 SQL Tricks for Beginners', 'Reel', 1500, '2023-05-01 10:00:00'),
(102, 1, 'My Setup for Coding in 2023', 'Carousel', 850, '2023-05-10 14:30:00'),
(103, 1, 'Why Database Indexing Matters', 'Reel', 2300, '2023-06-01 09:15:00'),
(104, 2, 'Morning HIIT Workout Routine', 'Reel', 3100, '2023-05-15 06:30:00'),
(105, 2, 'What I Eat in a Day', 'Carousel', 1200, '2023-05-20 12:00:00'),
(106, 2, 'Leg Day Motivation', 'Image', 450, '2023-06-10 17:45:00'),
(107, 3, 'Best Pizza in Town!', 'Image', 980, '2023-04-10 19:00:00'),
(108, 3, 'Behind the Scenes in Our Kitchen', 'Reel', 4200, '2023-05-02 18:30:00'),
(109, 4, 'Exploring Bali Beaches', 'Carousel', 1600, '2023-06-05 11:00:00'),
(110, 4, 'Sunset in Paradise', 'Image', 720, '2023-06-12 20:15:00'),
(111, 5, 'Summer Fashion Trends', 'Reel', 5100, '2023-05-25 15:00:00'),
(112, 5, 'OOTD Streetwear Edition', 'Carousel', 2900, '2023-06-08 16:30:00'),
(113, 6, 'Python vs SQL for Data Analysis', 'Reel', 1850, '2023-06-01 13:00:00'),
(114, 6, 'Bug Fixing at 2 AM', 'Image', 310, '2023-06-15 02:00:00'),
(115, 7, 'How to Bake Perfect Sourdough', 'Reel', 3800, '2023-06-20 08:00:00'),
(116, 7, 'Chocolate Lava Cake Recipe', 'Carousel', 2700, '2023-07-01 10:30:00'),
(117, 8, 'Monday Blues Meme', 'Image', 6200, '2023-06-26 08:00:00'),
(118, 8, 'Programmer Jokes Part 1', 'Carousel', 8900, '2023-07-05 14:00:00'),
(119, 8, 'When the Code Works First Try', 'Reel', 12500, '2023-07-15 19:30:00'),
(120, 9, 'Hidden Gems in Italy', 'Reel', 3400, '2023-08-01 11:20:00'),
(121, 9, 'Rome Architecture Walk', 'Image', 1100, '2023-08-08 17:00:00'),
(122, 10, 'Deadlift PR Day!', 'Reel', 2100, '2023-08-20 07:15:00'),
(123, 11, 'Oil Painting Timelapse', 'Reel', 1950, '2023-09-10 15:00:00'),
(124, 11, 'Exhibition Highlights', 'Carousel', 800, '2023-09-18 18:45:00'),
(125, 12, 'Weekend Dump with Friends', 'Carousel', 640, '2023-10-15 21:00:00'),
(126, 1, 'Data Engineering Roadmap 2024', 'Carousel', 1900, '2023-10-20 10:00:00'),
(127, 2, '5 Min Abs Workout', 'Reel', 4100, '2023-10-22 06:00:00'),
(128, 5, 'Autumn Outfit Ideas', 'Reel', 6300, '2023-10-25 14:00:00'),
(129, 8, 'Relatable Office Humor', 'Image', 7100, '2023-11-01 12:00:00'),
(130, 3, 'New Menu Launch!', 'Carousel', 1500, '2023-11-05 17:30:00'),
(131, 4, 'Hiking the Swiss Alps', 'Reel', 4800, '2023-11-10 09:00:00'),
(132, 6, 'SQL Window Functions Simplified', 'Reel', 3200, '2023-11-12 11:00:00'),
(133, 7, 'Croissant Making Masterclass', 'Reel', 5600, '2023-11-15 08:30:00'),
(134, 9, 'Tokyo Night Lights', 'Image', 2200, '2023-11-18 20:00:00'),
(135, 10, 'Protein Smoothie Recipes', 'Carousel', 1350, '2023-11-20 10:15:00');

-- 4. Insert Comments (40 Comments)
INSERT INTO comments (comment_id, post_id, user_id, comment_text, created_at) VALUES
(1, 101, 2, 'Great explanation!', '2023-05-01 10:15:00'),
(2, 101, 6, 'Super helpful tricks, thanks.', '2023-05-01 11:00:00'),
(3, 101, 10, 'Loved trick #3!', '2023-05-01 12:30:00'),
(4, 103, 6, 'Indexing saved my production database yesterday.', '2023-06-01 10:00:00'),
(5, 104, 10, 'Burned so many calories doing this!', '2023-05-15 07:00:00'),
(6, 104, 5, 'Adding this to my morning routine.', '2023-05-15 08:20:00'),
(7, 108, 1, 'Looks delicious! Need to visit.', '2023-05-02 19:00:00'),
(8, 108, 4, 'Where is this restaurant located?', '2023-05-02 19:30:00'),
(9, 108, 7, 'The kitchen looks immaculate!', '2023-05-02 20:10:00'),
(10, 109, 9, 'Bali is incredible, check out Ubud!', '2023-06-05 12:00:00'),
(11, 111, 4, 'Love the second outfit!', '2023-05-25 15:45:00'),
(12, 111, 12, 'Where is the jacket from?', '2023-05-25 16:10:00'),
(13, 113, 1, 'SQL is definitely better for quick joins.', '2023-06-01 13:30:00'),
(14, 115, 3, 'We need this bread in our restaurant!', '2023-06-20 09:00:00'),
(15, 117, 12, 'Literally me every Monday morning.', '2023-06-26 08:30:00'),
(16, 118, 6, 'The 404 error joke had me dying.', '2023-07-05 14:30:00'),
(17, 118, 1, 'Too accurate haha!', '2023-07-05 15:00:00'),
(18, 119, 2, 'Best feeling in the world!', '2023-07-15 20:00:00'),
(19, 119, 6, 'Rare event but glorious!', '2023-07-15 20:15:00'),
(20, 119, 8, 'Glad you guys like it!', '2023-07-15 20:30:00'),
(21, 120, 4, 'Saving this for my next trip!', '2023-08-01 12:00:00'),
(22, 122, 2, 'Clean form! Keep pushing.', '2023-08-20 08:00:00'),
(23, 123, 5, 'The detail work is stunning.', '2023-09-10 16:00:00'),
(24, 126, 6, 'Saving this roadmap for reference.', '2023-10-20 11:00:00'),
(25, 127, 10, 'That core burn is real.', '2023-10-22 07:00:00'),
(26, 128, 12, 'Fall fashion is the best season.', '2023-10-25 15:00:00'),
(27, 129, 4, 'Sending this to my manager right now.', '2023-11-01 13:00:00'),
(28, 131, 9, 'The Alps are unmatched!', '2023-11-10 10:00:00'),
(29, 132, 1, 'Solid breakdown of DENSE_RANK()!', '2023-11-12 12:00:00'),
(30, 133, 3, 'Flaky perfection!', '2023-11-15 09:00:00'),
(31, 101, 4, 'Awesome post!', '2023-05-01 13:00:00'),
(32, 108, 2, 'Yum!', '2023-05-02 21:00:00'),
(33, 111, 8, 'Fire drip 🔥', '2023-05-25 17:00:00'),
(34, 117, 2, 'So relatable', '2023-06-26 09:00:00'),
(35, 119, 10, 'A masterpiece meme', '2023-07-15 21:00:00'),
(36, 128, 2, 'Stunning outfit', '2023-10-25 16:00:00'),
(37, 132, 10, 'Subqueries vs CTE cleared up thanks!', '2023-11-12 13:00:00'),
(38, 119, 4, 'Shared this with my team!', '2023-07-15 22:00:00'),
(39, 103, 2, 'Need part 2 ASAP!', '2023-06-01 11:00:00'),
(40, 118, 5, 'Sending to my dev friends', '2023-07-05 16:00:00');


-- ==========================================================
-- INSTAGRAM ANALYTICS --
-- Database System: MySQL 8.0+
-- ==========================================================


-- ==========================================================
-- PHASE 1: Data Cleaning & Integrity Checks
-- ==========================================================

-- Problem 1: Check for invalid likes or missing post types
SELECT * 
FROM posts 
WHERE likes < 0 
   OR post_type IS NULL;

-- Problem 2: Identify orphan comments (comments linked to non-existent posts)
SELECT c.comment_id, c.post_id 
FROM comments c
LEFT JOIN posts p ON c.post_id = p.post_id
WHERE p.post_id IS NULL;


-- ==========================================================
-- PHASE 2: Exploratory Data Analysis (EDA)
-- ==========================================================

-- Problem 3: Post type distribution and percentage share
SELECT 
    post_type,
    COUNT(*) AS total_posts,
    ROUND((COUNT(*) * 100.0 / (SELECT COUNT(*) FROM posts)), 2) AS percentage_share
FROM posts
GROUP BY post_type
ORDER BY total_posts DESC;

-- Problem 4: Peak upload hours (Using MySQL HOUR function)
SELECT 
    HOUR(created_at) AS upload_hour,
    COUNT(*) AS total_posts
FROM posts
GROUP BY upload_hour
ORDER BY total_posts DESC;

-- Problem 5: Categorize posts into engagement tiers using CASE WHEN
SELECT 
    CASE 
        WHEN likes < 1000 THEN 'Low'
        WHEN likes BETWEEN 1000 AND 4000 THEN 'Medium'
        ELSE 'High'
    END AS engagement_tier,
    COUNT(*) AS post_count
FROM posts
GROUP BY engagement_tier
ORDER BY post_count DESC;


-- ==========================================================
-- PHASE 3: Business Analytics & Multi-Table Joins
-- ==========================================================

-- Problem 6: Top 3 users by total engagement (Likes + Comments)
SELECT 
    u.username,
    u.account_type,
    (IFNULL(SUM(p.likes), 0) + COUNT(c.comment_id)) AS total_engagement
FROM users u
JOIN posts p ON u.user_id = p.user_id
LEFT JOIN comments c ON p.post_id = c.post_id
GROUP BY u.user_id, u.username, u.account_type
ORDER BY total_engagement DESC
LIMIT 3;

-- Problem 7: Posts performing above platform average likes
SELECT 
    post_id,
    caption,
    likes
FROM posts
WHERE likes > (SELECT AVG(likes) FROM posts)
ORDER BY likes DESC;

-- Problem 8: Average comments per post across post types (MySQL CAST)
SELECT 
    p.post_type,
    COUNT(DISTINCT p.post_id) AS total_posts,
    COUNT(c.comment_id) AS total_comments,
    ROUND(CAST(COUNT(c.comment_id) AS DECIMAL(10,2)) / COUNT(DISTINCT p.post_id), 2) AS avg_comments_per_post
FROM posts p
LEFT JOIN comments c ON p.post_id = c.post_id
GROUP BY p.post_type
ORDER BY avg_comments_per_post DESC;


-- ==========================================================
-- PHASE 4: Advanced SQL (Window Functions & CTEs)
-- ==========================================================

-- Problem 9: Top 2 most-liked posts within each post_type (DENSE_RANK)
WITH RankedPosts AS (
    SELECT 
        post_id,
        user_id,
        post_type,
        likes,
        caption,
        DENSE_RANK() OVER (
            PARTITION BY post_type 
            ORDER BY likes DESC
        ) AS rank_position
    FROM posts
)
SELECT 
    r.post_type,
    r.rank_position,
    u.username,
    r.likes,
    r.caption
FROM RankedPosts r
JOIN users u ON r.user_id = u.user_id
WHERE r.rank_position <= 2
ORDER BY r.post_type, r.rank_position;

-- Problem 10: Creator ranking based on average likes (CTE + HAVING + Window Function)
WITH CreatorStats AS (
    SELECT 
        user_id,
        COUNT(post_id) AS total_posts,
        AVG(likes) AS avg_likes
    FROM posts
    GROUP BY user_id
    HAVING COUNT(post_id) >= 2
)
SELECT 
    u.username,
    cs.total_posts,
    ROUND(cs.avg_likes, 0) AS avg_likes,
    RANK() OVER (ORDER BY cs.avg_likes DESC) AS creator_rank
FROM CreatorStats cs
JOIN users u ON cs.user_id = u.user_id;
