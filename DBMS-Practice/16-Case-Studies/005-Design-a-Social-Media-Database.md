# Design a Social Media Database

## Scenario
A social platform needs: users, posts, a many-to-many "follows" relationship, likes, and a home feed showing recent posts from people you follow - at a scale where a naive feed query becomes a serious performance problem.

## Requirements
- Users follow other users (asymmetric - following isn't automatically mutual).
- Posts can be liked by many users; a user can only like a post once.
- The home feed shows recent posts from everyone a user follows, in reverse-chronological order.

## Diagram
```
User          Follow (self-referencing M:N)         Post              Like
+-------+  M    M  +-----------------+       +-----+  1   M  +------+   M    M  +------+
| u_id  |<--------->| follower_id  FK |       |p_id |------->|post  |<--------->|u_id  |
| name  |            | followee_id FK |       |u_id |        |_id   |            |p_id  |
+-------+            +-----------------+       |body |        +------+            +------+
                                                +-----+
```

## Schema
```sql
CREATE TABLE "User" (
    user_id  SERIAL PRIMARY KEY,
    username VARCHAR(50) UNIQUE NOT NULL
);

CREATE TABLE Follow (
    follower_id INT NOT NULL REFERENCES "User"(user_id),
    followee_id INT NOT NULL REFERENCES "User"(user_id),
    followed_at TIMESTAMPTZ NOT NULL DEFAULT now(),
    PRIMARY KEY (follower_id, followee_id),
    CHECK (follower_id <> followee_id)   -- can't follow yourself
);

CREATE TABLE Post (
    post_id    BIGSERIAL PRIMARY KEY,
    user_id    INT NOT NULL REFERENCES "User"(user_id),
    body       TEXT NOT NULL,
    created_at TIMESTAMPTZ NOT NULL DEFAULT now()
);

CREATE TABLE "Like" (
    user_id INT NOT NULL REFERENCES "User"(user_id),
    post_id BIGINT NOT NULL REFERENCES Post(post_id),
    liked_at TIMESTAMPTZ NOT NULL DEFAULT now(),
    PRIMARY KEY (user_id, post_id)   -- prevents liking the same post twice
);
```

## Naive feed query (works, but doesn't scale)
```sql
SELECT p.*
FROM Post p
JOIN Follow f ON f.followee_id = p.user_id
WHERE f.follower_id = :me
ORDER BY p.created_at DESC
LIMIT 20;
```
This join-and-sort-on-read approach ("pull" model) gets expensive once a user follows thousands of accounts, or a followed account has millions of posts to filter through, at read time, on every single feed refresh.

## Key design decision: fan-out on write
At scale, platforms typically pre-compute each user's feed at **write time** instead: when a post is created, insert an entry into every follower's `FeedEntry(user_id, post_id, created_at)` table immediately, so reading the feed later is a single fast indexed lookup on `FeedEntry WHERE user_id = :me ORDER BY created_at DESC LIMIT 20` - no join or fan-out cost at read time.

```
                 (on post creation, "fan out" to followers' feed tables)
Post created -> for each follower: INSERT INTO FeedEntry(follower_id, post_id, created_at)
```

**Tradeoff:** this "fan-out on write" trades write cost (one post from a celebrity with 10 million followers means 10 million feed-table inserts) for read speed - which is why most large platforms use a hybrid: fan-out on write for normal users, and a "pull" (fan-out on read) exception path specifically for accounts with huge follower counts.

## Takeaway
Feed generation is a canonical "fan-out on write vs. fan-out on read" tradeoff - the schema above supports the simple/correct read-time-join version, but understanding when and why you'd denormalize into a precomputed `FeedEntry` table is the real system-design lesson here.
