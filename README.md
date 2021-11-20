# Wordpress Docker
### Installation
1. Install docker and docker compose onto the host system.
2. run `docker-compose up -d` to start up the wordpress stack


### General
The database file in `db` will be used to seed the MySQL database.
By default wp-content folder is synced from repo into docker container.
Credentials for db are stored in docker-compose.yml for now
TODO: move these creds to env variables.

To bring the wp and db down run `docker-compose down`
To bring down the stack and remove the volumes run `docker-compose down -v`


### Site Migration
Sometimes you move the site to another url or port or domain or whatever.
You will need to update the site url in a couple of places

Enter the db container with `docker exec -it <container> /bin/bash`
Then enter mysql command prompt `mysql -u[MYSQL_USER] -p`

Common Commands:
  * `show databases;`: show the mysql databases
  * `use <database>;`: switch to specific database

In the DB...
Run the following queries

  * `UPDATE wp_options SET option_value = replace(option_value, '<oldsite>', '<new_url>') WHERE option_name = 'home' OR option_name = 'siteurl';`
  * `UPDATE wp_posts SET guid = replace(guid, 'http://localhost','http://localhost:8000');`
  * `UPDATE wp_posts SET post_content = replace(post_content, 'http://localhost', 'http://localhost:8000');`
  * `UPDATE wp_postmeta SET meta_value = replace(meta_value,'http://localhost','http://localhost:8000');`

and then in the docker-compose.yml change the site url and such to match.
Run docker-compose up -d again to make the changes take effect.


### Database Backup
To run a database backup 
`mysqldump --skip-comments --skip-extended-insert -u[MYSQL_USER] -p[MYSQL_PASSWORD] > backup.`date +%F`.sql`
from inside the mysql container.

From the host run 
`docker exec [MYSQL_CONTAINER] /usr/bin/mysqldump -u [MYSQL_USER] --password=[MYSQL_PASSWORD] [MYSQL_DATABASE] > backup.sql`

These dumps can be diffed with other users if the db remains small enough.
