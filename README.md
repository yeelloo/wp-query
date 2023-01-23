# wp-query
Optimization Queries 

```
// SHOW SIZE OF ALL autoloads:
SELECT SUM(LENGTH(option_value)) as autoload_size FROM wp_options WHERE autoload='yes';

// SHOW TOTAL AUTOLOAD AND TOP 10 IN SIZE:
SELECT 'autoloaded data in KiB' as name, ROUND(SUM(LENGTH(option_value))/ 1024) as value FROM wp_options WHERE autoload='yes'
2UNION
3SELECT 'autoloaded data count', count(*) FROM wp_options WHERE autoload='yes'
4UNION
5(SELECT option_name, length(option_value) FROM wp_options WHERE autoload='yes' ORDER BY length(option_value) DESC LIMIT 10)
6

// TERMS CLEANUP
DELETE FROM wp_terms WHERE term_id IN (SELECT term_id FROM wp_term_taxonomy WHERE count = 0 ); 

DELETE FROM wp_term_taxonomy WHERE term_id not IN (SELECT term_id FROM wp_terms); 

DELETE FROM wp_term_relationships WHERE term_taxonomy_id not IN (SELECT term_taxonomy_id FROM wp_term_taxonomy); 

// DELETE PINGBACKS
DELETE FROM wp_comments WHERE comment_type = 'pingback'; 

// DELETE TRACKBACKS
DELETE FROM wp_comments WHERE comment_type = 'trackback';

// DELETE ORPHAN POST META, CHECK DB SIZE BEOFRE RUNNING
DELETE pm FROM wp_postmeta pm LEFT JOIN wp_posts wp ON wp.ID = pm.post_id WHERE wp.ID IS NULL;

// COPY TABLE WITH SCHEMA
CREATE TABLE wp_options_bak_2 LIKE wp_options;

// COPY TABLE CONTENT
INSERT INTO wp_options_bak_2 SELECT * FROM wp_options;

// SWITCH TABLE
RENAME TABLE wp_options TO wp_options_backup,
             wp_options_copy TO wp_options;
             
SELECT `option_name`, `option_value`, COUNT(`option_value`) FROM wp_options_bak_2 GROUP BY `option_value` HAVING COUNT(`option_value`) > 1;

// SORT BY WP OPTIONS LENGTH
SELECT * FROM `wp_options` ORDER BY LENGTH(`wp_options`.`option_value`) DESC

// CHECK DUPLICATE POST TITLES
SELECT post_title, COUNT(post_title), `post_type` FROM wp_posts where post_status = 'publish' GROUP BY post_title HAVING COUNT(post_title) > 1;

SELECT post_title, COUNT(post_title) FROM wp_posts where post_status = 'publish' GROUP BY post_title HAVING COUNT(post_title) > 1

SELECT ID, post_title, COUNT(post_title) FROM wp_posts where post_status = 'publish' AND post_type = 'shop_coupon' GROUP BY post_title HAVING COUNT(post_title) > 1

// SPACIAL OCATION
SELECT t1 FROM wp_posts t1 INNER JOIN wp_posts t2  WHERE  t1.id < t2.id AND  t1.email = t2.email;
DELETE t1 FROM wp_posts t1 INNER JOIN wp_posts t2  WHERE  t1.id < t2.id AND  t1.email = t2.email;

// FIND DUPLICATE TITLE
SELECT * FROM wp_posts AS t1 INNER JOIN wp_posts AS t2 WHERE t1.ID > t2.ID AND t1.post_title = t2.post_title;

// FIND DUPLICATE COUPONS
SELECT * FROM wp_posts c1 INNER JOIN wp_posts c2 WHERE c1.ID = c2.ID AND c1.`post_title` = c2.`post_title` AND c1.`post_type` = 'shop_coupon' GROUP BY c1.`post_title` ASC;

// DELETE DUPLICATE COUPONS
DELETE c1 from wp_posts c1 inner join wp_posts c2 where c1.Id < c2.Id and c1.`post_title` = c2.`post_title` and c1.`post_type` = 'shop_coupon' ORDER by c1.`post_title` asc;


SELECT `post_type`, COUNT(*) FROM `wp_posts` WHERE ID is not null GROUP by post_type;

// DELETE OLD ORDERS
DELETE FROM wp_woocommerce_order_items WHERE order_id IN ( SELECT ID FROM wp_posts WHERE post_date <= '2023-01-01');

DELETE FROM wp_postmeta WHERE post_id IN ( SELECT ID FROM wp_posts WHERE post_type = 'shop_order' AND post_date <= '2023-01-01');

DELETE FROM wp_posts WHERE post_type = 'shop_order' AND post_date <= '2023-01-01';

wp_usermeta
DELETE FROM wp_usermeta WHERE post_id NOT IN ( SELECT ID FROM wp_users);

DELETE wp_commentmeta FROM wp_commentmeta LEFT JOIN wp_comments ON wp_comments.comment_ID = wp_commentmeta.comment_id WHERE wp_comments.comment_ID IS NULL;

DELETE wp_postmeta FROM wp_postmeta LEFT JOIN wp_posts ON wp_posts.ID = wp_postmeta.post_id WHERE wp_posts.ID IS NULL;

DELETE FROM wp_usermeta
WHERE NOT EXISTS (
  SELECT * FROM wp_users
    WHERE wp_usermeta.user_id = wp_users.ID
)
