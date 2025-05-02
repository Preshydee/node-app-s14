
# 🔐 Server Backup 

## What Was Implemented

### 1. SSH

* Restricted server access to **SSH key authentication only**.
* Disabled all password-based logins (`PasswordAuthentication no`).
* SSH config was reloaded to apply changes.

**Result:** Only users with registered SSH keys can access the server. Passwords no longer work for SSH login.

---

### 2. Automated Daily Backups

A daily cron job was set up to back up both:

* Application files (`/var/www/myapp`)
* PostgreSQL database (`mydatabase`)

#### 🔧 Backup Script Overview (`/opt/scripts/backup.sh`)

The script:

* Creates a timestamped SQL dump of the database.
* Compresses app data and the SQL dump into one archive.
* Logs each backup event.

```bash
#!/bin/bash

TIMESTAMP=$(date +"%F-%H-%M")
BACKUP_DIR="/opt/backups"
APP_DIR="/var/www/myapp"
DB_NAME="mydatabase"
DB_USER="postgres"
SQL_DUMP="/tmp/${DB_NAME}-${TIMESTAMP}.sql"
ARCHIVE="${BACKUP_DIR}/backup-${TIMESTAMP}.tar.gz"
LOG_FILE="/var/log/backup.log"

mkdir -p "$BACKUP_DIR"
pg_dump -U "$DB_USER" "$DB_NAME" > "$SQL_DUMP"
tar -czf "$ARCHIVE" "$APP_DIR" "$SQL_DUMP"
rm "$SQL_DUMP"
echo "[$(date)] Backup created: $ARCHIVE" >> "$LOG_FILE"
```

#### 🕐 Cron Job

Scheduled to run daily at **2:00 AM**:

```
0 2 * * * /bin/bash /opt/scripts/backup.sh
```

---

### 3. Restore Process

To restore from a backup:

1. **Extract the archive:**

   ```bash
   tar -xzf backup-YYYY-MM-DD-HH-MM.tar.gz -C /tmp/
   ```

2. **Restore app data:**

   ```bash
   sudo cp -r /tmp/var/www/myapp /var/www/
   sudo chown -R www-data:www-data /var/www/myapp
   ```

3. **Restore the database:**

   ```bash
   createdb -U postgres mydatabase   # if not already present
   psql -U postgres mydatabase < /tmp/mydatabase-YYYY-MM-DD-HH-MM.sql
   ```
