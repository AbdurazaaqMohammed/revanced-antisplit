# revanced-antisplit
ReVanced Manager modified to work with APK files that have been converted from a split APK to a regular APK

# Why
In order to install an APK on Android, it must first be signed, although there are workarounds for rooted devices. ReVanced Manager uses a library made by Google called [apksig](https://android.googlesource.com/platform/tools/apksig) to sign APKs, regardless of the user's root status.

This library performs several types of validation when signing APKs. One of them produces an error like the following when attempting to sign APK files that have been converted from a split APK to a regular APK using a tool like [REAndroid APKEditor](https://github.com/REAndroid/APKEditor), AntiSplit M which is based on it or Apktool M.
```
Caused by: com.android.apksig.zip.ZipFormatException: Data Descriptor presence mismatch between Local File Header and Central Directory for entry AndroidManifest.xml. LFH: true, CD: false
        at com.android.apksig.internal.zip.LocalFileRecord.getRecord(LocalFileRecord.java:180)
        at com.android.apksig.internal.zip.LocalFileRecord.outputUncompressedData(LocalFileRecord.java:427)
        at com.android.apksig.internal.zip.LocalFileRecord.getUncompressedData(LocalFileRecord.java:451)
        at com.android.apksig.ApkSigner.getAndroidManifestFromApk(ApkSigner.java:966)
        at com.android.apksig.ApkSigner.getMinSdkVersionFromApk(ApkSigner.java:1004)
```

APKs merged in this way are **not corrupt**. They will install and work perfectly. There are **no** confirmed cases on the internet of this exception being thrown for an APK file that is actually corrupt. Either the validation apksig uses to throw this error is incorrect or the mismatch in fact does not matter and no exception should be thrown.

If the condition for this exception being thrown is removed from the code of ReVanced Manager, it will patch these merged APKs perfectly with no error, and they will install and work perfectly. This problem can also be solved by using a different method of signing or allowing users to save the patched APK if an error happened at signing, as I [reported](https://github.com/ReVanced/revanced-manager/issues/2083) to ReVanced, but it seems at least one collaborator is not interested in fixing this problem on their end.

Reporting this problem to Google is not really the solution here, because, as I explained in the thread, there are other real forms of validation that *should* be present in apksig but not in ReVanced Manager, at least not in their current state. Namely, CRC validation is done to ensure the validity of files inside the APK. Normally invalid CRCs indicate an invalid archive, however, an APK with invalid CRCs can be installed and ran perfectly fine. This is important because some apps (example: apps with Google Pairip Integrity protection) verify the validity of the app by checking the CRCs of files in its own APK, and some of these protections have been bypassed before by spoofing CRCs.  For the purposes of modifying APKs, ReVanced patches should at least have the option to ignore CRC checks so apps with this kind of protection can be successfuly modified.

With these factors in mind, I simply modified ReVanced Manager to stop this exception from being thrown and uploaded the APK [here](https://github.com/AbdurazaaqMohammed/revanced-antisplit/releases). Note that the signature is different to that used by ReVanced, so if you have ReVanced Manager already installed, you will have to uninstall it to install this version.
