---
layout: post
title: Windows - Auto login
categories:
  - windows
---

# Problem

In order to save costs in the Azure cloud, shutting down VMs overnight is one way to maximise on that. 
But what if you have a vm that needs to serve something like automated tests (Selenium Grid).

**nugget**
In this particular case, we need to login and set up the GRID - hub and node(s).
The solution for that would be to put what you need to run upon startup in the startup folder.
The easiest way to find the startup folder, is to type ***shell:startup*** in the Expolorer window and it will take you right there.

# Solution

Ok, on with the solution.
The best way to do this, is to jump into REGEDIT, and navigate through to **Computer\HKEY_CURRENT_USER\SOFTWARE\Microsoft\Windows NT\CurrentVersion\Winlogon**

![RegEdit WinLogon](images/RegEditWinLogon.png)

As you can see from the screenshot, I don't have anything that looks like it defines auto logon, and that is fine, we can set it up now.
We are going to add 3 key-value pairs to achieve the desired outcome.

In the right-hand pane, right-click and select **New** and then **String Value**. Enter the name, and then double-click to enter the value.

| Name  | Value | Comment  |
| --- | :---: | --- |
| **DefaultUserName**  | Username  | username that you want to use for auto logon  |
| **DefaultPassword**  | Password  | password that you want to use for auto logon |
| **AutoAdminLogon**  | 1  | 0 meaning not to auto logon after a restart |

NB: If any of these values already exist, just double-click to update their values. You may want to check if they are already set,
with whoever has created the auto logon, in case that causes problems further down the line.

That is all that you have to do, and restarting will then automatically login with those credentials.
