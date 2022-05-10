## Empty project-template to jumpstart your OpenXR Quest Vulkan project

If you want a quick template to jump-start your OpenXR Quest Vulkan project read on..

I didn't find any instructions on how to setup a blank Vulkan-based OpenXR project for Oculus Quest. 
The most helpful thing I found was this article by [@gayanediriweera](https://twitter.com/gayanediriweera) where he shares how to run the hello_xr sample on Oculus Quest: https://gayanediriweera.github.io/code/2021/04/06/how-to-run-helloxr-on-oculus-quest.html. 
However that project was too tied with the OpenXR SDK-Source for my liking. I wanted
   1. A empty blank template for a Vulkan-based OpenXR project running on Oculus Quest 
   2.  No OpenXR source (apart from headers)
   3.  No non-essential CMakeLists.txt, 3rd party libraries
   4.  No cruft from non-Vulkan APIs, or non-Quest runtimes 

That is this project. The caveat is this is only for Vulkan and Windows x86-64 arch - which is the use-case I'm primarily concerned with.

### Why is this so complicated:

Essentially, the whole rigmarole is because the OpenXR Android Loader doesn't work with Oculus so we have to use the Oculus provided one. 
One day this setup will be as easy as Rift.

### Requirements:
   1. Download and install the Vulkan SDK: https://vulkan.lunarg.com/doc/view/1.1.121.2/windows/getting_started.html . Ensure VULKAN_SDK environment variable is set to the installation direction `C:\VulkanSDK\1.3.4` (whatever the version is) 
   2. Setup Android Studio per instructions here: https://developer.oculus.com/documentation/native/android/mobile-studio-basics/
   3. Download the Oculus OpenXR Mobile SDK here: https://developer.oculus.com/downloads/package/oculus-openxr-mobile-sdk/ 
   4. Download all the files in this project

### Getting Started started:

## Create Repo from template 	
   - From Github at  https://github.com/suvarshibhadra/OculusOpenXRTemplateProject click "Use this template" to create your own Github repo
   - Clone your github repo to your machine

## Prerequisite Steps
(From https://gayanediriweera.github.io/code/2021/04/06/how-to-run-helloxr-on-oculus-quest.html)

1. Follow the steps in Oculus Mobile Device Setup and enable developer mode on your device
2. Follow the steps in Android Studio Setup for Oculus Mobile Development to install Android Studio and set it up to build Oculus mobile applications
3. Download the Oculus OpenXR Mobile SDK. As of writing, the standard OpenXR Android loader does not appear to work for Oculus Quest, so we have to link against the 
custom OpenXR loader library provided by Oculus. This is found inside the Oculus OpenXR Mobile SDK, which appears to contain a small number of additional resources for OpenXR native development on 
Oculus Quest hardware
4. Download the Oculus OpenXR Mobile SDK. Extract the zip to a reasonable location, eg (C:\ovr_openxr_mobile_sdk_1.0.13). Create an environment variable called OCULUS_OPENXR_MOBILE_SDK and set it to the path to this directory. 
5. Clone or download OpenXR SDK Source from the OpenXR SDK Source GitHub repository to a reasonable location on your local machine
6. If you haven't already, download and install the Vulkan SDK: https://vulkan.lunarg.com/doc/view/1.1.121.2/windows/getting_started.html . Ensure VULKAN_SDK environment variable is set to the installation direction `C:\VulkanSDK\1.3.4` (whatever the version is).

## Setup Rift 
1. Point CMAKE to CMake.lists at root of project.
2. Generate Visual Studio x86-64 project at project-root/build. Open Visual Studio project.
3. Select hello_xr (or your project) as the "Startup Project" in Visual Studio
4. Add -g Vulkan as command arguments in Project properties

### Setup Quest 
	1. Ensure OpenXR does not build the Android loader and uses Oculus one instead
		  a) In OpenXR/src/CMakeLists.txt verify the BUILD_LOADER option is OFF for Android and 
		  b) Change C:/Users/suv/Documents/openxr_oculus_mobile_sdk_1014/OpenXR/Libs/Android/${ANDROID_ABI}/${CMAKE_BUILD_TYPE}/libopenxr_loader.so
		     to where you downloaded the  Oculus Quest OpenXR mobile SDK
	   ```
	   set_property(
	   TARGET
	   openxr_loader
	   PROPERTY
	   IMPORTED_LOCATION
	    C:/Users/suv/Documents/openxr_oculus_mobile_sdk_1014/OpenXR/Libs/Android/${ANDROID_ABI}/${CMAKE_BUILD_TYPE}/libopenxr_loader.so
	    )
	   ```
	   		
	2. Add Native Build Setup 
		1. Close all projects in Android Studio. 
		2. Click File > New Project. Name the project to be your project name. Save it in some arbitrary location.
		3. Select No Activity Template
		4. For Language: Java. Min SDK put in 24 and hit Finish
		5. Copy all the files at the root of the newly created project into the root of your cloned project - overriding files as necessary.
		6. In Android Studio, in the project explorer pane, right click on the app-module and select 'Add C++ to Module'
		7. Select "Link an existing CMakeLists.txt" and find the root CMakeLists.txt om the project7. In app/build.gradle verify this adds the following:	    

	    ```	    
		    externalNativeBuild {
			    cmake {
				path file('../CMakeLists.txt')
				version '3.10.2'
			    }
			}
	    ```
	   
	   8. In app/build.gradle under `default_config`  add
	   
	    ```
	    default_config {
	      ...
	      externalNativeBuild {
			cmake {
			    cppFlags ''
			}
			ndk {
			    abiFilters 'arm64-v8a'
			}
		    }
	      ...            
	    }
	    ```
	    
		9. From Project View in `app/src/main/AndroidManifest.xml` add
		
	    ```
<?xml version="1.0" encoding="utf-8"?>
<manifest xmlns:android="http://schemas.android.com/apk/res/android"
    package="com.khronos.hello_xr"
    android:versionCode="1"
    android:versionName="1.0" android:installLocation="auto" >
    <uses-feature android:glEsVersion="0x00030002" android:required="true"/>
    <uses-feature android:name="android.hardware.vr.headtracking" android:required="true" />
    <application android:allowBackup="false" android:label="Hello XR" android:hasCode="false">

        <activity
            android:name="android.app.NativeActivity"
            android:theme="@android:style/Theme.Black.NoTitleBar.Fullscreen"
            android:launchMode="singleTask"
            android:screenOrientation="landscape"
            android:excludeFromRecents="false"
            android:configChanges="screenSize|screenLayout|orientation|keyboardHidden|keyboard|navigation|uiMode">
            <!-- Tell NativeActivity the name of the .so -->
            <meta-data android:name="android.app.lib_name" android:value="hello_xr" />
            <!-- This filter lets the apk show up as a launchable icon. -->
            <intent-filter>
                <action android:name="android.intent.action.MAIN" />
                <category android:name="com.oculus.intent.category.VR" />
                <category android:name="android.intent.category.LAUNCHER" />
            </intent-filter>
        </activity>
    </application>
</manifest>
	    ```
	   10. Select Edit -> Find -> Replace in Files Search and Replace: **hello_xr** w/ your project name
	   11. Run `adb shell setprop debug.xr.graphicsPlugin Vulkan` from terminal
	   12 Run your app from Android Studio by `Shift+F10`. You should see the `hello_xr` sample app shared in the Khronos OpenXR SDK-Source repo.
	   13. Modify main.cpp in `hello_xr\src` to start to customize your Vulkan app

### License
The blank/template/starter app is the `hello_xr` sample from Khronos and is governed by their license here https://github.com/KhronosGroup/OpenXR-SDK-Source/blob/master/LICENSE. 
	
### References:
@gayanediriweera shared how to run the hello_xr sample from the  Khronos on Oculus Quest here: https://gayanediriweera.github.io/code/2021/04/06/how-to-run-helloxr-on-oculus-quest.html. This really helped me unpack what was going on under the hood of the gradle/CMake build process and helped me in putting this together!
