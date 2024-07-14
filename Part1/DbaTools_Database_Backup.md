# Backup SQL Database Script

```powershell
# Install and import the dbatools module
Install-Module -Name dbatools -Force -AllowClobber
Import-Module dbatools

# Define the server instance and user credentials
$serverInstance = "LAPTOP-UO4NL291"
$username = "SqlAuthenticationUsername"
$password = "UserAuthentication password"

# Convert the password to a secure string and create a PSCredential object
$securePassword = ConvertTo-SecureString $password -AsPlainText -Force
$credential = New-Object System.Management.Automation.PSCredential ($username, $securePassword)

# Define a connection string with TrustServerCertificate=True
$connectionString = "Server=$serverInstance;User Id=$username;Password=$password;TrustServerCertificate=True;"

# Define the master database and query to list all databases
$database = "master"
$query = "SELECT name FROM sys.databases"

# Execute the query using Invoke-Sqlcmd with the connection string
$result = Invoke-Sqlcmd -ConnectionString $connectionString -Database $database -Query $query

# Display the result
$result

# Define the database to backup and the backup directory
$databaseToBackup = "Alerts"
$backupDirectory = "C:\Program Files\Microsoft SQL Server\MSSQL16.MSSQLSERVER\MSSQL\Backup"

# Perform a full database backup with IgnoreFileChecks
Backup-DbaDatabase -SqlInstance $connectionString -Database $databaseToBackup -Path $backupDirectory -Type Full -SqlCredential $credential -IgnoreFileChecks

# Perform a transaction log backup with IgnoreFileChecks
Backup-DbaDatabase -SqlInstance $connectionString -Database $databaseToBackup -Path $backupDirectory -Type Log -SqlCredential $credential -IgnoreFileChecks
