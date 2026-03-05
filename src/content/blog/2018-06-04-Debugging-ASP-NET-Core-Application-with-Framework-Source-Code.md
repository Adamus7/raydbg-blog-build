---
title: Debugging ASP.NET Core Application with Framework Source Code
pubDate: 2018-06-04 16:37:07
tags:
- ASP.NET Core
- Debugging
- .NET Core
---
Since .NET Core and ASP.NET Core is open sourced, developers are able to step into the source code of the framework for different purposes. 
As you may know, the source code of the dotnet runtime ([CoreCLR](https://github.com/dotnet/coreclr)), foundational libraries ([CoreFX](https://github.com/dotnet/corefx)) and [ASP.NET Core](https://github.com/aspnet/Home) are hosted on GitHub. With all the source code available, is that possible to have a different debugging experience with the framework source code?
<!-- more -->
# PDB and SourceLink
You may familiar with PDB (Program Database) file which contains debugging information about a program (EXE) or a module (DLL). A PDB file is created from source files during compilation and it stores a list of symbols of a module. With the help of the PDB file, the debugger is able to locate symbols or execution state of the source code. .NET Core introduces a new symbol file (PDB) format - portable PDBs. Unlike traditional PDBs which are Windows-only, portable PDBs can be created and read on all platforms.
PDB is not good enough. [Taggart Software](https://github.com/ctaggart/SourceLink) introduced a new feature, [SourceLink](https://github.com/dotnet/designs/blob/master/accepted/diagnostics/source-link.md), into dotnet world. Later, SourceLink becomes a .NET Foundation project in Nov 2017. Basically, SourceLink is a developer productivity feature that allows unique information about an assembly's original source code to be embedded in its PDB file during compilation.
Staring from Visual Studio 2017 version 15.3, VS supports reading SourceLink information from PDB file while debugging. That makes it much easier for developers to step into the source code of a referenced DLL if SourceLink is enabled in that package. ASP.NET Core 2.0 is of that libraries that enables SourceLink and provides links to its source code which is hosted on GitHub.

# Debugging into ASP.NET Core
Well, I haven't tried to fix any issue of the framework by debugging its source code. But I think it will be very helpful to understand how does the framework work and learn the internals if I can step into the source code.

## Prerequisite
To do so, in Visual Studio 2017 Version 15.3+, you just need a few clicks:
2.  Uncheck the `Tools` -> `Options` -> `Debugging` -> `Just My Code` checkbox.
3.  Insure that the `Tools` -> `Options` -> `Debugging` -> `Symbol Settings` -> `Microsoft Symbol Servers` checkbox is checked.
4.  Insure that the `Tools` -> `Options` -> `Debugging` -> `Enable Source Link support` checkbox is checked.

Let us test this step by step with a sample ASP.NET Core 2.0 Web application.

## Enabling *Just My Code*
I have created a ASP.NET Core 2.0 Web API app, and set a breakpoint into my app. Before enabling `Just My Code`, when the breakpoint is hit, only my codes are shown in CallStack window.
![500](/images/2018-06-04-Debugging-ASP-NET-Core-Application-with-Framework-Source-Code/enable-just-my-code-1.png)
After enabling `Just My Code`, Visual Studio 2017 takes some time to download the symbols from `Microsoft Symbol Servers` and load them for the ASP.NET Core Framework. When it is done, you should able to see the call stack of `[External Code]` part.
![800](/images/2018-06-04-Debugging-ASP-NET-Core-Application-with-Framework-Source-Code/enable-just-my-code-2.png)
![600](/images/2018-06-04-Debugging-ASP-NET-Core-Application-with-Framework-Source-Code/enable-just-my-code-3.png)
## Enable Source Link support
Click on the `Debugging` -> `General` options and make sure `Enable Source Link support` is checked.
*Note*: SourceLink V1 requires `Enable source server support`; SourceLink V2 requires `Enable Source Link support`. See: [SourceLink](https://github.com/ctaggart/SourceLink).
![500](/images/2018-06-04-Debugging-ASP-NET-Core-Application-with-Framework-Source-Code/enable-source-link-1.png)
Start the debugging again and stop the program at one of your breakpoint. Double click on a frame of the framework functions in CallStack window, there will be a Source Link popup and let you decide to download the source code from GitHub repository or not.
![500](/images/2018-06-04-Debugging-ASP-NET-Core-Application-with-Framework-Source-Code/enable-source-link-2.png)
Download the source and step into the source code. We have the source file and it will be displayed at the appropriate location from the frame you clicked. More importantly, you can set your own breakpoints or watch variables anywhere inside the source file just like a local project.
![600](/images/2018-06-04-Debugging-ASP-NET-Core-Application-with-Framework-Source-Code/enable-source-link-3.png)
![600](/images/2018-06-04-Debugging-ASP-NET-Core-Application-with-Framework-Source-Code/enable-source-link-4.png)
To be honest, this is pretty cool!

# Enable Source Link for Your Project
ASP.NET Core is a good example, but how could developers bring this feature into their own modules or NuGet package?
Dotnet Core team have added support for generating SourceLink information in symbols, binaries, and NuGet packages in the .NET Core 2.1 RC SDK. The goal for the project is to enable anyone building NuGet libraries to provide source debugging for their users with almost no effort.
Simply, you can enable SourceLink in your own project hosted on GitHub by following this example:
``` XML
<Project Sdk="Microsoft.NET.Sdk">
 <PropertyGroup>
    <TargetFramework>netcoreapp2.1</TargetFramework>
 
    <!-- Optional: Declare that the Repository URL can be published to NuSpec -->
    <PublishRepositoryUrl>true</PublishRepositoryUrl>
 
    <!-- Optional: Embed source files that are not tracked by the source control manager to the PDB -->
    <EmbedUntrackedSources>true</EmbedUntrackedSources>

    <!-- Optional: Include PDB in the built .nupkg -->
    <AllowedOutputExtensionsInPackageBuildOutputFolder>$(AllowedOutputExtensionsInPackageBuildOutputFolder);.pdb</AllowedOutputExtensionsInPackageBuildOutputFolder>
  </PropertyGroup>
  <ItemGroup>
    <PackageReference Include="Microsoft.SourceLink.GitHub" Version="1.0.0-beta-62925-02" PrivateAssets="All"/>
  </ItemGroup>
</Project>
```
For more details, please check this [document](https://github.com/dotnet/sourcelink/).

Reference:
* [Debugging Through the .NET Core Framework V2.0.3+ (Windows)](https://blogs.msdn.microsoft.com/vancem/2017/12/20/update-debugging-through-the-net-core-framework-v2-0-3-windows/)
* [Debugging into ASP.NET Core 2.0 source code](https://laurentkempe.com/2017/09/26/Debugging-into-ASP.NET-Core-2.0-source-code/)
* [Debugging ASP.NET Core 2.0 Source Code](https://www.stevejgordon.co.uk/debugging-asp-net-core-2-source)
* [SourceLink](https://github.com/dotnet/sourcelink)