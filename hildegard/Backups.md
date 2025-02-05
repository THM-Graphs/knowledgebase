Die beiden Skripte zum Anlegen (`create_backup.sh`) und Wiederherstellen (`load_backup.sh`) der Backups liegen im `scripts`-Verzeichnis auf dem Server.

**Wichtig**: Um die Skripte ausführbar zu machen, die Execute-Berechtigungen setzen:

```sh
chmod +x create_backup.sh load_backup.sh
```

### Backup anlegen
Die Backups werden nicht manuell erzeugt, sondern automatisch per Cronjob täglich um 2 Uhr morgens. Um den Cronjob anzuzeigen, `crontab -l` in der Konsole eingeben, um ihn zu bearbeiten, `cronjob -e`. Sieht momentan in etwa so aus:

```sh
0 2 * * * /home/<username>/scripts/create_backup.sh >> /home/<username>/logs/backups.log 2>&1
```

Falls doch mal ein manuelle Backup erzeugt werden soll, hier der Befehl:

```sh
cd scripts
./create_backup.sh
```

Das Skript in `create_backup.sh` stoppt den Neo4j-Container, erzeugt eine `.dump`-Datei mit einem aktuellen Timestamp, legt sie im `backups`-Verzeichnis ab und startet den Container neu.

```sh
#!/usr/bin/env bash
HOME=/home/<username>
APP_DIR="$HOME/atag-editor"
BACKUP_DIR="$HOME/backups"

# Add separator for log file
echo -e "\n---"
echo "$(date '+%Y-%m-%d %H:%M:%S') Backup script started"

# Stop the database (required)
cd $APP_DIR

echo "Stopping database..."

docker compose -f docker-compose.prod.yml stop neo4j

if [ $? -eq 0 ]; then
    echo "Database stopped."
else
    echo "Error: Failed to stop the database."
    echo "---"
    exit 1
fi

# Create the backup directory if it doesn't exist
mkdir -p "$BACKUP_DIR"

# Generate the timestamped filename for the backup
TIMESTAMP=$(date +%Y%m%d_%H%M%S)
BACKUP_FILE="backup_hildegard_${TIMESTAMP}.dump"

echo "Creating backup..."

# Create the neo4j.dump file with a temporary neo4j-admin container and rename it
/usr/bin/docker run --interactive --rm \
   --volume=$APP_DIR/neo4j/data:/data \
   --volume=$BACKUP_DIR:/backups \
   neo4j/neo4j-admin:latest \
bash -c "neo4j-admin database dump neo4j --to-path=/backups && mv /backups/neo4j.dump /backups/${BACKUP_FILE}"

if [ $? -eq 0 ]; then
   echo "Backup created successfully: $BACKUP_FILE"
else
   echo "Error: Backup failed. The database may be in use. Please stop the database and try again."
fi

echo "Restarting database..."

# Restart the database
docker compose -f docker-compose.prod.yml up neo4j -d

echo "Database restarted."
echo "---"
```

Der Konsolen-Output landet in `~/logs/backups.log` und sieht ungefähr so aus:

```sh
---
2025-01-09 02:00:01 Backup script started
Stopping database...
time="2025-01-09T02:00:01+01:00" level=warning msg="/home/<username>/atag-editor/docker-compose.prod.yml: the attribute `version` is obsolete, it will be ignored, please remove it to avoid potential confusion"
 Container neo4j  Stopping
 Container neo4j  Stopped
Database stopped.
Creating backup...
2025-01-09 01:00:15.242+0000 INFO  [o.n.c.d.DumpCommand] Starting dump of database 'neo4j'

Files: 1/40, data:  0.0%
...
Files: 40/40, data: 100.0%
Done: 40 files, 309.2MiB processed.
2025-01-09 01:00:19.507+0000 INFO  [o.n.c.d.DumpCommand] Dump completed successfully
Backup created successfully: backup_hildegard_20250109_020007.dump
Restarting database...
time="2025-01-09T02:00:19+01:00" level=warning msg="/home/<username>/atag-editor/docker-compose.prod.yml: the attribute `version` is obsolete, it will be ignored, please remove it to avoid potential confusion"
 Container neo4j  Created
 Container neo4j  Starting
 Container neo4j  Started
Database restarted.
---
```

## Backup laden

Befehl:

```sh
cd scripts
./load_backup.sh <backup-file>
# Beispiel: ./load_backup.sh backup_hildegard_20241024_162412.dump
```

Das Skript in `load_backup.sh` stoppt den Neo4j-Container, spielt die mitgegebene `.dump`-Datei in das `data`-Verzeichnis der App ein und startet den Container neu.

```sh
#!/bin/bash

# Return if no filename parameter was provided
if [ "$#" -eq 0 ]; then
    echo "Please provide the name of the backup file to be restored."
    exit 1
fi

# Set filename, backup directory and temporary directory from where the dump will be loaded into the database
DUMP_FILENAME="$1"
APP_DIR="$HOME/atag-editor"
BACKUP_DIR="$HOME/backups"
TEMP_DIR="$BACKUP_DIR/temp"

# Stop the database (required)
cd $APP_DIR

echo "Stopping database..."

docker compose -f docker-compose.prod.yml stop neo4j > /dev/null 2>&1

if [ $? -eq 0 ]; then
    echo "Database stopped."
else
    echo "Error: Failed to stop the database."
    exit 1
fi

echo "Restoring backup..."

sudo chown -R <username> "$BACKUP_DIR"

# Create the temp directory and clean it
mkdir -p "$TEMP_DIR"
sudo rm -rf "$TEMP_DIR/*"

# Copy the given dump to the temp directory and rename it to neo4j.dump (default name, is required)
cp "$BACKUP_DIR/$DUMP_FILENAME" "$TEMP_DIR/"
mv "$TEMP_DIR/$DUMP_FILENAME" "$TEMP_DIR/neo4j.dump"

# Load dump into the mounted data folder
docker run --interactive --tty --rm \
    --volume="$APP_DIR/neo4j/data:/data" \
    --volume="$TEMP_DIR:/temp" \
    neo4j/neo4j-admin:latest \
    neo4j-admin database load neo4j --from-path=/temp --overwrite-destination > /dev/null 2>&1

# Check if the load command succeeded
if [ $? -eq 0 ]; then
    echo "Database restored successfully from $DUMP_FILENAME."
else
    echo "Error: Failed to load the database from $DUMP_FILENAME."
fi

# Finally: Remove the temp directory
rm -rf "$TEMP_DIR"

# Restart the database
docker compose -f docker-compose.prod.yml up neo4j -d > /dev/null 2>&1

echo "Database restarted."
```

