---
layout: post
title:  "back up mysql"
categories: MySQL
---

```bash
#! /bin/bash
BACKUP_PATH=/var/backup/db

# (1) set up all the mysqldump variables
DATE=`date +"%m%d%H%M"`
SQLFILE=${BACKUP_PATH}/db_backup_${DATE}.sql
DATABASE=<database_name>
USER=root
PASSWORD=<db_user_password>

# (2) in case you run this more than once a day,
# remove the previous version of the file
unalias rm     2> /dev/null
rm ${SQLFILE}     2> /dev/null
rm ${SQLFILE}.gz  2> /dev/null

# (3) do the mysql database backup (dump)
mysqldump -u ${USER} -p${PASSWORD} ${DATABASE} | gzip > ${SQLFILE}.gz


#This will delete backups older than 7 days.
find ${BACKUP_PATH} -mtime +7 -exec rm {} \;
```
