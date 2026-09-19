```
AZURE STORAGE PRACTICE - STEP BY STEP


1. STORAGE ACCOUNT

1. Login to Azure Portal.
2. Search "Storage Accounts".
3. Click Create.
4. Select Subscription.
5. Select/Create Resource Group.
6. Enter Storage Account Name.
7. Select Region.
8. Select Redundancy.
9. Click Review + Create.
10. Click Create.
11. Open the Storage Account.


==================================================

2. AZURE FILE SHARE

1. Open Azure Portal.
2. Create/Select Storage Account.
3. Go to Data Storage.
4. Select File Shares.
5. Click + File Share.
6. Enter File Share Name:

   myfileshare

7. Click Create.
8. Open myfileshare.
9. Click Connect.
10. Select Windows.
11. Select PowerShell.
12. Select a Drive Letter.
13. Copy the generated PowerShell script.


==================================================

3. CREATE WINDOWS SERVER 2025 VM

1. Open Azure Portal.
2. Search "Virtual Machines".
3. Click Create.
4. Select Azure Virtual Machine.
5. Select Resource Group.
6. Enter VM Name.
7. Select Windows Server 2025 image.
8. Select VM Size.
9. Enter Username.
10. Enter Password.
11. Allow RDP Port 3389.
12. Click Review + Create.
13. Click Create.
14. Wait for deployment.
15. Open the VM.
16. Click Connect.
17. Select RDP.
18. Download/Open RDP file.
19. Enter Username and Password.
20. Connect to Windows Server.


==================================================

4. CONNECT AZURE FILE SHARE TO WINDOWS VM

1. RDP into Windows Server 2025.
2. Search for PowerShell.
3. Right-click PowerShell.
4. Select "Run as Administrator".
5. Paste the PowerShell script copied from Azure File Share.
6. Press Enter.
7. Wait for the connection to complete.
8. Open File Explorer.
9. Select "This PC".
10. Verify the mapped Azure File Share drive.
11. Open the mapped drive.
12. Create a test file.

Example:

test.txt

13. Go back to Azure Portal.
14. Open Storage Account.
15. Open File Shares.
16. Open myfileshare.
17. Verify test.txt is available.


==================================================

5. AZURE FILE SHARES - CLASSIC

1. Open Azure Portal.
2. Open Storage Account.
3. Go to Data Storage.
4. Select File Shares / File Shares Classic.
5. Click + File Share.
6. Enter File Share Name.
7. Click Create.
8. Open the File Share.
9. Click Connect.
10. Select Windows.
11. Copy the PowerShell connection script.
12. RDP into Windows VM.
13. Open PowerShell as Administrator.
14. Paste the script.
15. Press Enter.
16. Open File Explorer.
17. Select This PC.
18. Verify the mapped drive.


==================================================

6. AZURE MANAGED DISK

1. Open Azure Portal.
2. Search "Virtual Machines".
3. Select your Windows VM.
4. Go to Disks.
5. Select "Create and attach a new disk".
6. Enter Disk Name.
7. Select Disk Type.
8. Select Disk Size.
9. Click Apply/Save.
10. RDP into Windows VM.
11. Search "Disk Management".
12. Open Disk Management.
13. Find the newly attached disk.
14. Initialize the disk.
15. Select GPT.
16. Right-click Unallocated Space.
17. Select "New Simple Volume".
18. Click Next.
19. Select Volume Size.
20. Assign Drive Letter.
21. Select NTFS.
22. Format the disk.
23. Click Finish.
24. Open File Explorer.
25. Verify the new data disk.


==================================================

7. STORAGE TYPES - POINTS TO REMEMBER

Blob Storage
- Type: Object Storage
- Used for application/web data.
- Used for images.
- Used for videos.
- Used for backups.
- Used for logs.

Azure File Share
- Type: File Storage
- Used for shared storage.
- Supports file/folder structure.
- Can be mapped as a network drive.
- Commonly accessed using SMB from Windows.

Azure Managed Disk
- Type: Block / Volume Storage.
- Used with Azure Virtual Machines.
- Used as OS Disk.
- Used as Data Disk.
- Can be attached to a VM.


==================================================

8. AZURE STATIC WEBSITE HOSTING

Storage Account Name:

mystorageacco78687697696


STEP 1:

1. Login to Azure Portal.
2. Search "Storage Accounts".
3. Open:

   mystorageacco78687697696


STEP 2:

1. Inside Storage Account, search/select:

   Static Website

2. Click Enable.


STEP 3:

Configure:

Static Website:
Enabled

Index Document Name:
index.html

Error Document Path:
404.html

3. Click Save.


STEP 4:

Azure creates a special container:

$web


STEP 5:

Create index.html file.

Example:

<html>
<head>
    <title>Azure Website</title>
</head>
<body>
    <h1>Hello from Azure</h1>
    <p>Azure Static Website is working.</p>
</body>
</html>


STEP 6:

1. Open Storage Account.
2. Go to Storage Browser.
3. Select Blob Containers.
4. Open:

   $web

5. Click Upload.
6. Select index.html.
7. Click Upload.


STEP 7:

1. Go to Storage Account.
2. Select Static Website.
3. Copy the Primary Endpoint.
4. Open the endpoint in a web browser.

Example:

https://mystorageacco78687697696.z9.web.core.windows.net/

5. Verify index.html is displayed.


==================================================

9. QUICK REVISION

Blob Storage
Object Storage
Application / Web Data / Images / Videos / Backups


Azure File Share
File Storage
Shared Storage
SMB
Mapped Network Drive


Azure Managed Disk
Block / Volume Storage
OS Disk
Data Disk
Azure VM


Azure Static Website
Storage Account
        |
Static Website Enable
        |
$web
        |
index.html
        |
Primary Endpoint
        |
Website
```
