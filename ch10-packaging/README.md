# 10. Packaging Apps for the Desktop
## Jpackage Usage
The jpackage usage is as follows:
```bash
jpackage <options>
```
### Requirements
The images created by jpackage are not different from other applications developers create for native platforms. Therefore, the same tools that are used to generate native applications for a specific operating system are used by jpackage as well.

For Windows, in order to generate native packages, developers need to install:
* WiX Toolset, a free third-party tool that generates exe and msi installers

### WiX Setup
Download WiX Toolset from https://wixtoolset.org/releases/. For usage with jpackage, the current recommended version is 3.14.1. It can be downloaded from https://github.com/wixtoolset/wix3/releases/tag/wix3141rtm . Once downloaded, process with the installer, and when finished, add it to the path, running from the command line

```bash
setx /M PATH "%PATH%;C:\Program Files (x86)\WiX Toolset v3.14\bin"
```
Depending on your configuration, you might need administrator rights to do this. Alternatively, you can modify the path via the Settings: go to Settings ➤ System ➤ About ➤ Advanced System Settings ➤ Environment variables. Then edit System variables ➤ Edit Path and add
```
C:\Program Files (x86)\WiX Toolset v3.14\bin
```

Note The following output is very dependent on the version of WiX. The current version of jpackage works with WiX 3, so make sure to install a corresponding version. Keep in mind that you will most likely see different output than what is shown below.

### Samples
We will now present a few samples that show you how to use jpackage. The samples themselves are very simple JavaFX applications, and we will not discuss their functionality in this chapter.

Let’s start using jpackage from a terminal window without build tools or plugins. Since you use jpackage slightly differently depending on which platform you are running, we distinguish between using jpackage on Windows, macOS, and Linux.

#### Non-modular Application: Sample1
As a first sample, we explain how to package a Java application that itself is not a module. The application still uses the JavaFX modules, but there is no specific module-info.java in the application itself.

We describe the instructions on how to package this application into an installer for Windows, Mac, and Linux. The pattern we follow is similar for every platform:
1.	Define a number of environment variables.
2.	Compile the JavaFX application into Java bytecode.
3.	Run and test the application.
4.	Create a single jar file, containing the application.
5.	Create an installer using jpackage.

##### Instructions for Windows
These are the required steps to create an installer for a non-modular application, such as this one:
https://github.com/modernclientjava/mcj-samples/tree/master/ch10-packaging/Sample1

Clone the sample and, from a terminal, cd into the application’s root.

These first four steps are not different from regular Java compilation and running.

1. Export these environment variables:
    ```bash
    set JAVA_HOME="C:\Users\<user>\Downloads\jdk-17"
    set PATH_TO_FX="C:\Users\<user>\Downloads\javafx-sdk-17\lib"
    set PATH_TO_FX_MODS="C:\Users\<user>\Downloads\javafx-jmods-17"
    ```
    Note that if you have a different JDK added to the PATH environment variable, this will take precedence.
2.	Compile your application and copy the fxml and css resource files to path `out\org\modernclients`:  

    ```bash
    dir /s /b src\*.java > sources.txt & javac --module-path %PATH_TO_FX% --add-modules javafx.controls,javafx.fxml -d out @sources.txt & del sources.txt
 
    copy src\org\modernclients\scene.fxml out\org\modernclients\ & copy src\org\modernclients\styles.css out\org\modernclients\
    ``` 
3.	Run and test:  

    ```bash 
    java --module-path %PATH_TO_FX% --add-modules javafx.controls,javafx.fxml -cp out org.modernclients.Main
    ```
4.	Create a jar:   

    ```bash
    mkdir libs
    jar --create --file=libs\sample1.jar --main-class=org.modernclients.Main -C out .
    ```
5.	Create the installer.
    In this step, we use jpackage to create an installer. We showed the different options that can be provided to jpackage in Table 10-1. In the following command, we specify a number of options:  

    ```bash
    %JAVA_HOME%\bin\jpackage --type exe -d installer -i libs --main-jar sample1.jar -n Sample1 --module-path %PATH_TO_FX_MODS% --add-modules javafx.controls,javafx.fxml --main-class org.modernclients.Main
    ```

As a result, jpackage creates Sample1-1.0.exe (26 MB) that can be distributed, and it requires just a double-click to install the application (Figure 10-6).

unning the jpackage tool with `--verbose` shows the following output that helps identify how jpackage builds the installer, where it stores the default resources, and how you can customize these settings:
```bash
 %JAVA_HOME%\bin\jpackage --verbose --type exe -d installer -i libs --main-jar sample1.jar -n Sample1 --module-path %PATH_TO_FX_MODS% --add-modules javafx.controls,javafx.fxml --main-class org.modernclients.Main

[09:04:06.461] Running candle.exe
[09:04:06.527] Running light.exe
[09:04:06.617] Detected [candle.exe] version [3.14.1.8722].
[09:04:06.617] Detected [light.exe] version [3.14.1.8722].
[09:04:06.617] WiX 3.14.1.8722 detected. Enabling advanced cleanup action.
[09:04:08.918] Command [PID: -1]:
    jlink --output C:\Users\ROBERT~1\AppData\Local\Temp\jdk.jpackage14724585981837635947\images\win-msi.image\Sample1\runtime --module-path D:\\Data\\JAVA\\old_jdks\\javafx-jmods-21.0.6;D:\\Data\\JAVA\\old_jdks\\java-17-openjdk-17.0.14.0.7-1.win.jdk.x86_64\\jmods --add-modules javafx.controls,javafx.fxml --strip-native-commands --strip-debug --no-man-pages --no-header-files
[09:04:08.918] Output:

[09:04:08.919] Returned: 0

[09:04:08.921] Using default package resource JavaApp.ico [icon] (add Sample1.ico to the resource-dir to customize).
[09:04:08.927] Warning: Windows Defender may prevent jpackage from functioning. If there is an issue, it can be addressed by either disabling realtime monitoring, or adding an exclusion for the directory "C:\Users\ROBERT~1\AppData\Local\Temp\jdk.jpackage14724585981837635947".
[09:04:08.938] Using default package resource WinLauncher.template [Template for creating executable properties file] (add Sample1.properties to the resource-dir to customize).
[09:04:09.177] MSI ProductCode: 6ad6fbff-52ef-3f2f-959a-a12e4c9b1f68.
[09:04:09.177] MSI UpgradeCode: 4e3a7148-be2c-3a36-bc72-feb6033ea68f.
[09:04:09.197] Using default package resource main.wxs [Main WiX project file] (add main.wxs to the resource-dir to customize).
[09:04:09.198] Using default package resource overrides.wxi [Overrides WiX project file] (add overrides.wxi to the resource-dir to customize).
[09:04:09.199] Preparing MSI config: C:\Users\ROBERT~1\AppData\Local\Temp\jdk.jpackage14724585981837635947\images\win-exe.image\Sample1-1.0.msi.
[09:04:09.200] Generating MSI: C:\Users\ROBERT~1\AppData\Local\Temp\jdk.jpackage14724585981837635947\images\win-exe.image\Sample1-1.0.msi.
[09:04:09.205] Running candle.exe in C:\Users\ROBERT~1\AppData\Local\Temp\jdk.jpackage14724585981837635947\images\win-msi.image\Sample1
[09:04:09.341] Command [PID: 14572]:
    candle.exe -nologo C:\Users\ROBERT~1\AppData\Local\Temp\jdk.jpackage14724585981837635947\config\main.wxs -ext WixUtilExtension -arch x64 -out C:\Users\ROBERT~1\AppData\Local\Temp\jdk.jpackage14724585981837635947\wixobj\main.wixobj -dJpAppDescription=Sample1 -dJpProductCode=6ad6fbff-52ef-3f2f-959a-a12e4c9b1f68 -dJpAppName=Sample1 -dJpIsSystemWide=yes -dJpAllowDowngrades=yes -dJpIcon=C:\Users\ROBERT~1\AppData\Local\Temp\jdk.jpackage14724585981837635947\images\win-msi.image\Sample1\Sample1.exe -dJpAppSizeKb=83319 -dJpAppVersion=1.0 -dJpAllowUpgrades=yes -dJpProductUpgradeCode=4e3a7148-be2c-3a36-bc72-feb6033ea68f -dJpAppVendor=Unknown -dJpConfigDir=C:\Users\ROBERT~1\AppData\Local\Temp\jdk.jpackage14724585981837635947\config
[09:04:09.341] Output:
    main.wxs
[09:04:09.342] Returned: 0

[09:04:09.342] Running candle.exe in C:\Users\ROBERT~1\AppData\Local\Temp\jdk.jpackage14724585981837635947\images\win-msi.image\Sample1
[09:04:09.488] Command [PID: 14728]:
    candle.exe -nologo C:\Users\ROBERT~1\AppData\Local\Temp\jdk.jpackage14724585981837635947\config\bundle.wxf -ext WixUtilExtension -arch x64 -out C:\Users\ROBERT~1\AppData\Local\Temp\jdk.jpackage14724585981837635947\wixobj\bundle.wixobj
[09:04:09.488] Output:
    bundle.wxf
[09:04:09.488] Returned: 0

[09:04:09.488] Running candle.exe in C:\Users\ROBERT~1\AppData\Local\Temp\jdk.jpackage14724585981837635947\images\win-msi.image\Sample1
[09:04:09.583] Command [PID: 17680]:
    candle.exe -nologo C:\Users\ROBERT~1\AppData\Local\Temp\jdk.jpackage14724585981837635947\config\ui.wxf -ext WixUtilExtension -arch x64 -out C:\Users\ROBERT~1\AppData\Local\Temp\jdk.jpackage14724585981837635947\wixobj\ui.wixobj
[09:04:09.583] Output:
    ui.wxf
[09:04:09.583] Returned: 0

[09:04:09.584] Running light.exe in C:\Users\ROBERT~1\AppData\Local\Temp\jdk.jpackage14724585981837635947\images\win-msi.image\Sample1
[09:04:16.805] Command [PID: 5832]:
    light.exe -nologo -spdb -ext WixUtilExtension -out C:\Users\ROBERT~1\AppData\Local\Temp\jdk.jpackage14724585981837635947\images\win-exe.image\Sample1-1.0.msi -sice:ICE27 -loc C:\Users\ROBERT~1\AppData\Local\Temp\jdk.jpackage14724585981837635947\config\MsiInstallerStrings_en.wxl -cultures:en-us C:\Users\ROBERT~1\AppData\Local\Temp\jdk.jpackage14724585981837635947\wixobj\main.wixobj C:\Users\ROBERT~1\AppData\Local\Temp\jdk.jpackage14724585981837635947\wixobj\bundle.wixobj C:\Users\ROBERT~1\AppData\Local\Temp\jdk.jpackage14724585981837635947\wixobj\ui.wixobj
[09:04:16.805] Output:
    C:\Users\robert0714\AppData\Local\Temp\jdk.jpackage14724585981837635947\config\main.wxs(53) : warning LGHT1076 : ICE61: This product should remove only older versions of itself. No Maximum version was detected for the current product. (JP_DOWNGRADABLE_FOUND)
[09:04:16.805] Returned: 0

[09:04:16.806] Generating EXE for installer to: D:\Data\workspaces\intelliJ\apress-definitive-guide-modern-java-clients-javafx17-2022\ch10-packaging\Sample1\installer.
[09:04:16.808] Warning: Windows Defender may prevent jpackage from functioning. If there is an issue, it can be addressed by either disabling realtime monitoring, or adding an exclusion for the directory "C:\Users\ROBERT~1\AppData\Local\Temp\jdk.jpackage14724585981837635947".
[09:04:16.816] Using default package resource WinInstaller.template [Template for creating executable properties file] (add WinInstaller.properties to the resource-dir to customize).
[09:04:17.107] Installer (.exe) saved to: D:\Data\workspaces\intelliJ\apress-definitive-guide-modern-java-clients-javafx17-2022\ch10-packaging\Sample1\installer
[09:04:17.107] Succeeded in building EXE Installer Package package
```
##### Modifying the Installer
We can add more options to the jpackage command. For instance, we can add the application to the system menu, create a desktop shortcut, let the user choose the installation directory, and use a custom icon based on the Duke image from
https://hg.openjdk.java.net/duke/duke/raw-file/e71b60779736/3D/Duke%20Waving/openduke.png

Running the following jpackage command builds the customized installer that will create the application shown in Figure 10-7:
```bash
%JAVA_HOME%\bin\jpackage --type exe -d installer -i libs --main-jar sample1.jar -n Sample1 --module-path %PATH_TO_FX_MODS% --add-modules javafx.controls,javafx.fxml --main-class org.modernclients.Main --win-menu --win-shortcut --win-dir-chooser --icon assets\win\openduke.ico
../images/468104_2_En_10_Chapter/468104_2_En_10_Fig7_HTML.jpg
```


#### Modular Application: Sample2
Our second application is a modular application. The source code can be found at https://github.com/modernclientjava/mcj-samples/tree/master/ch10-packaging/Sample2.

It contains a module-info.java file, and the jpackage tool can process this to deal with the modular dependencies. The module-info file is very simple: it declares dependencies on the javafx.controls and javafx.fxml modules, and it exports the module org.modernclients. Also, module org.modernclients is opened to the javafx.fxml module.

The module-info.java file is shown here:

```java
module modernclients {
    requires javafx.controls;
    requires javafx.fxml;
    opens org.modernclients to javafx.fxml;
    exports org.modernclients;
}
```
#### Windows
These are the required steps to create an installer for a modular application on Windows:
1. Export these environment variables:
    ```bash 
    set JAVA_HOME="C:\Users\<user>\Downloads\jdk-17"
    set PATH_TO_FX="C:\Users\<user>\Downloads\javafx-sdk-17\lib"
    set PATH_TO_FX_MODS="C:\Users\<user>\Downloads\javafx-jmods-17"
    ```
2. Compile your application and copy the fxml and css resource files to path out\org\modernclients:
    ```bash 
    dir /s /b src\*.java > sources.txt & javac --module-path %PATH_TO_FX% --add-modules javafx.controls,javafx.fxml -d mods\modernclients @sources.txt & del sources.txt
    copy src\org\modernclients\scene.fxml mods\modernclients\org\modernclients\ & copy src\org\modernclients\styles.css mods\modernclients\org\modernclients\
    ```
3. Run and test:
    ```bash 
    java --module-path %PATH_TO_FX%;mods -m modernclients/org.modernclients.Main
    ```
4. Create the custom image with jlink:
    ```bash 
    %JAVA_HOME%\bin\jlink --module-path %PATH_TO_FX_MODS%;mods --add-modules modernclients --output image
    ```
5. Run and test the image:
    ```bash 
    image\bin\java -m modernclients/org.modernclients.Main
    ```
6. Create the installer:
    ```bash 
    %JAVA_HOME%\bin\jpackage --type exe -d installer -n Sample2 -m modernclients/org.modernclients.Main --runtime-image image
    ```
As a result, you will get Sample2-1.0.exe (32 MB) that can be distributed, and it requires just a double-click to install the application.

unning the jpackage tool with `--verbose` shows the following output that helps identify how jpackage builds the installer, where it stores the default resources, and how you can customize these settings:
```bash
 %JAVA_HOME%\bin\jpackage --verbose  --type exe -d installer -n Sample2 -m modernclients/org.modernclients.Main --runtime-image image

[09:19:53.302] Running candle.exe
[09:19:53.367] Running light.exe
[09:19:53.467] Detected [candle.exe] version [3.14.1.8722].
[09:19:53.467] Detected [light.exe] version [3.14.1.8722].
[09:19:53.468] WiX 3.14.1.8722 detected. Enabling advanced cleanup action.
[09:19:53.665] Using default package resource JavaApp.ico [icon] (add Sample2.ico to the resource-dir to customize).
[09:19:53.676] Warning: Windows Defender may prevent jpackage from functioning. If there is an issue, it can be addressed by either disabling realtime monitoring, or adding an exclusion for the directory "C:\Users\ROBERT~1\AppData\Local\Temp\jdk.jpackage11081459426552932038".
[09:19:53.699] Using default package resource WinLauncher.template [Template for creating executable properties file] (add Sample2.properties to the resource-dir to customize).
[09:19:54.010] MSI ProductCode: 2b137071-a878-3bb1-823f-3edf3a5dc556.
[09:19:54.010] MSI UpgradeCode: 5ed54d1f-12a1-3486-a50b-1d5c07f9a6e1.
[09:19:54.032] Using default package resource main.wxs [Main WiX project file] (add main.wxs to the resource-dir to customize).
[09:19:54.033] Using default package resource overrides.wxi [Overrides WiX project file] (add overrides.wxi to the resource-dir to customize).
[09:19:54.034] Preparing MSI config: C:\Users\ROBERT~1\AppData\Local\Temp\jdk.jpackage11081459426552932038\images\win-exe.image\Sample2-1.0.msi.
[09:19:54.035] Generating MSI: C:\Users\ROBERT~1\AppData\Local\Temp\jdk.jpackage11081459426552932038\images\win-exe.image\Sample2-1.0.msi.
[09:19:54.041] Running candle.exe in C:\Users\ROBERT~1\AppData\Local\Temp\jdk.jpackage11081459426552932038\images\win-msi.image\Sample2
[09:19:54.178] Command [PID: 22240]:
    candle.exe -nologo C:\Users\ROBERT~1\AppData\Local\Temp\jdk.jpackage11081459426552932038\config\main.wxs -ext WixUtilExtension -arch x64 -out C:\Users\ROBERT~1\AppData\Local\Temp\jdk.jpackage11081459426552932038\wixobj\main.wixobj -dJpAppDescription=Sample2 -dJpProductCode=2b137071-a878-3bb1-823f-3edf3a5dc556 -dJpAppName=Sample2 -dJpIsSystemWide=yes -dJpAllowDowngrades=yes -dJpIcon=C:\Users\ROBERT~1\AppData\Local\Temp\jdk.jpackage11081459426552932038\images\win-msi.image\Sample2\Sample2.exe -dJpAppSizeKb=98809 -dJpAppVersion=1.0 -dJpAllowUpgrades=yes -dJpProductUpgradeCode=5ed54d1f-12a1-3486-a50b-1d5c07f9a6e1 -dJpAppVendor=Unknown -dJpConfigDir=C:\Users\ROBERT~1\AppData\Local\Temp\jdk.jpackage11081459426552932038\config
[09:19:54.178] Output:
    main.wxs
[09:19:54.179] Returned: 0

[09:19:54.180] Running candle.exe in C:\Users\ROBERT~1\AppData\Local\Temp\jdk.jpackage11081459426552932038\images\win-msi.image\Sample2
[09:19:54.335] Command [PID: 8976]:
    candle.exe -nologo C:\Users\ROBERT~1\AppData\Local\Temp\jdk.jpackage11081459426552932038\config\bundle.wxf -ext WixUtilExtension -arch x64 -out C:\Users\ROBERT~1\AppData\Local\Temp\jdk.jpackage11081459426552932038\wixobj\bundle.wixobj
[09:19:54.335] Output:
    bundle.wxf
[09:19:54.335] Returned: 0

[09:19:54.335] Running candle.exe in C:\Users\ROBERT~1\AppData\Local\Temp\jdk.jpackage11081459426552932038\images\win-msi.image\Sample2
[09:19:54.440] Command [PID: 1156]:
    candle.exe -nologo C:\Users\ROBERT~1\AppData\Local\Temp\jdk.jpackage11081459426552932038\config\ui.wxf -ext WixUtilExtension -arch x64 -out C:\Users\ROBERT~1\AppData\Local\Temp\jdk.jpackage11081459426552932038\wixobj\ui.wixobj
[09:19:54.440] Output:
    ui.wxf
[09:19:54.440] Returned: 0

[09:19:54.441] Running light.exe in C:\Users\ROBERT~1\AppData\Local\Temp\jdk.jpackage11081459426552932038\images\win-msi.image\Sample2
[09:20:02.716] Command [PID: 13424]:
    light.exe -nologo -spdb -ext WixUtilExtension -out C:\Users\ROBERT~1\AppData\Local\Temp\jdk.jpackage11081459426552932038\images\win-exe.image\Sample2-1.0.msi -sice:ICE27 -loc C:\Users\ROBERT~1\AppData\Local\Temp\jdk.jpackage11081459426552932038\config\MsiInstallerStrings_en.wxl -cultures:en-us C:\Users\ROBERT~1\AppData\Local\Temp\jdk.jpackage11081459426552932038\wixobj\main.wixobj C:\Users\ROBERT~1\AppData\Local\Temp\jdk.jpackage11081459426552932038\wixobj\bundle.wixobj C:\Users\ROBERT~1\AppData\Local\Temp\jdk.jpackage11081459426552932038\wixobj\ui.wixobj
[09:20:02.716] Output:
    C:\Users\robert0714\AppData\Local\Temp\jdk.jpackage11081459426552932038\config\main.wxs(53) : warning LGHT1076 : ICE61: This product should remove only older versions of itself. No Maximum version was detected for the current product. (JP_DOWNGRADABLE_FOUND)
[09:20:02.716] Returned: 0

[09:20:02.716] Generating EXE for installer to: D:\Data\workspaces\intelliJ\apress-definitive-guide-modern-java-clients-javafx17-2022\ch10-packaging\Sample2\installer.
[09:20:02.721] Warning: Windows Defender may prevent jpackage from functioning. If there is an issue, it can be addressed by either disabling realtime monitoring, or adding an exclusion for the directory "C:\Users\ROBERT~1\AppData\Local\Temp\jdk.jpackage11081459426552932038".
[09:20:02.723] Using default package resource WinInstaller.template [Template for creating executable properties file] (add WinInstaller.properties to the resource-dir to customize).
[09:20:03.066] Installer (.exe) saved to: D:\Data\workspaces\intelliJ\apress-definitive-guide-modern-java-clients-javafx17-2022\ch10-packaging\Sample2\installer
[09:20:03.066] Succeeded in building EXE Installer Package package 
```

## Using GraalVM’s Native Image
The jpackage tool lets you build native applications for specific operating systems. The Java runtime is bundled with the application, and when the native application is executed, it will internally use the Java runtime to execute the bytecodes. Typically, the Java runtime contains a Just In Time (JIT) compiler that compiles Java bytecode to native code.

Another option for building a native application moves the compilation step to build time. With GraalVM’s Native Image, the Java code is compiled Ahead Of Time (AOT). This means the Java bytecode is compiled to native code before it is executed and before it is bundled into an application.

Although the GraalVM project (“Run Programs Faster Anywhere”) has been active for many years, it only recently became available as a product. GraalVM is still evolving, and parts of it are integrated with the OpenJDK project and vice versa. We recommend that you regularly keep an eye on the https://graalvm.org website and on the GitHub site for the open source code at https://github.com/oracle/graal.

While GraalVM provides the AOT compiler that translates Java bytecode into native code for a given platform, there are more actions needed in order to link the program code into an executable. A GraalVM maven plugin exists for Java applications on any platform, but this doesn’t deal with JavaFX applications. Fortunately, there are open source tools available that help developers achieve this. The GluonFX plugin (from gluonhq.com) lets developers create native images for Linux, macOS, and Windows based on existing Java and JavaFX code.

This same plugin also generates native images for mobile apps, which we discuss in the next chapter.

We will now show you how to build a native executable with the HelloFX sample application using the GluonFX plugin.
### Platform Requirements
In order to build a native image, you use JDK 17 or GraalVM JDK 17. We’ll briefly describe the requirements for both Maven and Gradle projects on macOS, Linux, and Windows.

You can download JDK 17 or GraalVM JDK 17 for each target system. For example, you can download AdoptOpenJDK from the following URL (choose the appropriate release for the target platform):
https://adoptopenjdk.net/releases.html

RedHat:
* https://access.redhat.com/products/openjdk/
* https://access.redhat.com/jbossnetwork/restricted/listSoftware.html?downloadType=distributions&product=core.service.openjdk&version=17.0.14

GraalVM: 
* You can download the Gluon GraalVM release from this URL (choose the appropriate release for the target platform):
https://github.com/gluonhq/graal/releases/
* You can download the github GraalVM-CE release from this URL: https://github.com/graalvm/graalvm-ce-builds/releases/tag/jdk-17.0.9
* You can download the Oracle GraalVM release from this URL: https://www.oracle.com/java/technologies/javase/graalvm-jdk17-archive-downloads.html

```bash
set GRAALVM_HOME="C:\Users\<user>\Downloads\graalvm-community-openjdk-17.0.9+9.1" 
set PATH=%GRAALVM_HOME%\bin
```
or Linux

```bash
export GRAALVM_HOME=/opt/graalvm-ce-java17
export PATH=$GRAALVM_HOME/bin:$PATH
```

To install `native-image`
```bash
gu install native-image
native-image --version
```

See https://docs.gluonhq.com/#_graalvm_3120

### Requirements for macOS
To use the plugin to develop and deploy native applications on macOS platforms, you need a Mac with macOS 10.13.2 or higher and Xcode 11 or higher, available from the Mac App Store. Once Xcode is downloaded and installed, open it to accept the license terms.

Once downloaded and installed, set JAVA_HOME to the JDK, for example:
```bash
export JAVA_HOME=/Users/<user>/Downloads/jdk-17/Contents/Home
```
### Requirements for Linux
After downloading a JDK for Linux, export the JAVA_HOME environment variable for the Linux platform JDK, for example:
```bash
export JAVA_HOME=/home/<user>/Downloads/jdk-17
```
### Requirements for Windows
After downloading a JDK for Windows, set the JAVA_HOME environment variable for the Windows platform JDK, for example:
```bash
set JAVA_HOME=C:\path\to\jdk-17
```
Add JAVA_HOME to the Environment Variables list (Advanced system settings).

In addition to the Java JDK, Microsoft Visual Studio 2022is also required. The community edition is sufficient, which you can download from
https://visualstudio.microsoft.com/downloads/

During the installation process, make sure to select at least the following individual components:
* Choose the English Language Pack.
* C++/CLI support for v143 build tools (latest version).
* MSVC v143 – VS 2022 C++ x64/x86 build tools (latest version).
* Windows Universal CRT SDK.
* Windows 10 SDK (10.0.20348.0 or later).

Note that all build commands must be executed in a Visual Studio 2022 command prompt called x64 Native Tools Command Prompt for VS 2022.

### The Code
The code for this example is on GitHub:
* https://github.com/gluonhq/gluon-samples/tree/master/HelloFX/src/main/java/hellofx
* https://github.com/gluonhq/gluon-samples/tree/master/HelloFXML/src/main/java/hellofx

### JavaFX References 
* https://openjfx.io/openjfx-docs/#maven
### Build the Project
The first step is to build and run the project as a regular Java project (on a regular JVM that you use for your local development, e.g., HotSpot).
With Gradle: 
```bash
./gradlew clean build run (Mac OSX or Linux)
gradlew clean build run (Windows)
```
With Maven:  
```bash
mvn clean javafx:run

```

We will now compile, package, and run the native desktop application.

#### Compile
Run with Gradle:
```bash
./gradlew nativeCompile (Mac OS or Linux)
gradlew nativeCompile (Windows)
```
Or with Maven:
```bash
mvn gluonfx:compile
```
You’ll need to wait until the task finishes successfully (depending on your machine, it may take 5 minutes or more). You will see the feedback provided during the process, such as in Listing 10-6.
```bash
[INFO] Scanning for projects...
[INFO]
[INFO] --------------------< com.gluonhq.samples:hellofx >---------------------
[INFO] Building HelloFX 1.0.0-SNAPSHOT
[INFO]   from pom.xml
[INFO] --------------------------------[ jar ]---------------------------------
[INFO]
[INFO] >>> gluonfx-maven-plugin:1.0.23:compile (default-cli) > process-classes @ hellofx >>>
[INFO]
[INFO] --- maven-resources-plugin:2.6:resources (default-resources) @ hellofx ---
[INFO] Using 'UTF-8' encoding to copy filtered resources.
[INFO] Copying 2 resources
[INFO]
[INFO] --- maven-compiler-plugin:3.8.1:compile (default-compile) @ hellofx ---
[INFO] Nothing to compile - all classes are up to date
[INFO]
[INFO] <<< gluonfx-maven-plugin:1.0.23:compile (default-cli) < process-classes @ hellofx <<<
[INFO]
[INFO]
[INFO] --- gluonfx-maven-plugin:1.0.23:compile (default-cli) @ hellofx ---
1月 30, 2025 10:51:24 上午 com.gluonhq.substrate.util.Logger logInfo
資訊: Substrate is tested with the Gluon's GraalVM build which you can find at https://github.com/gluonhq/graal/releases.
While you can still use other GraalVM builds, there is no guarantee that these will work properly with Substrate
[週四 1月 30 10:51:24 TST 2025][資訊] ==================== COMPILE TASK ====================
             _______  ___      __   __  _______  __    _
            |       ||   |    |  | |  ||       ||  |  | |
            |    ___||   |    |  | |  ||   _   ||   |_| |
            |   | __ |   |    |  |_|  ||  | |  ||       |
            |   ||  ||   |___ |       ||  |_|  ||  _    |
            |   |_| ||       ||       ||       || | |   |
            |_______||_______||_______||_______||_|  |__|

    Access to the latest docs, tips and tricks and more info on
    how to get support? Register your usage of Gluon Substrate now at

    https://gluonhq.com/activate



[週四 1月 30 10:51:25 TST 2025][資訊] We will now compile your code for x86_64-microsoft-windows. This may take some time.
[週四 1月 30 10:51:26 TST 2025][資訊] [SUB] Warning: Ignoring server-mode native-image argument --no-server.
[週四 1月 30 10:51:28 TST 2025][資訊] [SUB] ========================================================================================================================
[週四 1月 30 10:51:28 TST 2025][資訊] [SUB] GraalVM Native Image: Generating 'org.modernclients.main' (shared library)...
[週四 1月 30 10:51:28 TST 2025][資訊] [SUB] ========================================================================================================================
[週四 1月 30 10:51:28 TST 2025][資訊] [SUB] For detailed information and explanations on the build output, visit:
[週四 1月 30 10:51:28 TST 2025][資訊] [SUB] https://github.com/oracle/graal/blob/master/docs/reference-manual/native-image/BuildOutput.md
[週四 1月 30 10:51:28 TST 2025][資訊] [SUB] ------------------------------------------------------------------------------------------------------------------------
[週四 1月 30 10:51:32 TST 2025][資訊] [SUB] [1/8] Initializing...                                                                                    (5.8s @ 0.20GB)
[週四 1月 30 10:51:32 TST 2025][資訊] [SUB]  Java version: 17.0.9+9, vendor version: GraalVM CE 17.0.9+9.1
[週四 1月 30 10:51:32 TST 2025][資訊] [SUB]  Graal compiler: optimization level: 2, target machine: x86-64-v3
[週四 1月 30 10:51:32 TST 2025][資訊] [SUB]  C compiler: cl.exe (microsoft, x64, 19.42.34436)
[週四 1月 30 10:51:32 TST 2025][資訊] [SUB]  Garbage collector: Serial GC (max heap size: 80% of RAM)
[週四 1月 30 10:51:32 TST 2025][資訊] [SUB]  1 user-specific feature(s)
[週四 1月 30 10:51:32 TST 2025][資訊] [SUB]  - org.graalvm.home.HomeFinderFeature: Finds GraalVM paths and its version number
[週四 1月 30 10:51:55 TST 2025][資訊] [SUB] [2/8] Performing analysis...  [******]                                                                  (22.7s @ 1.03GB)
[週四 1月 30 10:51:55 TST 2025][資訊] [SUB]   11,649 (89.18%) of 13,062 types reachable
[週四 1月 30 10:51:55 TST 2025][資訊] [SUB]   20,552 (66.41%) of 30,949 fields reachable
[週四 1月 30 10:51:55 TST 2025][資訊] [SUB]   56,687 (61.92%) of 91,556 methods reachable
[週四 1月 30 10:51:55 TST 2025][資訊] [SUB]    3,262 types,   179 fields, and 2,624 methods registered for reflection
[週四 1月 30 10:51:55 TST 2025][資訊] [SUB]      135 types,   152 fields, and   207 methods registered for JNI access
[週四 1月 30 10:51:55 TST 2025][資訊] [SUB]        4 native libraries: crypt32, ncrypt, version, winhttp
[週四 1月 30 10:51:58 TST 2025][資訊] [SUB] [3/8] Building universe...                                                                               (2.6s @ 1.33GB)
[週四 1月 30 10:52:00 TST 2025][資訊] [SUB] [4/8] Parsing methods...      [**]                                                                       (2.1s @ 1.80GB)
[週四 1月 30 10:52:01 TST 2025][資訊] [SUB] [5/8] Inlining methods...     [****]                                                                     (1.3s @ 1.22GB)
[週四 1月 30 10:52:18 TST 2025][資訊] [SUB] [6/8] Compiling methods...    [****]                                                                    (16.3s @ 1.96GB)
[週四 1月 30 10:52:21 TST 2025][資訊] [SUB] [7/8] Layouting methods...    [**]                                                                       (3.2s @ 2.72GB)
[週四 1月 30 10:52:24 TST 2025][資訊] [SUB]
[週四 1月 30 10:52:24 TST 2025][資訊] [SUB] ------------------------------------------------------------------------------------------------------------------------
[週四 1月 30 10:52:24 TST 2025][資訊] [SUB]                         2.8s (4.8% of total time) in 105 GCs | Peak RSS: 3.92GB | CPU load: 8.10
[週四 1月 30 10:52:24 TST 2025][資訊] [SUB] ------------------------------------------------------------------------------------------------------------------------
[週四 1月 30 10:52:24 TST 2025][資訊] [SUB] Produced artifacts:
[週四 1月 30 10:52:24 TST 2025][資訊] [SUB]  D:\Data\workspaces\intelliJ\apress-definitive-guide-modern-java-clients-javafx17-2022\ch10-packaging\Sample3\target\gluonfx\x86_64-windows\gvm\HelloFX\graal_isolate.h (c_header)
[週四 1月 30 10:52:24 TST 2025][資訊] [SUB]  D:\Data\workspaces\intelliJ\apress-definitive-guide-modern-java-clients-javafx17-2022\ch10-packaging\Sample3\target\gluonfx\x86_64-windows\gvm\HelloFX\graal_isolate_dynamic.h (c_header)
[週四 1月 30 10:52:24 TST 2025][資訊] [SUB]  D:\Data\workspaces\intelliJ\apress-definitive-guide-modern-java-clients-javafx17-2022\ch10-packaging\Sample3\target\gluonfx\x86_64-windows\gvm\HelloFX\org.modernclients.main.h (c_header)
[週四 1月 30 10:52:24 TST 2025][資訊] [SUB]  D:\Data\workspaces\intelliJ\apress-definitive-guide-modern-java-clients-javafx17-2022\ch10-packaging\Sample3\target\gluonfx\x86_64-windows\gvm\HelloFX\org.modernclients.main_dynamic.h (c_header)
[週四 1月 30 10:52:24 TST 2025][資訊] [SUB] ========================================================================================================================
[週四 1月 30 10:52:24 TST 2025][資訊] [SUB] Finished generating 'org.modernclients.main' in 58.0s.
[INFO] ------------------------------------------------------------------------
[INFO] BUILD SUCCESS
[INFO] ------------------------------------------------------------------------
[INFO] Total time:  01:02 min
[INFO] Finished at: 2025-01-30T10:52:25+08:00
[INFO] ------------------------------------------------------------------------
```
Listing 10-6Output during the native compilation phase
As a result, you will see `org.modernclients.main.o` (65.0 MB) or `org.modernclients.main.obj` under `target/gluonfx/{target-architecture}/gvm/tmp/SVM-***/`.

If that is not the case, check for any possible failures in the log files under `target/gluonfx/{target-architecture}/gvm/log`.

#### Link
Now that the Java code for the application is compiled to native code, we can package the generated code with the required libraries and resources using the nativeLink task .

Run with Gradle:
```bash
./gradlew nativeLink (Mac OSX or Linux)
gradlew nativeLink (Windows)
```
Or with Maven:
```bash
mvn gluonfx:link

[INFO] Scanning for projects...
[INFO]
[INFO] --------------------< com.gluonhq.samples:hellofx >---------------------
[INFO] Building HelloFX 1.0.0-SNAPSHOT
[INFO]   from pom.xml
[INFO] --------------------------------[ jar ]---------------------------------
[INFO]
[INFO] --- gluonfx-maven-plugin:1.0.23:link (default-cli) @ hellofx ---
1月 30, 2025 10:58:26 上午 com.gluonhq.substrate.util.Logger logInfo
資訊: Substrate is tested with the Gluon's GraalVM build which you can find at https://github.com/gluonhq/graal/releases.
While you can still use other GraalVM builds, there is no guarantee that these will work properly with Substrate
[週四 1月 30 10:58:26 TST 2025][資訊] ==================== LINK TASK ====================
[週四 1月 30 10:58:26 TST 2025][資訊] Default icon.ico image generated in D:\Data\workspaces\intelliJ\apress-definitive-guide-modern-java-clients-javafx17-2022\ch10-packaging\Sample3\target\gluonfx\x86_64-windows\gensrc\windows\assets.
Consider copying it to D:\Data\workspaces\intelliJ\apress-definitive-guide-modern-java-clients-javafx17-2022\ch10-packaging\Sample3\src\windows before performing any modification
[週四 1月 30 10:58:26 TST 2025][資訊] [SUB] Microsoft (R) Incremental Linker Version 14.42.34436.0
[週四 1月 30 10:58:26 TST 2025][資訊] [SUB] Copyright (C) Microsoft Corporation.  All rights reserved.
[週四 1月 30 10:58:26 TST 2025][資訊] [SUB]
[週四 1月 30 10:58:26 TST 2025][資訊] [SUB]    正在建立程式庫 D:\Data\workspaces\intelliJ\apress-definitive-guide-modern-java-clients-javafx17-2022\ch10-packaging\Sample3\target\gluonfx\x86_64-windows\HelloFX.lib 和物件 D:\Data\workspaces\intelliJ\apress-definitive-guide-modern-java-clients-javafx17-2022\ch10-packaging\Sample3\target\gluonfx\x86_64-windows\HelloFX.exp
             _______  ___      __   __  _______  __    _
            |       ||   |    |  | |  ||       ||  |  | |
            |    ___||   |    |  | |  ||   _   ||   |_| |
            |   | __ |   |    |  |_|  ||  | |  ||       |
            |   ||  ||   |___ |       ||  |_|  ||  _    |
            |   |_| ||       ||       ||       || | |   |
            |_______||_______||_______||_______||_|  |__|

    Access to the latest docs, tips and tricks and more info on
    how to get support? Register your usage of Gluon Substrate now at

    https://gluonhq.com/activate



[INFO] ------------------------------------------------------------------------
[INFO] BUILD SUCCESS
[INFO] ------------------------------------------------------------------------
[INFO] Total time:  2.377 s
[INFO] Finished at: 2025-01-30T10:58:28+08:00
[INFO] ------------------------------------------------------------------------

```
The link step produces the executable in the target subdirectory `target/gluonfx/{target-architecture}/HelloFX` (65.4 MB) or in `target\gluonfx\x86_64-windows\hellofx.exe` for Windows. Figure 10-13 shows the executable in a macOS file system.

It can be executed directly or with `mvn gluonfx:nativerun`.
#### Run
Finally, you can run it with Gradle:
```bash
./gradlew nativeRun (Mac OS or Linux)
gradlew nativeRun (Windows)
```
Or with Maven:
```bash
mvn gluonfx:run
```

Note that you can distribute this native application to any machine with the matching architecture (macOS, Linux, or Windows) and run it directly as any other regular application.

## Conclusion
Packaging application code with all the required dependencies, Java runtime, and resources is becoming increasingly popular on desktop, mobile, and embedded devices.

The historical disadvantages, including larger size and longer download times, are less important due to improvements in bandwidth and storage.

The advantages of packaging an application into a self-contained bundle mean less hassle for end users, who can use an installation approach that is familiar and similar to other applications.

JavaFX applications are regular Java applications. Thus, you can use the packaging tools that exist for Java, such as jpackage, jlink, and Graal Native Image, with JavaFX applications as well.

Since these tools are evolving rapidly, we recommend you keep an eye on the samples in the GitHub repository https://github.com/gluonhq/gluon-samples, as they will be updated to the latest version once available.