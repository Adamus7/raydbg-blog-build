---
title: Configure Wired Authentication Settings via GPO
pubDate: 2017-11-10 16:26:30
tags:
- GPO
- NAC authentication
---
Use the following procedure to deploy sample wired authentication settings to NAP client computers for use with NAP and 802.1X enforcement.
<!-- more -->
1.	Open Group Policy Management on Domain Controller.
2.	Create a new Group Policy Object or choose an existing Group Policy Object.
3.	Edit the GPO
    ![01.png](/images/2017-11-10-How-to-Configure-Wired-Authentication-Settings-via-GPO/01.png)
4.	In Group Policy Management Editor, navigate to `Computer Configuration` -> `Policies` -> `Windows Settings` -> `Security Settings` -> `System Services` -> `Wired AutoConfig`
Then check Define this policy setting and choose Automatic.
Click Ok to save the configuration.
    ![02.png](/images/2017-11-10-How-to-Configure-Wired-Authentication-Settings-via-GPO/02.png)
5.	Navigate to Computer `Configuration` -> `Policies` -> `Windows Settings` -> `Security Settings` -> `Wired Network (IEEE 802.3) Policies`.
Right click on the right panel and click Create A New Wired Network Policy for Windows Vista and Later Release.
    ![03.png](/images/2017-11-10-How-to-Configure-Wired-Authentication-Settings-via-GPO/03.png)
6.	Give a name and description for this policy on General tab.
    ![04.png](/images/2017-11-10-How-to-Configure-Wired-Authentication-Settings-via-GPO/04.png)
7.	Set details in Security Page: 
    Select authentication method: Microsoft Smart Card or other certificate
    Select authentication Mode: User or Computer authentication
    Click Properties for more details
    ![05.png](/images/2017-11-10-How-to-Configure-Wired-Authentication-Settings-via-GPO/05.png)
    Select `User a certificate on this computer` and `Use simple certificate selection (Recommended)`
    Select `Verify the server’s identity by validating the certificate`. In the Trusted Root Certificate Authorities: Select the `SP Root Certification Authority` with latest expiry Date if multiple `SP Root Certificate Authority` certificates are found on the notebook. 
    Click OK
    ![06.png](/images/2017-11-10-How-to-Configure-Wired-Authentication-Settings-via-GPO/06.png)
    Click Ok
8.	You should see the settings in right panel.
    ![07.png](/images/2017-11-10-How-to-Configure-Wired-Authentication-Settings-via-GPO/07.png)
9.	Close the Group Policy Management Editor and back to Group Policy Management.
    You should see 802.1x authentication’s settings are listed in GPO details
    ![08.png](/images/2017-11-10-How-to-Configure-Wired-Authentication-Settings-via-GPO/08.png)
10.	Apply this policy to target machines.
11.	On client site, once the GPO is applied (you can run gpupdate /force in cmd.exe), user should get the settings.
    ![09.png](/images/2017-11-10-How-to-Configure-Wired-Authentication-Settings-via-GPO/09.png)

#Non domain environment
Alternatively, you can export the Interface configuration profile from one machine and import to other machines.
1.	Manually configure the 802.1x authentication settings on one test machine
2.	Export the NIC profile:
    {% codeblock lang:powershell %}
    netsh lan show profiles
    netsh lan export profile folder=PATH_TO_FOLDER interface="INTERFACE_NAME"
    {% endcodeblock %}
3.	Copy the XML file to target machine.
    Run the below commands to import the wired profile:
    {% codeblock lang:powershell %}
    netsh lan add profile filename="PATH_AND_FILENAME.xml" interface="INTERFACE_NAME"
    {% endcodeblock %}

Reference:
* [Configure 802.1X Wired Access Clients by using Group Policy Management](https://msdn.microsoft.com/en-us/library/cc731213.aspx)
* [Import and Export Wired Authentication Settings with Netsh](http://techgenix.com/importandexportwiredauthenticationsettingswithnetsh/)

 

