---
title: Call Native Win32 API in PowerShell
pubDate: 2017-11-28 10:31:22
tags:
- PowerShell
- Win32 API
---
Windows built-in DLLs exposes a lot of APIs for many low-level functionalities. It is quite useful to call native Windows API in PowerShell to accomplish specific tasks which are not being wrapped in any tools to commands. For example, if you want to set a system parameter of Windows, you will find that there is no commands or cmdlet to do that. But [SystemParametersInfo](https://msdn.microsoft.com/en-us/library/windows/desktop/ms724947.aspx) function in user32.dll could help you. But writing a native Win32 application to just call this API for a specific task is too heavy. In PowerShell, everything become easy.
<!-- more -->
# Methodology
Matt published an excellent [blog](https://blogs.technet.microsoft.com/heyscriptingguy/2013/06/25/use-powershell-to-interact-with-the-windows-api-part-1/) on three ways of interacting with Windows API:
*   **Use the Add-Type cmdlet to compile C# code. This is the officially documented method.**
*   Get a reference to a private type in the .NET Framework that calls the method.
*   Use reflection to dynamically define a method that calls the Windows API function.
In this blog, I will choose first one.

# Background
My task is to set a Windows system parameters. Although you can play tricks with registry key if you can _hack_ it, in some scenario it is hard to manipulate the key because the registry could be a combination of binary code.
After some research, we found a native Windows API, SystemParametersInfo, for this task:
```c
BOOL WINAPI SystemParametersInfo(
  _In_    UINT  uiAction,
  _In_    UINT  uiParam,
  _Inout_ PVOID pvParam,
  _In_    UINT  fWinIni
);
```

# Add-Type
[Add-Type](https://docs.microsoft.com/en-us/powershell/module/microsoft.powershell.utility/add-type?view=powershell-5.1) is a cmdlet allow us to extend PowerShell session with .NET Framework type (a class).
There is an example about how to call native Windows APIs in the document:
```PowerShell
PS C:\> $Signature = @"
[DllImport("user32.dll")]public static extern bool ShowWindowAsync(IntPtr hWnd, int nCmdShow);
"@
$ShowWindowAsync = Add-Type -MemberDefinition $Signature -Name "Win32ShowWindowAsync" -Namespace Win32Functions -PassThru 

# Minimize the Windows PowerShell console

$ShowWindowAsync::ShowWindowAsync((Get-Process -Id $pid).MainWindowHandle, 2)

# Restore it

$ShowWindowAsync::ShowWindowAsync((Get-Process -Id $Pid).MainWindowHandle, 4)
```

# Solution
First, use Add-Type to introduce the specific function into PowerShell session.
Second, call the API in your code. You can pass the parameters either in native code block or PowerShell code block.
```PowerShell
function Update-UserPreferencesMask {
    $Signature = @"
[DllImport("user32.dll", EntryPoint = "SystemParametersInfo")]
public static extern bool SystemParametersInfo(uint uiAction, uint uiParam, uint pvParam, uint fWinIni);
 
const int SPI_SETTHREADLOCALINPUTSETTINGS = 0x104F; 
const int SPIF_UPDATEINIFILE = 0x01; 
const int SPIF_SENDCHANGE = 0x02;
 
public static void UpdateUserPreferencesMask() {
    SystemParametersInfo(SPI_SETTHREADLOCALINPUTSETTINGS, 0, 1, SPIF_UPDATEINIFILE | SPIF_SENDCHANGE);
}
"@
    Add-Type -MemberDefinition $Signature -Name UserPreferencesMaskSPI -Namespace User32
    [User32.UserPreferencesMaskSPI]::UpdateUserPreferencesMask()
}

Update-UserPreferencesMask
```