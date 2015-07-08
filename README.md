# ADB-Backup-APK-Injection
Android ADB backup APK Injection POC

What is ADB backup/restore?
---------------------------
The Android Operating System offers a backup/restore mechanism of installed packages through the ADB utility.
By default, full backup of applications including the private files stored in /data is performed, but this behaviour can be customized by implementing a BackupAgent (http://developer.android.com/reference/android/app/backup/BackupAgent.html) class. This way applications can feed the backup process with custom files and data.

APK Injection issue
-------------------



CVE
---
The ID CVE-2014-8757 was assigned to this vulnerability.

Affected Versions
-----------------
As of today, 2015-07-08, all Android versions are affected, including Android L.

Timeline
--------
The most important milestones of the issue are listed in the next lines:

2014-07-14: The vulnerability was disclosed to the Android security team
2014-07-28: Google refused to treat the issue as a potential threat, but requests to hold off publishing it
2014-10-15: Google answered that it did not get fixed in the L release and keeps requesting to hold off publishing it
2015-06-02: Google promised further info in a few days, but it never arrived
