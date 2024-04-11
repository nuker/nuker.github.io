---
layout: post
title: Reading Windows 10 Active Hours
date: 2024-01-03
---

---
# **A curious case**

> You may have seen this dialog box before

![Adjust Active Hours](/images/AdjustActive.png){:class="img-responsive"}

> The most important thing to me here is at the start it says,
> **We noticed you regularly use your device between time1 (AM/PM) - time2 (AM/PM).**
> then it talks about how it wont restart for updates during these active hours.

> When I saw that I thought to myself, how can I also read that information without having to look at that dialog box everytime.
> It could be useful right? While building a profile on your clients as a Red Teamer. So with that motivation I went to google my first query was

```Windows update restart```

> Which lead me to this link on MSDN: [Manage device restarts after updates](https://learn.microsoft.com/en-us/windows/deployment/update/waas-restart)

# **To the registry**

> Towards the bottom where it says **Registry keys used to manage restart** the registry key ```HKLM\Software\Policies\Microsoft\Windows\WindowsUpdate``` can be used to read the active hours

![MSDN](/images/ActiveHours.png){:class="img-responsive"}

> I thought I was on the right track so, I then went to google again and searched 

```Windows registry active hours```

> Which lead me to: [3 Ways to Change Windows 10 Active Hours](https://www.majorgeeks.com/content/page/3_ways_to_change_windows_10_active_hours.html)

> This blog specified the sub key ```HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\WindowsUpdate\UX\Settings```

![Registry](/images/ActiveHoursReg.png){:class="img-responsive"}

> Looking at this we can see the values **SmartActiveHoursStart** and **SmartActiveHoursEnd** that are read and displayed in the **Adjust active hours to reduce disruptions** dialog box.

# **Putting it to work**

> With the previous registry subkey we can read the values for ourselves.
> We can do this using the Windows API with C/C++.


> **It should also be noted that you should be running with elevated privileges when querying HKLM**

```c
// ActiveHours.c

#include <stdio.h>
#include <stdlib.h>
#include <windows.h>

int main() {

    // Open the key

    HKEY key = NULL;
    LONG Open = RegOpenKeyEx(HKEY_LOCAL_MACHINE, "SOFTWARE\\Microsoft\\WindowsUpdate\\UX\\Settings", 0, KEY_READ, &key);

    if (Open != ERROR_SUCCESS) {

        printf("Could not open key.\n");
        return -1;
    }

    // Values we want

    DWORD szStart;
    DWORD tStart;

    DWORD szEnd;
    DWORD tEnd;

    const char* Value1 = "SmartActiveHoursStart";
    const char* Value2 = "SmartActiveHoursEnd";

    // Get data type and size

    LONG check1 = RegQueryValueExA(key, Value1, NULL, &tStart, NULL, &szStart);
    LONG check2 = RegQueryValueExA(key, Value2, NULL, &tEnd, NULL, &szEnd);

    if (check1 != ERROR_SUCCESS || check2 != ERROR_SUCCESS) {

        printf("Count not get data types or sizes.\n");

        RegCloseKey(key);

        return -1;

    }

    // Do the querys

    BYTE* Start = (BYTE*)malloc(szStart);
    BYTE* End = (BYTE*)malloc(szEnd);

    LONG check3 = RegQueryValueExA(key, Value1, NULL, &tStart, Start, &szStart);
    LONG check4 = RegQueryValueExA(key, Value2, NULL, &tEnd, End, &szEnd);

    if (check3 != ERROR_SUCCESS || check4 != ERROR_SUCCESS) {

        printf("Could not query values.\n");

        RegCloseKey(key);

        free(Start);
        free(End);

        return -1;
    }

    // Print values

    printf("SmartActiveHoursStart: %lu\n", *(DWORD*)Start);
    printf("SmartActiveHoursEnd: %lu\n", *(DWORD*)End);

    // Close key

    RegCloseKey(key);

    // No longer using this data we can free it

    free(Start);
    free(End);

    return 0;
}
```

> To compile this I used **mingw**

```x86_64-w64-mingw32-gcc ActiveHours.c -o ActiveHours.exe```

# **Final**

> This code was executed on an virtual machine (Windows 10 Pro)

![Final](/images/Final.png){:class="img-responsive"}

> Ultimately I think this could be used to make sure your implants only do certain things based on
> the times where your clients are most active on their computer. I hope this information was useful to someone.

