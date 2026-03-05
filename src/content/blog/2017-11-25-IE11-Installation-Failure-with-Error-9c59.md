---
layout: '[post]'
title: IE11 Installation Failure with Error Code 9c59
pubDate: 2017-11-25 22:26:04
tags:
- Internet Explorer


---
In this blog, I will document how to solve a typical scenario of IE 11 install/upgrade failure.
<!-- more -->
# Background
Since IE 11 is the only supported IE version on Win 7 SP1 and Win Server 2008 R2 currently, you have to update IE to latest version to get security and feature updates. However, IE is not a standalone application which can be installed, uninstalled or upgraded easily.
You might already have the [official IE 11 installation package](https://support.microsoft.com/en-us/help/17621/internet-explorer-downloads), but sometime the installation just failed as below:
![image001.jpg](/images/2017-11-25-IE11-Installation-Failure-with-Error-9c59/image001.jpg)

# Logs
There are some logs can help you to dig into this error:
*   %windir%\IE11_main.log
*   %windir%\Logs\CBS\CBS.log
*   %windir%\Logs\DISM\DISM.log

# A typical failure with error code 9c59 in IE11_main.log
Probably 50% of IE 11 installation failure has a similar pattern in IE11_mail.log as below:
{% codeblock %}
04:36.262: INFO:    Waiting for Active Setup to complete. ({A509B1A8-37EF-4b3f-8CFC-4F3A74704073})
04:41.269: INFO:    Waiting for Active Setup to complete. ({A509B1A8-37EF-4b3f-8CFC-4F3A74704073})
04:46.277: INFO:    Waiting for Active Setup to complete. ({A509B1A8-37EF-4b3f-8CFC-4F3A74704073})
04:51.285: INFO:    Waiting for Active Setup to complete. ({A509B1A8-37EF-4b3f-8CFC-4F3A74704073})
04:56.292: INFO:    Waiting for Active Setup to complete. ({A509B1A8-37EF-4b3f-8CFC-4F3A74704073})
05:01.331: INFO:    PauseOrResumeAUThread: Successfully resumed Automatic Updates.
05:12.704: INFO:    Setup exit code: 0x00009C59 (40025) - The neutral cab failed to install.
05:12.704: INFO:    Cleaning up temporary files in: C:\Windows\TEMP\IE17934.tmp
05:12.766: INFO:    Unable to remove directory C:\Windows\TEMP\IE17934.tmp, marking for deletion on reboot.
05:12.766: INFO:    Released Internet Explorer Installer Mutex
{% endcodeblock %}
The setup exit code: `0x00009C59 (40025)` means [USER_ERROR_NEUTRAL_CAB_INSTALL_FAILED](https://docs.microsoft.com/en-us/internet-explorer/ie11-deploy-guide/setup-problems-with-ie11)

Then you might see a similar error pattern in CBS.log as below:
{% codeblock %}
2017-11-24 14:07:39, Info                  CBS    Exec: Staging Package: Package_3_for_KB2706045~31bf3856ad364e35~amd64~~6.1.1.0, Update: 2706045-8_neutral_GDR, PinDeployment: amd64_695c405833e475d1d5937b41a3983d92_31bf3856ad364e35_8.0.7601.17866_none_f28003cc92801a49
2017-11-24 14:07:40, Info                  CBS    Failed to find file: amd64_microsoft-windows-scripting-vbscript_31bf3856ad364e35_6.1.7600.21238_none_a51187a7608a895d\vbscript.dll [HRESULT = 0x80070002 - ERROR_FILE_NOT_FOUND]
2017-11-24 14:07:40, Info                  CBS    Failed to gather all required files. [HRESULT = 0x80070002 - ERROR_FILE_NOT_FOUND]
2017-11-24 14:07:40, Info                  CBS    Failed to gather all missing files for package: Microsoft-Windows-InternetExplorer-Package-TopLevel~31bf3856ad364e35~amd64~~11.2.9600.16428 [HRESULT = 0x80070002 - ERROR_FILE_NOT_FOUND]
2017-11-24 14:07:40, Info                  CSI    00000016@2017/11/24:06:07:40.030 CSI Transaction @0xf8020 destroyed
2017-11-24 14:07:40, Error                 CBS    Failed to pre- stage package: Microsoft-Windows-InternetExplorer-Package-TopLevel~31bf3856ad364e35~amd64~~11.2.9600.16428 [HRESULT = 0x80070002 - ERROR_FILE_NOT_FOUND]
2017-11-24 14:07:40, Info                  CBS    Perf: Stage chain complete.
2017-11-24 14:07:40, Info                  CBS    Failed to stage execution chain. [HRESULT = 0x80070002 - ERROR_FILE_NOT_FOUND]
2017-11-24 14:07:40, Error                 CBS    Failed to process single phase execution. [HRESULT = 0x80070002 - ERROR_FILE_NOT_FOUND]
2017-11-24 14:07:40, Info                  CBS    WER: Generating failure report for package: Microsoft-Windows-InternetExplorer-Package-TopLevel~31bf3856ad364e35~amd64~~11.2.9600.16428, status: 0x80070002, failure source: Stage, start state: Resolved, target state: Installed, client id: DISM Package Manager Provider
2017-11-24 14:07:40, Info                  CBS    Failed to query DisableWerReporting flag.  Assuming not set... [HRESULT = 0x80070002 - ERROR_FILE_NOT_FOUND]
2017-11-24 14:07:40, Info                  CBS    Failed to add %windir%\winsxs\pending.xml to WER report because it is missing.  Continuing without it...
2017-11-24 14:07:40, Info                  CBS    Failed to add %windir%\winsxs\pending.xml.bad to WER report because it is missing.  Continuing without it...
2017-11-24 14:07:40, Info                  CBS    Reboot mark refs: 0
2017-11-24 14:07:40, Info                  CBS    SQM: Reporting package change for package: Microsoft-Windows-InternetExplorer-Package-TopLevel~31bf3856ad364e35~amd64~~11.2.9600.16428, current: Resolved, pending: Default, start: Resolved, applicable: Installed, target: Installed, limit: Installed, hotpatch status: StillGoing, status: 0x0, failure source: Stage, reboot required: False, client id: DISM Package Manager Provider, initiated offline: False, execution sequence: 1186, first merged sequence: 1186
2017-11-24 14:07:40, Info                  CBS    SQM: Upload requested for report: PackageChangeBegin_Microsoft-Windows-InternetExplorer-Package-TopLevel~31bf3856ad364e35~amd64~~11.2.9600.16428, session id: 142859, sample type: Standard
2017-11-24 14:07:40, Info                  CBS    SQM: Ignoring upload request because the sample type is not enabled: Standard
2017-11-24 14:07:40, Info                  CBS    SQM: Reporting package change completion for package: Microsoft-Windows-InternetExplorer-Package-TopLevel~31bf3856ad364e35~amd64~~11.2.9600.16428, current: Resolved, original: Resolved, target: Installed, status: 0x80070002, failure source: Stage, failure details: "(null)", client id: DISM Package Manager Provider, initiated offline: False, execution sequence: 1186, first merged sequence: 1186
2017-11-24 14:07:40, Info                  CBS    SQM: stage time performance datapoint is invalid. [HRESULT = 0x80070490 - ERROR_NOT_FOUND]
2017-11-24 14:07:40, Info                  CBS    SQM: execute time performance datapoint is invalid. [HRESULT = 0x80070490 - ERROR_NOT_FOUND]
2017-11-24 14:07:40, Info                  CBS    SQM: Failed to initialize Win SAT assessment. [HRESULT = 0x80040154 - Unknown Error]
2017-11-24 14:07:40, Info                  CBS    SQM: average disk throughput datapoint is invalid [HRESULT = 0x80040154 - Unknown Error]
2017-11-24 14:07:40, Info                  CBS    SQM: Upload requested for report: PackageChangeEnd_Microsoft-Windows-InternetExplorer-Package-TopLevel~31bf3856ad364e35~amd64~~11.2.9600.16428, session id: 142862, sample type: Standard
2017-11-24 14:07:40, Info                  CBS    SQM: Ignoring upload request because the sample type is not enabled: Standard
2017-11-24 14:07:40, Info                  CBS    Enabling LKG boot option
{% endcodeblock %}
The installation failed start at `Exec: Staging Package: Package_3_for_KB2706045~31bf3856ad364e35~amd64~~6.1.1.0.`
That means one package of KB2706045 is corrupted for some reasons.

## Solution
As the installation failed because of a outdated and corrupted package, we can try to remove that package by DISM command first.
1.  Run below command to get the package full name :
    ```
    > dism /online /get-packages |findstr KB2706045
    Package Identity : Package_3_for_KB2706045~31bf3856ad364e35~amd64~~6.1.1.0
    Package Identity : Package_for_KB2706045_SP1~31bf3856ad364e35~amd64~~6.1.1.0
    Package Identity : Package_for_KB2706045~31bf3856ad364e35~amd64~~6.1.1.0
    ```
2.  Then use DISM command to remove that top level package:
    ```
    > dism /online /remove-package /packagename:Package_for_KB2706045~31bf3856ad364e35~amd64~~6.1.1.0
    Deployment Image Servicing and Management tool
    Version: 6.1.7600.16385
 
    Image Version: 6.1.7600.16385
 
    Processing 1 of 1 - Removing package Package_for_KB2706045~31bf3856ad364e35~amd64~~6.1.1.0
    [==========================100.0%==========================]
    The operation completed successfully.

    ```
3.  If the above action finished with no error, please try to reinstall IE 11 again.
**Note**: It might fail again because there might be multiple corrupted files. If this happens, please redo the procedure for any additional package you might have according the CBS.log. Usually we saw there are 2-3 Packages corrupted. 

# More words
If you need an offline IE 11 full installation package, please follow Joji's steps:
*   [Create Internet Explorer 11 batch deployment package](http://joji.me/en-us/blog/create-internet-explorer-11-batch-deployment-package)

Also you can use [Internet Explorer Administration Kit (IEAK)](https://technet.microsoft.com/en-us/microsoft-edge/bb219517.aspx) to customize IE 11 installation.

 

