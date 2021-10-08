# Extras

## New User Share

We have the following data table:

| id | int |
| - | - |
| **time_id** | **datetime** |
| **user_id** | **varchar** |
| **customer_id** | **varchar** |
| **client_id** | **varchar** |
| **event_type** | **varchar** |
| **event_id** | **int** |

We want to calculate the share of new and existing users and output the month, share of new users, and share of existing users as a ratio. Here, new user refers to the that started using the product within current month. Two alternative queries are proposed to do this calculation and you can find both of them below.

### Alternative Query 1

```sql
DELIMITER //
CREATE PROCEDURE GetNewUserShare()
BEGIN
	DECLARE new_users INT;
	DECLARE users INT;
	DECLARE cur_month INT;
	SELECT MONTH(current_date) INTO cur_month;
	SELECT COUNT(DISTINCT user_id) INTO new_users
    FROM Test
    WHERE MONTH(time_id) = cur_month;
    SELECT COUNT(DISTINCT user_id) INTO users
    FROM Test;
    CREATE TEMPORARY TABLE t(
      current_ month INT NULL,
      new_user_share FLOAT NULL,
      existing_user_share FLOAT NULL);  
    INSERT INTO t VALUES(cur_month, new_users/(users), (users-new_users)/users);
    SELECT * FROM t;
END //

DELIMITER ;

CALL GetNewUserShare();
```

### Alternative Query 2

```sql
DELIMITER //

CREATE PROCEDURE GetNewUserShare2()
BEGIN
	SELECT
    	MONTH(current_date) AS current_month, 
    	SUM(CASE WHEN MONTH(time_id) = MONTH(current_date) THEN 1 ELSE 0 END) / COUNT(*) AS new_user_ratio, 
        SUM(CASE WHEN MONTH(time_id) = MONTH(current_date) THEN 0 ELSE 1 END) / COUNT(*) AS existing_user_ratio
	FROM Test;
END //

DELIMITER ;

CALL GetNewUserShare2();
```

Both of the queries outputs the same table represented below.

| GetNewUserShare() | |
| - | - |
| current_month | int |
| new_user_ratio | float |
| existing_user_ratio | float |

## nth Highest Follower Count

We have the following followerCount table:

| Id | followerCount |
|----|---------------|
| 1 | 100 |
| 2 | 200 |
| 3 | 300 |

Following query returns the instance with nth highest follower count on the table:

```sql
DELIMITER //
CREATE PROCEDURE NthHighestFollowerCount
(
  IN n INT
)
BEGIN
    DECLARE n2 INT DEFAULT n-1;
	SELECT followerCount
	FROM followerCount
    ORDER BY followerCount DESC LIMIT n2 , 1;
END //

DELIMITER ;

CALL NthHighestFollowerCount(2)
```

Query output will be as follows:

| NthHighestFollowerCount (2) |
| - |
| 200 |

## Dynamic Feed

Let say there is a social media app that currently has a static content feed (similar to Instagram’s explore page but the content is the same for everyone). Currently, the feed is static where it shows the most liked outfits from the last 12-hour window. We want to devise a plan to make this feed fully dynamic such that the user’s previous activity and geographically trending outfits are shown. What can we do?

To do that first we need to identify what would be helpful for our analysis:

1. User statistics
    This data will be representing the individual actions of the user (e.g. liked, saved, comments liked, comments posted).

2. Finding similar accounts
    Accounts people have interacted with before is crucial for our dynamic feed. We would find better content for a user if we can identify other accounts a person might be interested in. We can use this interacted accounts to find media these accounts have posted or engaged with. This interactions might be occur in several ways like follow, like, comment, save, share etc.

3. Using spatial data
    Spatial data useful while improving our similar account pool meaning we can use geo-distance as a measure to similarity to the other accounts.

Similar accounts for the user might result in very high number of people that we cannot fit to our feed. In that case, we hve to implement ranking algorithm to narrow down those choices to fit our feed. How we can do this ranking?

1. We can create similarity-dissimilarity metrics with individual statistics and similar account statistics. However, those statistics needs to be weighted and therefore tuned on past data.

2. We can use algorithms like KDTree (usually runs very fast even with the large amount of data) to rank our similar accounts.

3. We can integrate kmeans clustering for the contents. Hence, instead of ranking the accounts we can rank the related posts.

After all we don't want to end up with a feed with very similar posts. Therefore, we should implement categorization to maintain diversity in the content feed. This can be done by labeling posts in our database.
