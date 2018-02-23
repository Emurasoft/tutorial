# How To Create Plugins for EmEditor
<sup>By Makoto Emura</sup><br />
<sup>[CC-BY](#License)</sup>
### Contents:
1. [Getting Started](#Getting Started)
2. [Start A New Project From Scratch](#Start A New Project From Scratch)
3. [What Can You Do With a Plugin?](#What Can You Do With a Plugin?)
4. [Common Problems and How To Fix Them](#Common Problems and How To Fix Them)
### Prerequisites for this tutorial:
* Some experience in Visual C++
* Microsoft Visual Studio
* EmEditor Professional
# Getting Started
The first part of this tutorial is to make sure we can compile a simple program and get it working in EmEditor.
1. **Clone the [hello world program](https://github.com/Emurasoft/HelloWorld) and open HelloWorld.vcxproj in Visual Studio.**
2. **Build the project.** Use the default configuration for now - debug on x64 or x86.
3. **Open EmEditor.** Go to Tools > Plug-ins > Customize Plug-ins... and click the button that says "Add" and add the DLL that was created.
4. **Click on the funny looking light bulb icon in your plug-ins toolbar.** If you don't see the plug-ins toolbar, you can enable it in View > Toolbars > Plug-ins Toolbar. Yes, that is supposed to resemble a light bulb. The plugin writes "Hello World! " on the editor.
5. **Remove the plugin** by opening the Customize Plug-ins window, selecting the HelloWorld plugin, and clicking "Remove."

# Start A New Project From Scratch
You can use HelloWorld as the starting point for your program, or alternatively, follow these instructions to setup a workspace to develop a new plugin.
1. **Create a new DLL project.** In Visual Studio, make a new project using the "Dynamic-Link Library" template for Visual C++.
2. **Add the [three template files](https://github.com/Emurasoft/template) to your new project.** The template repository contains files that are required to link a plugin to EmEditor. Treat these files as read-only. The template files are:
	* etlframe.h - helper functions for plugin.h
	* plugin.h - handles message passing tasks
	* exports.def - exports functions from the plugin so that they are callable from EmEditor
3. **Make a couple changes to the project properties:**
	1. **Change the linker settings to use exports.def as the module definition file.** In project properties, go to Linker > Input. Change the value of Module Definition File to exports.def. Make sure this property is changed for all configurations and all platforms
	2. **Change the Runtime Library to "Multi-threaded Debug (/MTd)" for debug configuration, and "Multi-threaded (/MT)" for release configuration.** This is found under C/C++ > Code Generation > Runtime Library.
	3. **Change the debugging command path to EmEditor.exe.** This is optional but recommended, so that the debugger opens EmEditor automatically. The setting is found in Debugging. Change the Command value to your installation path which ends with EmEditor.exe, without quotation marks.
4. **Add the following code into stdafx.h.** These include directives must be in this order.
```C++
#include <windowsx.h>
#include <crtdbg.h>
#include <tchar.h>
#pragma comment(lib, "shlwapi.lib")
```
6. **Add the following code into the main .cpp file.** This is the bare minimum of code in order to get a plugin working in EmEditor. 
	* The enum constants tell EmEditor about several properties, such as the name of this plugin.
	* The functions `OnCommand` and `OnEvents` are the main entry points.
	* `_ETL_IMPLEMENT` allows etlframe.h to call necessary construct and destroy tasks.
```C++
#include "stdafx.h"
#define ETL_FRAME_CLASS_NAME MyCFrame
#include "etlframe.h"
#include "resource.h"

class MyCFrame : public CETLFrame<MyCFrame>
{
public:
	enum {
		_USE_LOC_DLL = 0,
		_IDS_MENU = 0,
		_IDS_STATUS = 0,
		_IDS_NAME = 0,
		_IDS_VER = 0,
		_IDB_BITMAP = 0,
		_IDB_16C_24 = 0,
		_IDB_TRUE_16_DEFAULT = 0,
		_IDB_TRUE_16_HOT = 0,
		_IDB_TRUE_16_BW = 0,
		_IDB_TRUE_24_DEFAULT = 0,
		_IDB_TRUE_24_HOT = 0,
		_IDB_TRUE_24_BW = 0,
		_MASK_TRUE_COLOR = 0,
		_MASK_256_COLOR = 0,
		_ALLOW_OPEN_SAME_GROUP = 0,
		_ALLOW_MULTIPLE_INSTANCES = 0,
		_MAX_EE_VERSION = 0,
		_MIN_EE_VERSION = 0,
		_SUPPORT_EE_PRO = 0,
		_SUPPORT_EE_STD = 0
	};

	void OnCommand(HWND hwnd)
	{
		
	}

	void OnEvents(HWND hwnd, UINT nEvent, LPARAM lParam)
	{

	}

	BOOL QueryStatus(HWND hwnd, LPBOOL pbChecked)
	{*pbChecked = FALSE; return TRUE;}

	BOOL DisableAutoComplete(HWND hwnd)
	{return FALSE;}

	BOOL UseDroppedFiles(HWND hwnd)
	{return FALSE;}

	BOOL QueryUninstall(HWND hwnd)
	{return FALSE;}

	BOOL SetUninstall(HWND hwnd, LPTSTR command, LPTSTR param)
	{return UNINSTALL_SIMPLE_DELETE;}

	BOOL QueryProperties(HWND hwnd)
	{return FALSE;}

	BOOL SetProperties(HWND hwnd)
	{return FALSE;}

	BOOL PreTranslateMessage(HWND hwnd, MSG* pMsg)
	{return FALSE;}

	LRESULT UserMessage(HWND hwnd, WPARAM wParam, LPARAM lParam)
	{return NULL;}
};
_ETL_IMPLEMENT
```
5. **Give the plugin a name and an icon.** EmEditor retrieves this information through resource files.
	 1. **Add a new resource file** to your project. It should have the extension ".rc".
	 2. **Add a String Table to the resource file.** Double click on your resource file to open the Resource View. Now right click on the resource folder and click Add Resource and add a String Table. Change the Caption value to the plugin name.
	 3. **Import a 16 bit color, 16x16 pixel bitmap image to the resource file.** If EmEditor is unable to load the icon, try opening and saving it in paint, as a .bmp with 256 colors.
	 4. In the enum on the main .cpp file, **change the value of `_IDS_MENU`, `_IDS_STATUS`, and `_IDS_NAME` to the ID name of the name string, and `_IDB_BITMAP` to the bitmap ID.** The IDs are listed on resource.h.

# What Can You Do With a Plugin?
To be added soon!

# Common Problems and How To Fix Them

### License
This text, *How To Create Plugins for EmEditor*, is licensed under a [Creative Commons Attribution 4.0 International License](https://creativecommons.org/licenses/by/4.0/).