# WordPress Useful Queries

Follows a list of useful queries to handle WordPress database related tasks.

## Change Siteurl & Homeurl

WordPress stores the absolute path of the site URL and home URL in the database. Therefore, if you transfer your WordPress site from the localhost to your server, your site will not load online. This is because the absolute path URL is still pointing to your localhost. You will need to change the site URL and the home URL in order for the site to work.

```SQL
UPDATE wp_options SET option_value = replace(option_value, 'http://www.oldsiteurl.com', 'http://www.newsiteurl.com') WHERE option_name = 'home' OR option_name = 'siteurl';
```

## Change GUID

After you have migrated your blog from the localhost to your server or from another domain to a new domain, you will need to fix the URLs for the GUID field in wp_posts table. This is crucial because GUID is used to translate your post or page slug to the correct article absolute path if it is entered wrongly.

```SQL
UPDATE wp_posts SET guid = REPLACE (guid, 'http://www.oldsiteurl.com', 'http://www.newsiteurl.com');
```

## Change URL in Content

WordPress uses absolute path in the URL link instead of a relative path in the URL link when storing them in the database. Within the content of each post record, it stores all the old URLs referencing the old source. Therefore you will need to change all these URLs to the new domain location.

```SQL
UPDATE wp_posts SET post_content = REPLACE (post_content, 'http://www.oldsiteurl.com', 'http://www.newsiteurl.com');
```

## Change Image Path Only

If you decide to use Amazon CloudFront as your Content Delivery Network (CDN) to offload the delivery of images from your server. After your have created your CNAME record, you can use the query below to change all the image paths in WordPress to load all your images from Amazon CloudFront.

```SQL
UPDATE wp_posts SET post_content = REPLACE (post_content, 'src="http://www.oldsiteurl.com', 'src="http://yourcdn.newsiteurl.com');
```

You will also need to update the GUID for Image Attachment with the following query:

```SQL
UPDATE wp_posts SET  guid = REPLACE (guid, 'http://www.oldsiteurl.com', 'http://yourcdn.newsiteurl.com') WHERE post_type = 'attachment';
```

## Update Post Meta

Updating Post Meta works almost the same way as updating the URL in post content. If you have stored extra URL data for each post, you can use the follow query to change all of them.

```SQL
UPDATE wp_postmeta SET meta_value = REPLACE (meta_value, 'http://www.oldsiteurl.com','http://www.newsiteurl.com');
```

## Change Default "Admin" Username

Every default WordPress installation will create an account with a default Admin username. This is wide spread knowledge, everyone who uses WordPress knows this. However, this can be a security issue because a hacker can brutal force your WordPress admin panel. If you can change your default “Admin” username, you will give your WordPress admin panel additional security.

```SQL
UPDATE wp_users SET user_login = 'Your New Username' WHERE user_login = 'Admin';
```

## Change Password

Ever wanted to reset your password in WordPress, but cannot seem to use the reset password section whatever the reason?

```SQL
UPDATE wp_users SET user_pass = MD5( 'new_password' ) WHERE user_login = 'your-username';
```

## Change Author

If you want to transfer the articles under Author B to merge with those under Author A, it will be very time consuming if you do it article by article. With the following SQL query, you can easily go through all the records and assign articles by Author B to go under Author A.

You will first need to obtain the author ID of both authors by going to your Author & User page in your WordPress admin panel. Click on the author’s name to view their profile. At the address bar, look for "user_id". That is the author ID information we require.

```SQL
UPDATE wp_posts SET post_author = 'new-author-id' WHERE post_author = 'old-author-id';
```

## Delete Revision

When you are editing an article in WordPress, there will be many revision copies being saved. This is a waste of resources because excessive revision records can increase the burden of the database. Over time, when you have thousands of entries, your database will have grown significantly. This will increase loop iterations, data retrieval and will affect the page loading time.

```SQL
DELETE a,b,c FROM wp_posts a
LEFT JOIN wp_term_relationships b ON (a.ID = b.object_id)
LEFT JOIN wp_postmeta c ON (a.ID = c.post_id)
WHERE a.post_type = 'revision';
```

## Delete Post Meta

Installing or removing plugins is a very common task for WordPress. Some of the plugins make use of the post meta to store data pertaining to the plugin. After you have removed the plugin, those data are still left inside the post_meta table, which will no longer be needed. Run the following query to clean up the unused post meta value. This will help to speed up and reduce the size of your database.

```SQL
DELETE FROM wp_postmeta WHERE meta_key = 'your-meta-key';
```

## Export all Comment Emails with no Duplicate

Over a period of time, your blog will have received many comments. These comments will include the email addresses left by the commenter. You can retrieve all these emails for your mailing list without any duplicate.

```SQL
SELECT DISTINCT comment_author_email FROM wp_comments;
```

## Delete all Pingback

Popular articles receive plenty of pingback. When this happens, the size of your database increases. In order to reduce size of the database, you can try removing all the pingbacks.

```SQL
DELETE FROM wp_comments WHERE comment_type = 'pingback';
```

## Delete all Spam Comments

If you have plenty of spam comments, going through each page to delete spam can be tedious and frustrating. With the following SQL query, even if you have to face deleting 500 over spam comments, it will be a breeze.

```SQL
DELETE FROM wp_comments WHERE comment_approved = 'spam';
```

* 0 = Comment Awaiting Moderation
* 1 = Approved Comment
* spam = Comment marked as Spam

## Identify Unused Tags

In a WordPress database, if you run a query to delete old posts manually from MySQL, the old tags will remain and appear in your tag cloud/listing. This query allows you to identify the unused tags.

```SQL
SELECT * From wp_terms wt
INNER JOIN wp_term_taxonomy wtt ON wt.term_id=wtt.term_id WHERE wtt.taxonomy='post_tag' AND wtt.count=0;
```
