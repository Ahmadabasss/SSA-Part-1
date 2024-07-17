# SQL Server Log Shipping Configuration

Log shipping in SQL Server involves automatically sending transaction log backups from a primary database to one or more secondary databases. This helps maintain a backup server that can quickly take over in case the primary server fails.

## Components of Log Shipping

1. **Primary Server and Database**:
   - The source server where the primary database resides.
   - Transaction logs are backed up periodically.

2. **Secondary Server and Database**:
   - The destination server where the secondary database is kept in sync with the primary database.
   - Transaction log backups from the primary database are copied and restored here.

3. **Monitor Server (Optional)**:
   - This server monitors the log shipping process and can alert administrators of any issues.
   - It's optional but recommended for better monitoring and alerting.

## Steps to Configure Log Shipping

### Backup the Transaction Log on the Primary Server

Schedule a job to periodically back up the transaction log on the primary database. The backup file is saved to a network share accessible by the secondary server(s).

### Copy the Transaction Log Backup to the Secondary Server

Schedule a job on the secondary server to periodically copy the transaction log backup files from the network share.

### Restore the Transaction Log Backup on the Secondary Server

Schedule a job on the secondary server to restore the copied transaction log backups to the secondary database. The secondary database must be in a state that allows transaction log backups to be applied (either in a restoring state or standby mode).

## Configuring Log Shipping Using SQL Server Management Studio (SSMS)

1. **Enable Log Shipping**:
   - Right-click the database you want to configure log shipping for, select `Tasks`, then `Ship Transaction Logs`.
   - Check the box `Enable this as a primary database in a log shipping configuration`.

2. **Configure the Transaction Log Backup Settings**:
   - Set the backup schedule, destination folder, and retention period for the backups.

3. **Configure the Secondary Server**:
   - Add a secondary database and configure the settings for copying and restoring the transaction log backups.
   - Set the delay between restoring backups and the mode for restoring (No Recovery or Standby).

4. **Optional: Configure the Monitor Server**:
   - Add a monitor server to keep track of the log shipping status and configure alerts.

## Example: Setting Up Log Shipping via T-SQL

### 1. Backup the Primary Database

```sql
BACKUP DATABASE [YourPrimaryDatabase]
TO DISK = 'C:\Backup\YourPrimaryDatabase.bak'
WITH INIT;
