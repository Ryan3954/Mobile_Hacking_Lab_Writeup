# Mobile Hacking Lab
Machine Name: Cyclic Scanner

#### APK Installation and Initial Behavior
- Upon installing and opening the APK on an Android device, the app prompts for permission to access all files. If this permission is not granted, the application throws a `"Permission Denied"` error.
![screenshot](https://imgur.com/XVZxdGP.png)
- Once the permission is granted, the app launches normally and displays its user interface.
![screenshot](https://imgur.com/8u1pFIo.png)
- When the scanner is enabled, the application displays the message:
`"Scan service started, your device will be scanned regularly."`
![screenshot](https://imgur.com/lvUU46F.png)

#### Source Code Analysis
- To understand the internal workings of the app, I decompiled the APK using `JADX GUI`. I started by reviewing the `AndroidManifest.xml` and navigated to the `MainActivity` class.
![screenshot](https://imgur.com/eEhxYje.png)
- In the `MainActivity`, around lines `90–92`, I noticed a reference to a class named `ScanService`, which seemed interesting.
![screenshot](https://imgur.com/vxYK5Ad.png)
- Opening the `ScanService` class, I examined how the application scans files.
![screenshot](https://imgur.com/VUO6G8f.png)

#### ScanService Functionality
- The ScanService class is responsible for scanning external storage (commonly the `sdcard` directory). It identifies all files present and sends each one to the `ScanEngine` class. This scan is performed repeatedly every 6 seconds.
![screenshot](https://imgur.com/VUO6G8f.png)
- To further investigate, I analyzed the `ScanEngine` class.
![screenshot](https://imgur.com/KiwRVzf.png)

#### ScanEngine Vulnerability
- Within `ScanEngine`, each file received from `ScanService` is processed by retrieving its absolute path. That path is then directly inserted into a system command without any input sanitization, potentially leading to `Remote Code Execution` (RCE).
![screenshot](https://imgur.com/3V7XVMa.png)
![screenshot](https://imgur.com/hwEe3ch.png)
- To test this vulnerability, I attempted to exploit it.

#### Proof of Concept (PoC)
- Navigating to the /sdcard/Download directory, I created a file named:
`test; touch RCE_POC.txt`
![screenshot](https://imgur.com/421w1Dv.png)
- If the RCE works, the system should execute the second part of the command (touch RCE_POC.txt) and create that file.
### Result:
- The file named test; touch RCE_POC.txt was successfully created.
![screenshot](https://imgur.com/wX3Gqxm.png)
- A new file named RCE_POC.txt appeared outside the Download directory.
![screenshot](https://imgur.com/GqMxo4I.png)
- This confirms that the command injection and RCE vulnerability are present and exploitable.