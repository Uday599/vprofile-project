FROM mysql:5.7.25
LABEL "project"="vprofile"
LABEL "Author"="Uday"
ENV MYSQL_ROOT_PASSWORD="vprodbpass"
ENV MYSQL_DATABASE="accounts"

# we will add our customized schema file to default path in container
ADD db_backup.sql docker-entrypoint-initdb.d/db_backup.sql 
