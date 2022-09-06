# Auto-backup near node script

Create new script file in script directory

```bash
sudo apt-get install zip
cd scripts
nano autobackup.sh
```

```bash
#!/bin/bash
DATE=$(date +%Y-%m-%d)
DATADIR=/home/root/.near
BACKUPS_DIR=/home/root/backups
BACKUP_DATA=${BACKUPS_DIR}/near_${DATE}
SERVICE_NEAR_NAME=neard.service
NEW_BACKUP_FILE=$BACKUP_DATA.zip
# create dir if not exists
if [ ! -d "$BACKUPS_DIR" ]; then
  echo "Create directory $BACKUPS_DIR" | ts
  mkdir $BACKUPS_DIR
fi
# delete old backup
OLD_BACKUPS=`ls $BACKUPS_DIR/*.zip`
for file in $OLD_BACKUPS
  do
     rm $file
     if [ ! -d "$file" ]; then
       echo "$file has been deleted" | ts
     fi
  done
sudo systemctl stop $SERVICE_NEAR_NAME
wait
echo "New backup started" | ts
zip -r $BACKUP_DATA $DATADIR
echo "Backup completed" | ts
sudo systemctl start $SERVICE_NEAR_NAME
# check file
if [ -f $NEW_BACKUP_FILE ]; then
   curl -s -X POST $TG_URL -d chat_id=$CHAT_ID -d parse_mode=markdown \
         -d text="The node backup was created. New File: $NEW_BACKUP_FILE "
else
   curl -s -X POST $TG_URL -d chat_id=$CHAT_ID -d text="Some problems with new backup. Backup not created!"
fi
```

Add exacutable permissions

```bash
chmod +x autobackup.sh
```

To run script

```bash
sudo ./scripts/autobackup.sh
```

For automatic start add task to crontab:

```bash
sudo nano /etc/crontab
```

```bash
# Example of job definition:
# .---------------- minute (0 - 59)
# |  .------------- hour (0 - 23)
# |  |  .---------- day of month (1 - 31)
# |  |  |  .------- month (1 - 12) OR jan,feb,mar,apr ...
# |  |  |  |  .---- day of week (0 - 6) (Sunday=0 or 7) OR sun,mon,tue,wed,thu,fri,sat
# |  |  |  |  |
# *  *  *  *  * user-name command to be executed
0  12 *  *  * near      <WORK_DIR>/NEARmain/backup.sh >> <WORK_DIR>/NEARmain/backups/backup.log 2>&1
```
