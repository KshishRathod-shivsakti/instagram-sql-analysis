# instagram-sql-analysis

Driven by curiosity and a desire to test my SQL analysis skills, I built a mini-project analyzing Instagram user engagement and content performance in MySQL

Concept used : DDL &amp; DML , Aggregations &amp; Filtering: GROUP BY, HAVING, and conditional logic (CASE WHEN),joins ,cte, window function 

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
