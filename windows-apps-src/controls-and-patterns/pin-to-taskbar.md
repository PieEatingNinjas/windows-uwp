---
author: mijacobs
Description: You can programmatically pin your app to the taskbar,  bnd you can check if it's currently pinned.
title: Pin your app to the taskbar
template: detail.hbs
ms.author: mijacobs
ms.date: 02/08/2017
ms.topic: article
ms.prod: windows
ms.technology: uwp
keywords: windows 10, uwp, taskbar, taskbar manager, pin to taskbar, primary tile
---
# Pin your app to the taskbar

You can programmatically pin your own app to the taskbar, just like you can [pin your app to the Start menu](tiles-and-notifications-primary-tile-apis.md). And you can check whether your app is currently pinned, and whether the taskbar allows pinning. 

![Taskbar](images/taskbar/taskbar.png)

> [!IMPORTANT]
> **PRERELEASE | Requires Fall Creators Update**: You must target [Insider SDK 16225](https://www.microsoft.com/en-us/software-download/windowsinsiderpreviewSDK) and be running [Insider build 16226](https://blogs.windows.com/windowsexperience/2017/06/21/announcing-windows-10-insider-preview-build-16226-pc/) or higher to use the taskbar APIs.

> **Important APIs**: TaskbarManager class 


## When should you ask the user to pin your app to the taskbar? 

The TaskbarManager class lets you ask the user to pin your app to the taskbar; the user must approve the request. You put a lot of effort into building a stellar app, and now you have the opportunity to ask the user to pin it to taskbar. But before we dive into the code, here are some things to keep in mind as you are designing your experience:

* **Do** craft a non-disruptive and easily dismissible UX in your app with a clear "Pin to taskbar" call to action.
* **Do** clearly explain the value of your app before asking the user to pin it.
* **Don't** ask a user to pin your app if the tile is already pinned or the device doesn’t support it. (This article explains how to determine whether pinning is supported.)
* **Don't** repeatedly ask the user to pin your app (they will probably get annoyed).
* **Don't** call the pin API without explicit user interaction or when your app is minimized/not open.


## 1. Check whether the required APIs exist

If your app supports older versions of Windows 10, you need to check whether the TaskbarManager class is available. You can use the  [ApiInformation.IsTypePresent method](https://docs.microsoft.com/en-us/uwp/api/windows.foundation.metadata.apiinformation#Windows_Foundation_Metadata_ApiInformation_IsTypePresent_System_String_) to perform this check. If the TaskbarManager class isn't available, avoid executing any calls to the APIs.

```csharp
if (ApiInformation.IsTypePresent("Windows.UI.Shell.TaskbarManager"))
{
    // Taskbar APIs exist!
}

else
{
    // Older version of Windows, no taskbar APIs
}
```


## 2. Check whether taskbar is present and allows pinning

UWP apps can run on a wide variety of devices; not all of them support the taskbar. Right now, only Desktop devices support the taskbar. 

Even if the taskbar is available, a group policy on the user's machine might disable taskbar pinning. So, before you attempt to pin your app, you need to check whether pinning to the taskbar is supported. The TaskbarManager.IsPinningAllowed property returns true if the taskbar is present and allows pinning. 

```csharp
// Check if taskbar allows pinning (Group Policy can disable it, or some device families don't have taskbar)
bool isPinningAllowed = TaskbarManager.GetDefault().IsPinningAllowed;
```

> [!NOTE]
> If you don't want to pin your app to the taskbar and just want to find out whether the taskbar is available, use the TaskbarManager.IsSupported property.


## 3. Check whether your app is currently pinned to the taskbar

Obviously, there's no point in asking the user to let you pin the app to the taskbar if it's already pinned there. You can use the TaskbarManager.IsCurrentAppPinnedAsync method to check whether the app is already pinned before asking the user.

```csharp
// Check if your app is currently pinned
bool isPinned = await TaskbarManager.GetDefault().IsCurrentAppPinnedAsync();

if (isPinned)
{
	// The app is already pinned--no point in asking to pin it again!
}
else 
{
	//The app is not pinned. 
}
```


##  4. Pin your app

If the taskbar is present and pinning is allowed and your app currently isn't pinned, you might want to show a tip, in the form of a flyout with a button, to let users know that they can pin your app.

If the user clicks your button to pin the app, you would then call the TaskbarManager.RequestPinCurrentAppAsync method. This method displays a dialog that asks the user to confirm that they want your app pinned to the taskbar.

> [!IMPORTANT]
> This must be called from a foreground UI thread, otherwise an exception will be thrown.

```csharp
// Request to be pinned to the taskbar
bool isPinned = await TaskbarManager.GetDefault().RequestPinCurrentAppAsync();
```

![Pin dialog](images/taskbar/pin-dialog.png)

This method returns a boolean value that indicates whether your app is now pinned to the taskbar. If your app was already pinned, the method immediately returns true without showing the dialog to the user. If the user clicks "no" on the dialog, or pinning your app to the taskbar isn't allowed, the method returns false. Otherwise, the user clicked yes and the app was pinned, and the API will return true.


## Resources

* [Full code sample on GitHub](https://github.com/WindowsNotifications/quickstart-pin-to-taskbar)
* [Pin an app to the Start menu](tiles-and-notifications-primary-tile-apis.md)
* [Tiles, badges, and notifications](tiles-badges-notifications.md)