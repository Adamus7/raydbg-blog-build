---
title: Understanding the Servicing Model for Windows
pubDate: 2017-06-22 22:28:07
tags:
- Windows
- Update
---
Recently, one of IE KB bring a print issue to IE 11 on different versions of Windows. Before getting the fix, the only solution is to uninstall the problematic KB. But many people are confused by one question: which KB should I uninstall? To answer it, we need to figure out the patch releasing model of Windows.
<!-- more --> 
# Before Windows 10
From Oct 2016, a new patch releasing model, *Rollup Model*, was introduced to the family of Windows 7SP1/2008R2, Windows 8.1 and Windows 2012/R2 by Microsoft. Later, starting with February 2017, Microsoft decided to exclude the Cumulative Security Update for Internet Explorer from Security Only updates. Meanwhile, there will be a standalone patch for IE will be released every month. So now, normally, you should see 4 different types of patch in one month:
* `Security Monthly Quality Update` (aka the `Monthly Rollup`)
    IE CU + Security and Non-Security fixes in this month + all fixes in previous months
* `Security Only Quality Update` (aka the `Security Only updates`)
    Security fix in this month                                                          
* `Preview of Monthly Quality Rollup` (aka the `Preview Rollup`)
    Non-Security fixes in this month +  all fixes in previous months                    
* `Cumulative Security Update for Internet Explorer` (aka the `CU for IE`)
    Fixes for IE                                                                        
    
Here is an diagram which can illustrate the relationship between different updates:
![Four different patches](/images/2017-06-22-Understanding-the-Servicing-Model-for-Windows/image1.png)

Therefore, let's go back to the IE printing issue, what you need to do is to uninstall the Monthly Rollup KB if you have installed or the Cumulative Security Update for Internet Explorer if you have installed.

For more details, you can check the KB articles here:
* Win 7 SP1 & 2008 R2: [Windows 7 SP1 and Windows Server 2008 R2 SP1 update history](https://support.microsoft.com/en-us/help/4009469)
* Win 8.1 & 2012 R2: [Windows 8.1 and Windows Server 2012 R2 update history](https://support.microsoft.com/en-us/help/4009470)

# Windows 10
On Windows 10, the story is simple. There is only one package, which contains all fixes, for different builds of Windows 10. You can check the update history here:
* Win 10: [Windows 10 update history](https://support.microsoft.com/en-us/help/4018124/windows-10-update-history)