<pre>
USE master;  
GO
--CHECK MASTER KEY
SELECT name ,is_master_key_encrypted_by_server 
FROM sys.databases;
GO

--CREATE MASTER KEY
/*
CREATE MASTER KEY ENCRYPTION BY PASSWORD = 'YourPassword';
GO
*/
</pre>
<pre>
USE master;  
GO 
--CHECK CERTIFICATE
SELECT name ,subject,start_date,expiry_date,
pvt_key_encryption_type_desc,pvt_key_last_backup_date
FROM sys.certificates
WHERE pvt_key_encryption_type='MK';
GO

--CREATE CERTIFICATE
/*
CREATE CERTIFICATE YourServerCert WITH SUBJECT = 'YourDB DEK Certificate';  
GO  
*/
</pre>
<pre>
--BACKUP CERTIFICATE
USE master;
GO
/* 
BACKUP CERTIFICATE YourServerCert
TO FILE='localpath\YourServerCert'
WITH PRIVATE KEY(FILE='localpath\YourServerCertPrik',ENCRYPTION BY PASSWORD='YourPassword')
GO
*/

--IMPORT CERTIFICATE
USE master;
GO
/*
--After CREATE CERTIFICATE
CREATE CERTIFICATE YourServerCert 
FROM FILE='localpath\YourServerCert'
WITH PRIVATE KEY(FILE='localpath\YourServerCertPrik',ENCRYPTION BY PASSWORD='YourPassword')
GO
*/
</pre>
<pre>
USE master;  
GO
--CHECK TDE
SELECT DB_NAME(database_id) AS DatabaseName, encryption_state,
encryption_state_desc =
CASE encryption_state
         WHEN '0'  THEN  'No database encryption key present, no encryption'
         WHEN '1'  THEN  'Unencrypted'
         WHEN '2'  THEN  'Encryption in progress'
         WHEN '3'  THEN  'Encrypted'
         WHEN '4'  THEN  'Key change in progress'
         WHEN '5'  THEN  'Decryption in progress'
         WHEN '6'  THEN  'Protection change in progress (The certificate or asymmetric key that is encrypting the database encryption key is being changed.)'
         ELSE 'No Status'
         END,
percent_complete,encryptor_thumbprint, encryptor_type  FROM sys.dm_database_encryption_keys;

--CREATE TDE
/*
USE YourDB;  
GO  
CREATE DATABASE ENCRYPTION KEY  
WITH ALGORITHM = AES_128  
ENCRYPTION BY SERVER CERTIFICATE YourServerCert;  
GO  
ALTER DATABASE YourDB  
SET ENCRYPTION ON;  
GO 
*/
</pre>
<pre>
--SET ENCRYPTION OFF
USE master;  
GO 
/*
ALTER DATABASE YourDB  
SET ENCRYPTION OFF;  
GO  
*/

--'''Wait''' for decryption operation to complete, look for a value of  1 in the query below.
--Keep CHECK TDE till encryption_state changed
SELECT encryption_state  
FROM sys.dm_database_encryption_keys;  
GO

--DROP ENCRYPTION KEY
/*
USE YourDB;  
GO  
DROP DATABASE ENCRYPTION KEY;  
GO  
*/
</pre>
