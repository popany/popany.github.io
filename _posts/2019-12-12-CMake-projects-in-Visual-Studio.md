---
layout: post
title: CMake projects in Visual Studio
---

## [CMake projects in Visual Studio](https://docs.microsoft.com/en-us/cpp/build/cmake-projects-in-visual-studio?view=vs-2019)

The C++ CMake tools for Windows component uses the **Open Folder feature** to consume CMake project files (such as CMakeLists.txt) directly for the purposes of IntelliSense and browsing.

### IDE Integration

When you choose **File** > **Open** > **Folder** to open a folder containing a `CMakeLists.txt` file, the following things happen:

* Visual Studio adds CMake items to the Project menu, with commands for viewing and editing CMake scripts.

* Solution Explorer displays the folder structure and files.

* Visual Studio runs `cmake.exe` and generates the CMake cache file (CMakeCache.txt) for the **default configuration**, which is `x64 Debug`. The CMake command line is displayed in the Output Window, along with additional output from CMake.

* In the background, Visual Studio starts to index the source files to enable IntelliSense, browsing information, refactoring, and so on. As you work, Visual Studio monitors changes in the editor and also on disk to keep its index in sync with the sources.

You can open folders containing **any number of CMake projects**. Visual Studio detects and configures all the "root" `CMakeLists.txt` files in your workspace. CMake operations (configure, build, debug) as well as C++ IntelliSense and browsing are available to all CMake projects in your workspace.

Visual Studio uses a file called `CMakeSettings.json` which enables you to define and store multiple build configurations and conveniently switch between them in the IDE. A configuration is a **Visual Studio construct** that encapsulates settings that are specific to a given build type. The settings are used to configure the default **command-line options** that Visual Studio passes to `cmake.exe`. You can also specify additional CMake options here, and define any additional variables you like. All options are written to the CMake cache either as internal or external variables.

One setting, `intelliSenseMode` is not passed to CMake, but is used only by Visual Studio.

Use the `CMakeLists.txt` file in each project folder just as you would in any CMake project to specify source files, find libraries, set compiler and linker options, and specify other build system related information.

If you need to pass arguments to an executable at debug time, you can use another file called `launch.vs.json`. In some scenarios, Visual Studio will automatically generate these files; you can edit them manually. You can also create the file yourself.

### Building CMake projects

To build a CMake project, you have these choices:

1. In the General toolbar, find the `Configurations` dropdown. It probably shows "`x64-Debug`" by default. Select the desired configuration and press `F5`, or click the `Run` (green triangle) button on the toolbar. The project automatically builds first, just like a Visual Studio solution.

2. Right click on the `CMakeLists.txt` and select **Build** from the context menu. If you have multiple targets in your folder structure, you can choose to build all or only one specific target.

3. From the main menu, select **Build** > **Build All** (`F7` or `Ctrl+Shift+B`). Make sure that a CMake target is already selected in the **Startup Item** dropdown in the **General** toolbar.

In a folder with multiple build targets, you can choose the **Build** item on the **CMake** menu or the `CMakeLists.txt` context menu to specify which CMake target to build. Pressing `Ctrl+Shift+B` in a CMake project builds the current active document.

### CMake configure step

When significant changes are made to the `CMakeSettings.json` or to `CMakeLists.txt` files, Visual Studio automatically reruns the CMake configure step. If the configure step finishes without errors, the information that is collected is available in C++ IntelliSense and language services and also in build and debug operations.

## [Create and configure a Linux CMake project](https://docs.microsoft.com/en-us/cpp/linux/cmake-linux-project?view=vs-2019)

To create a new Linux CMake project in Visual Studio 2019:

1. Select **File** > **New Project** in Visual Studio, or press **Ctrl + Shift + N**.

2. Set the **Language** to **C++** and search for "CMake". Then choose **Next**. Enter a **Name** and **Location**, and choose **Create**.

...

## [Customize CMake build settings](https://docs.microsoft.com/en-us/cpp/build/customize-cmake-settings?view=vs-2019)

...

## Useful materials

[CMake predefined build configurations](https://docs.microsoft.com/en-us/cpp/build/cmake-predefined-configuration-reference?view=vs-2019)

[My First C/C++ App Built with CMake on Windows](https://www.codepool.biz/cc-barcode-app-cmake-windows.html)

[CMake: Build C++ Project for Windows, Linux and macOS](https://www.codepool.biz/cmake-cc-windows-linux-macos.html)

[A CMake tutorial for Visual C++ developers](https://www.codeproject.com/Articles/1181455/A-CMake-tutorial-for-Visual-Cplusplus-developers)
