---
layout: post
title: Windows Environment Variables Using %SystemRoot%
tags: [sysadmin, powershell, windows, path, environment variables]
---

### Overview of my issue

This past week I was working an issue where a specific agent in our stack software was failing some commands on a group of Windows 2012 R2 servers. It seemed like there was a "Google Gap", as I call it. I couldn't find any recent information really describing my problem, so I've been inspired to write this blog post on my issue and resolution.

All the servers experiencing the issue were test EC2 instances built from the same codebase for the same application. I figured, "find the problem in the code base and it will solve the issue for all servers". First, I opened a case with our vendor to get some assistance and was informed the agent was failing to execute the wmic command. [WMIC](https://technet.microsoft.com/en-us/library/cc181088.aspx) is a command line tool that provides access to a lot of Windows management functions.

### Quick primer on the PATH environment variable

When you boot for the first time from a clean install of Windows Server, you are going to see something like the following for your system PATH environment variables:

![](\img\Windows-Environment-Variables-using-SystemRoot\path-variables.jpg)

But when you go to edit your PATH, you will actually see something like this on a clean boot Windows 2012 R2 on EC2:
```
%SystemRoot%\system32;%SystemRoot%;%SystemRoot%\System32\Wbem;%SYSTEMROOT%\System32\WindowsPowerShell\v1.0\; C:\Program Files\Amazon\cfn-bootstrap\
```

The reason for the difference between what you see in the GUI and what you get when you edit the field is because %SystemRoot% is interpreted and expanded to c:\\Windows. You can discover what your system root is like with this command:

```
C:\>set systemroot
SystemRoot=C:\WINDOWS
```

By having the PATH variable set correctly, you would ordinarily be able to launch wmic, or any other executables within your variables without having to specify the full path to the executable. For example, here I launch wmic from root of C:, even though the file actually resides in C:\\Windows\\System32\\Wbem

```
C:\>wmic
wmic:root\cli>os
BootDevice               BuildNumber  BuildType            Caption                                    CodeSet
\Device\HarddiskVolume1  9600         Multiprocessor Free  Microsoft Windows Server 2012 R2 Standard  1252
```

### Problem identified

Now I noticed when running wmic on an affected instance, the system was not actually able to locate the executable.

```
C:\>wmic
'wmic' is not recognized as an internal or external command,
operable program or batch file.
```
This first thing I did is to check to make sure wmic was actually where it should be and of course it is. This indicated to me that there was a problem with the PATH. Within the application code base, I was easily able to locate some lines of code modifying the PATH. From a clean boot machine, I checked the registry value type, ran the same powershell code the application used, and checked the value type again.

```
$newPath = "%SystemRoot%\system32;%SystemRoot%;%SystemRoot%\System32\Wbem;%SYSTEMROOT%\System32\WindowsPowerShell\v1.0\;C:\Program Files\Amazon\cfn-bootstrap\;c:\myApp"
[Environment]::SetEnvironmentVariable( "Path", $newPath, [System.EnvironmentVariableTarget]::Machine )
```

This is when I found the problem. The registry value type was actually getting changed using their code from an expandable string to a regular string which was unable to expand %SystemRoot%. You can see the difference between the two value types here:

* REG_SZ: "A null-terminated string. This will be either a Unicode or an ANSI string, depending on whether you use the Unicode or ANSI functions."
* REG_EXPAND_SZ: "A null-terminated string that contains unexpanded references to environment variables (for example, "%PATH%"). It will be a Unicode or ANSI string depending on whether you use the Unicode or ANSI functions. To expand the environment variable references, use the ExpandEnvironmentStrings function."

Learn more about [Registry Value Types](https://msdn.microsoft.com/en-us/library/windows/desktop/ms724884%28v=vs.85%29.aspx?f=255&MSPPError=-2147217396) here.

### A better approach

Aside from using an antiquated method for updating the registry which was causing problems, the application team had hard code the path with what they expected it to be. Not only does this cause a problem if we put new paths in our base AMI that would be erased by this code change, but %SystemRoot% was being explicitly added to a non expandable registry type. [Borrowed from this blog](http://www.computerperformance.co.uk/powershell/powershell_env_path.htm), a pull request with the following changes fixed everything:

```
$Reg = "Registry::HKLM\System\CurrentControlSet\Control\Session Manager\Environment"
$OldPath = (Get-ItemProperty -Path "$Reg" -Name PATH).Path
$NewPath = $OldPath + ’;c:\myApp\’
Set-ItemProperty -Path "$Reg" -Name PATH –Value $NewPath
```
