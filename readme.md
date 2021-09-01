# How To Create Plug-ins for EmEditor

### Contents:

1. [Getting Started](#getting-started)
2. [Start A New Project From Scratch](#start-a-new-project-from-scratch)
3. [What Can You Do With a Plug-in?](#what-can-you-do-with-a-plug-in)
4. [Potential Problems and How To Fix Them](#potential-problems-and-how-to-fix-them)

### Prerequisites for this tutorial:

* Some Visual C++ experience
* Microsoft Visual Studio
* EmEditor Professional

# Getting Started

The first part of this tutorial is to make sure we can compile a simple program and get it working in [EmEditor](https://www.emeditor.com/).

1. **Download the [hello world program](https://github.com/Emurasoft/HelloWorld) and open HelloWorld.vcxproj in Visual Studio.**
2. **Build the project in Debug x64 configuration.**.
3. **Open EmEditor.** Go to Tools | Plug-ins | Customize Plug-ins..., click "Add" button and add the DLL (should be in `x64\Debug\HelloWorld.dll`).
4. **Click on the funny looking light bulb icon in your plug-ins toolbar.** Yes, that is supposed to be a light bulb. If you don't see the plug-ins toolbar, you can enable it in View | Toolbars | Plug-ins Toolbar. The plug-in writes "Hello World!" on the editor.

![Plug-ins toolbar](/plugins_toolbar.png)

5. **Remove the plug-in** by opening the Customize Plug-ins window, selecting the HelloWorld plug-in, and clicking "Remove".
# Start A New Project From Scratch
You can use HelloWorld as the starting point for your program, or alternatively, follow these instructions to setup a workspace to develop a new plug-in.
1. **Create a new DLL project.** In Visual Studio, make a new project using the **Dynamic-Link Library** template for Visual C++.
2. **Add the [three template files](https://github.com/Emurasoft/template) to your new project.** The template repository contains two header files and one module-definition file that are required to link a plug-in to EmEditor. Treat these files as read-only. The template files are:
	* `etlframe.h` - helper functions for plug-in.h
	* `plugin.h` - handles message passing tasks
	* `exports.def` - exports functions from the plug-in so that they are callable from EmEditor
3. **Make a couple changes to the project properties:**
	1. **Change the linker settings to use exports.def as the module definition file.** In project properties, go to Linker | Input. At the top of the window, change configuration and platform to all, so that this change affects all builds. Change the value of Module Definition File to `exports.def`.
	2. **Go to C/C++ | Code Generation | Runtime Library and change the Runtime Library to "Multi-threaded Debug (/MTd)" for debug configuration, and "Multi-threaded (/MT)" for release configuration.**
	3. **Under Linker | Input, add `shlwapi.lib` as an additional dependency.**
	4. **Change the debugging command to EmEditor.exe.** This is optional but recommended, so that the EmEditor is opened when you start debugging. In Debugging, change the Command value to the path to your EmEditor binary, `EmEditor.exe`.
4. **Create a new header file and add the following code.**
	* `CETLFrame` encapsulates the plug-in. It is constructed only when EmEditor opens, and destroyed only when EmEditor closes.
	* The enum constants contain information about the plug-in, such as the name and icon image of this plug-in. There is more information about these variables in ["CETLFrame Member Variables"](http://www.emeditor.org/en/plugin_member_variables_index.html).
	* The functions [`OnCommand()` and `OnEvents()`](http://www.emeditor.org/en/plugin_exports_index.html) are the main entry points for your application.
	* You can learn more about these callback functions, plus some optional ones, in ["Messages to Plug-ins"](http://www.emeditor.org/en/plugin_plugin_message_index.html) and ["Functions to Export"](http://www.emeditor.org/en/plugin_exports_index.html).

```C++
#pragma once

#include "resource.h"
#define WIN32_LEAN_AND_MEAN
#include <windows.h>
#include <windowsx.h>
#define VERIFY(f) _ASSERTE(f)
#define ETL_FRAME_CLASS_NAME MyCFrame
#include "etlframe.h"

class MyCFrame : public CETLFrame<MyCFrame>
{
public:
	enum {
		_IDS_NAME = 0,
		_IDS_MENU = 0,
		_IDS_STATUS = 0,
		_IDS_VER = 0,
		_USE_LOC_DLL = 0,
		_IDB_BITMAP = 0,
		_IDB_16C_24 = 0,
		_IDB_TRUE_16_DEFAULT = 0,
		_IDB_TRUE_16_HOT = 0,
		_IDB_TRUE_16_BW = 0,
		_IDB_TRUE_24_DEFAULT = 0,
		_IDB_TRUE_24_HOT = 0,
		_IDB_TRUE_24_BW = 0,
		_MASK_TRUE_COLOR = 0,
		_ALLOW_OPEN_SAME_GROUP = 0,
		_ALLOW_MULTIPLE_INSTANCES = 0,
		_MAX_EE_VERSION = 0,
		_MIN_EE_VERSION = 0,
		_SUPPORT_EE_PRO = 0,
		_SUPPORT_EE_STD = 0
	};

	void OnCommand(HWND hwnd);

	void OnEvents(HWND hwnd, UINT nEvent, LPARAM lParam) {}

	BOOL QueryStatus(HWND hwnd, LPBOOL pbChecked) {
		*pbChecked = FALSE;
		return TRUE;
	}

	BOOL DisableAutoComplete(HWND hwnd) {
		return FALSE;
	}

	BOOL UseDroppedFiles(HWND hwnd) {
		return FALSE;
	}

	BOOL QueryUninstall(HWND hwnd) {
		return FALSE;
	}

	BOOL SetUninstall(HWND hwnd, LPTSTR command, LPTSTR param) {
		return UNINSTALL_SIMPLE_DELETE;
	}

	BOOL QueryProperties(HWND hwnd) {
		return FALSE;
	}

	BOOL SetProperties(HWND hwnd) {
		return FALSE;
	}

	BOOL PreTranslateMessage(HWND hwnd, MSG* pMsg) {
		return FALSE;
	}

	LRESULT UserMessage(HWND hwnd, WPARAM wParam, LPARAM lParam) {
		return NULL;
	}
};
```

5. **Clear the main .cpp file and add the following code.** In this code snippet, change `main.h` to the name of the header file that you have created in the last step.
	* `_ETL_IMPLEMENT` calls necessary initialization code, and must come after the definition for `CETLFrame`.

```C++
#include "main.h"
_ETL_IMPLEMENT

void MyCFrame::OnCommand(HWND hwnd) {
	// TODO
}
```

6. **Give the plug-in a name and an icon.** EmEditor retrieves this information through resource files.
	1. **Add a new resource file** to your project. It should have the extension ".rc".
	2. **Add a String Table to the resource file.** Double click on your resource file to open the Resource View. Now right click on the resource folder and click Add Resource and add a String Table. Change the Caption value to the plug-in name.
	3. **Import a 16 bit color, 16x16 pixel bitmap image to the resource file.** If EmEditor is unable to load the icon, try opening and saving it in Paint, as a .bmp with 256 colors.
	4. In the enum on the main .cpp file, **change the value of `_IDS_MENU`, `_IDS_STATUS`, and `_IDS_NAME` to the ID name of the name string, and `_IDB_BITMAP` to the bitmap ID.**
7. Additional info
	* To include `etlframe.h` in other files, you must define `EE_EXTERN_ONLY` before including `etlframe.h`. (Do that for all source files except for the main .cpp file.) Thus, the top of the new file should look like this:

	```C++
	#define WIN32_LEAN_AND_MEAN
	#include <windows.h>
	#include <windowsx.h>
	#define VERIFY(f) _ASSERTE(f)
	#define EE_EXTERN_ONLY
	#define ETL_FRAME_CLASS_NAME CMyFrame
	#include "etlframe.h"
	```
	
	* There is a helpful function called `GetFrame()` which returns a pointer to your current `CETLFrame`.

# What Can You Do With a Plug-in?

A plug-in is best suited for an application that will be used across many projects, and that requires a graphical interface to be shown. EmEditor also allows you to write macros using Javascript, and this may be an easier option for tasks that don't require C++ or a GUI.
This section will outline what you can do with the plug-in API. Refer to the [API Documentation](http://www.emeditor.org/en/plugin_index.html) for detailed explanations.
1. A plug-in is able to **receive events**. Most plug-ins are activated when its button is clicked in the Plug-ins toolbar. When this happens, [`OnCommand()`](http://www.emeditor.org/en/plugin_exports_index.html) is called. You can also capture various other events in EmEditor such as selection change, using [`OnEvents()`](http://www.emeditor.org/en/plugin_exports_index.html).
2. A plug-in can **retrieve or write text in the editor**. To get the text shown in the editor, use [`Editor_GetLineW()`](http://www.emeditor.org/en/plugin_macro_editor_getlinew.html) or [`Editor_GetSelTextW()`](http://www.emeditor.org/en/plugin_macro_editor_getseltextw.html). (There is currently no function to retrieve the entire document in one call.) You can write text using [`Editor_InsertW()`](http://www.emeditor.org/en/plugin_macro_editor_insertw.html), [`Editor_InsertFileW()`](http://www.emeditor.org/en/plugin_macro_editor_insertfilew.html) and other related functions. There is an option to run a macro as well, with [`Editor_RunMacro()`](http://www.emeditor.org/en/plugin_macro_editor_runmacro.html). Most commands will require a window handle to your plug-in, which is obtained through `CETLFrame::m_hWnd`.
3. You can **send commands that are available in the menu bar** using ['Editor_ExecCommand()'](http://www.emeditor.org/en/plugin_macro_editor_execcommand.html).
4. You can **search for a string** using [`Editor_FindW()`](http://www.emeditor.org/en/plugin_macro_editor_findw.html) and [`Editor_FindRegex()`](http://www.emeditor.org/en/plugin_macro_editor_findregex.html).
5. **Manipulate CSV documents** with [`Editor_GetCell()`](http://www.emeditor.org/en/plugin_macro_editor_getcell.html), [`Editor_GetColumn()`](http://www.emeditor.org/en/plugin_macro_editor_getcolumn.html), [`Editor_AutoFill()`](http://www.emeditor.org/en/plugin_macro_editor_autofill.html).
6. **Get information about the current workspace** with [`Editor_Info()`](http://www.emeditor.org/en/plugin_macro_editor_info.html), [`Editor_GetClip()`](http://www.emeditor.org/en/plugin_macro_editor_getclip.html), [`Editor_GetVersion()`](http://www.emeditor.org/en/plugin_macro_editor_getversion.html), etc.
7. To **display a graphical interface**, you first must get a handle to the DLL using `EEGetLocaleInstanceHandle()`. Dialogs are simple to make, but to be able to use the editor while interacting with the plug-in, a better option might be to make a sidebar or a toolbar, such as what Projects and HTMLBar employs. For debugging or for outputting simple data, you can write to the status bar of the editor using [`Editor_SetStatusW()`](http://www.emeditor.org/en/plugin_macro_editor_setstatusw.html).

# Potential Problems and How To Fix Them

Here are some problems that I came across while making HelloWorld and CharacterCount:
* **"... is not a member of 'CMyFrame'."** You did not implement this required function or variable.
* **"C:\...\NewPlugin.dll is not a valid Win32 application."** See step 3.4 of "Start A New Project From Scratch".
* **Linking issues related to the plug-in library:** "error C2027: use of undefined type 'CMyFrame'", "error LNK2019: unresolved external symbol ... _ETLCreateFrame", "error C2027: use of undefined type 'ETL_FRAME_CLASS_NAME'", "error LNK2005: "void __cdecl DeleteAllFrames(void)" ... already defined in NewPlugin.obj". Make sure all required header includes are added, in the correct order.
* **"'_USE_LOC_DLL': undeclared identifier."** You must either define the variable `_USE_LOC_DLL` or `#define EE_EXTERN_ONLY`. See step 7.
* **My new plug-in does not show on the plug-in list after adding it.** Make sure you are importing the correct 32 or 64 bit version of your plug-in.
* **My plug-in icon on the toolbar looks like the Internet Explorer icon (or some other icon).** The BMP image was not readable. Read step 6.3.

The [Emurasoft organization page](https://github.com/Emurasoft) contains several source codes for plug-ins that you may reference.