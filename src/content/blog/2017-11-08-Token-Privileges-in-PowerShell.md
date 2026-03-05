---
title: Token Privileges in PowerShell
pubDate: 2017-11-08 17:06:51
tags:
- PowerShell
- TokenPrivileges
---
One common task in my daily work is to create a PowerShell script to modify the ACL(Access Control List) of Folders or Registrys on Windows. When I run some commands to take the ownership of the Folders or Registrys, sometimes I get permission denied error, even I have ran it as Administrator. What happens?
<!-- more --> 
# Access Tokens and Privileges
According the article [here](https://msdn.microsoft.com/en-us/library/windows/desktop/ms721532.aspx#_security_access_token_gly): An access token contains the security information for a logon session. The system creates an access token when a user logs on, and every process executed on behalf of the user has a copy of the token. The token identifies the user, the user's groups, and the user's **privileges**.
A **privileges** is the right of an account, such as a user or group account, to perform various system-related operations on the local computer, such as shutting down the system, loading device drivers, or changing the system time.
So when you get the permission error, it doesn’t mean that you can’t do it – just that you need to enable the privilege before doing it.

# Solution
Windows provided a API, [AdjustTokenPrivileges](https://msdn.microsoft.com/en-us/library/windows/desktop/aa375202.aspx), to adjust the privileges in the specified access token.
Here is an example, I would like to delete some registry key under [HKLM\SOFTWARE\Microsoft\Windows\CurrentVersion\Component Based Servicing\Packages\]. If I ran the command Remove-Item to delete it, probably it will be denied by the OS.
So I need call the API to manipulate the privileges before calling Remove-Item.
Here is sample script:

```powershell
$AdjustTokenPrivileges=@"
using System;
using System.Runtime.InteropServices;

  public class TokenManipulator {
    [DllImport("kernel32.dll", ExactSpelling = true)]
      internal static extern IntPtr GetCurrentProcess();

    [DllImport("advapi32.dll", ExactSpelling = true, SetLastError = true)]
      internal static extern bool AdjustTokenPrivileges(IntPtr htok, bool disall, ref TokPriv1Luid newst, int len, IntPtr prev, IntPtr relen);
    [DllImport("advapi32.dll", ExactSpelling = true, SetLastError = true)]
      internal static extern bool OpenProcessToken(IntPtr h, int acc, ref IntPtr phtok);
    [DllImport("advapi32.dll", SetLastError = true)]
      internal static extern bool LookupPrivilegeValue(string host, string name, ref long pluid);

    [StructLayout(LayoutKind.Sequential, Pack = 1)]
    internal struct TokPriv1Luid {
      public int Count;
      public long Luid;
      public int Attr;
    }

    internal const int SE_PRIVILEGE_DISABLED = 0x00000000;
    internal const int SE_PRIVILEGE_ENABLED = 0x00000002;
    internal const int TOKEN_QUERY = 0x00000008;
    internal const int TOKEN_ADJUST_PRIVILEGES = 0x00000020;

    public static bool AddPrivilege(string privilege) {
     bool retVal;
      TokPriv1Luid tp;
      IntPtr hproc = GetCurrentProcess();
      IntPtr htok = IntPtr.Zero;
      retVal = OpenProcessToken(hproc, TOKEN_ADJUST_PRIVILEGES | TOKEN_QUERY, ref htok);
      tp.Count = 1;
      tp.Luid = 0;
      tp.Attr = SE_PRIVILEGE_ENABLED;
      retVal = LookupPrivilegeValue(null, privilege, ref tp.Luid);
      retVal = AdjustTokenPrivileges(htok, false, ref tp, 0, IntPtr.Zero, IntPtr.Zero);
      return retVal;
    }

    public static bool RemovePrivilege(string privilege) {
      bool retVal;
      TokPriv1Luid tp;
      IntPtr hproc = GetCurrentProcess();
      IntPtr htok = IntPtr.Zero;
      retVal = OpenProcessToken(hproc, TOKEN_ADJUST_PRIVILEGES | TOKEN_QUERY, ref htok);
      tp.Count = 1;
      tp.Luid = 0;
      tp.Attr = SE_PRIVILEGE_DISABLED;
      retVal = LookupPrivilegeValue(null, privilege, ref tp.Luid);
      retVal = AdjustTokenPrivileges(htok, false, ref tp, 0, IntPtr.Zero, IntPtr.Zero);
      return retVal;
    }
  }
"@

Write-Verbose "Giving current process token ownership rights"
    Add-Type $AdjustTokenPrivileges -PassThru > $null
    [void][TokenManipulator]::AddPrivilege("SeTakeOwnershipPrivilege") 
    [void][TokenManipulator]::AddPrivilege("SeRestorePrivilege") 

$AddACL = New-Object System.Security.AccessControl.RegistryAccessRule ("BUILTIN\Administrators","FullControl","ObjectInherit,ContainerInherit","None","Allow")
$owner = [System.Security.Principal.NTAccount]"Administrators"

$keyCR = [Microsoft.Win32.Registry]::LocalMachine.OpenSubKey("SOFTWARE\Microsoft\Windows\CurrentVersion\Component Based Servicing\Packages\", [Microsoft.Win32.RegistryKeyPermissionCheck]::ReadWriteSubTree,[System.Security.AccessControl.RegistryRights]::takeownership)
# Get a blank ACL since you don't have access and need ownership
$aclCR = $keyCR.GetAccessControl([System.Security.AccessControl.AccessControlSections]::None)
$aclCR.SetOwner($owner)
$keyCR.SetAccessControl($aclCR)

# Get the acl and modify it
$aclCR = $keyCR.GetAccessControl()
$aclCR.SetAccessRule($AddACL)
$keyCR.SetAccessControl($aclCR)
$keyCR.Close()

Remove-Item -path 'HKLM:\SOFTWARE\Microsoft\Windows\CurrentVersion\Component Based Servicing\Packages\Microsoft-Windows-PowerShell-V2-ServerCore-WOW64-Package~31bf3856ad364e35~amd64~ja-JP~6.3.9600.16384' -Recurse

Write-Host "..Done." 
```

Enjoy it.
References:
* [Adjusting Token Privileges in PowerShell](http://www.leeholmes.com/blog/2010/09/24/adjusting-token-privileges-in-powershell/)
* [How do I take ownership of a registry key via PowerShell?](https://stackoverflow.com/questions/12044432/how-do-i-take-ownership-of-a-registry-key-via-powershell)

