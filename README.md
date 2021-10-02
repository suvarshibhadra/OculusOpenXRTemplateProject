# OculusOpenXRTemplateProject
 Empty Template Project to Start a Oculus(Rift + Quest) project on OpenXR

## Empty project-template to jumpstart your OpenXR Quest Vulkan project

If you want a quick template to jump-start your OpenXR Quest Vulkan project read on..

I didn't find any instructions on how to setup a blank Vulkan-based OpenXR project for Oculus Quest.
 The most helpful thing I found was this article by [@gayanediriweera](https://twitter.com/gayanediriweera) 
 where he shares how to run the hello_xr sample on Oculus Quest: https://gayanediriweera.github.io/code/2021/04/06/how-to-run-helloxr-on-oculus-quest.html. 

However that project was too tied with the OpenXR SDK-Source for my liking. I wanted
   1. A empty blank template for a Vulkan-based OpenXR project running on Oculus Quest 
   2.  No OpenXR source (apart from headers)
   3.  No non-essential CMakeLists.txt, 3rd party libraries
   4.  No cruft from non-Vulkan APIs, or non-Quest runtimes 

That is this project. The caveat is this is only for Vulkan and Windows x86-64 arch - which is the use-case I'm primarily concerned with.

### Requirements:
   1. Download and install the Vulkan SDK: https://vulkan.lunarg.com/doc/view/1.1.121.2/windows/getting_started.html
   2. Setup Android Studio per instructions here: https://developer.oculus.com/documentation/native/android/mobile-studio-basics/
   3. Download the Oculus OpenXR Mobile SDK here: https://developer.oculus.com/downloads/package/oculus-openxr-mobile-sdk/ 
   4. Download all the files in this project
 
### There are two ways you can get started:
   - Easy Way:
      1. Open downloaded project in Android Studio
      2. Select Edit -> Find -> Replace in Files Search and Replace: **my_vulkan_quest_openxr_app** w/ your project name
      3. Right click on the top-level-project and select Refactor -> Rename and rename w/ your project name
      4. Run project! Start customizing from main.cpp in `MyVulkanQuestOpenXRApp\src`.  

   - Hard Way:
	- This will give you a peek into what is going on under the hood
	- Steps:
	    1. Click File > New Project 
	    2. Select No Activity Template
	    3. For Min SDK put in 24 and hit Finish
	    4. Copy the **MyVulkanQuestOpenXRApp** subfolder in the downloaded project into the top-level of your new project. 
	    5. In the project explorer pane, right click on the app-module and select 'Add C++ to Module'
	    6. Select "Link an existing CMakeLists.txt" and find the CMakeLists.txt in the **MyVulkanQuestOpenXRApp** subfolder you just copied over.
	    7. In app/build.gradle verify this adds the following:	    
	    ```	    
		    externalNativeBuild {
			    cmake {
				path file('../MyVulkanQuestOpenXRApp/CMakeLists.txt')
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
	    9. In `app/src/main/AndroidManifest.xml` add
	    ```
	    <?xml version="1.0" encoding="utf-8"?>
	    <manifest xmlns:android="http://schemas.android.com/apk/res/android"
		package="com.mycompany.myvulkanquestopenxrapp"
		android:versionCode="1"
		android:versionName="1.0" android:installLocation="auto">
		<uses-feature android:name="android.hardware.vr.headtracking" android:required="true" />
		<application android:allowBackup="false" android:label="MyVulkanQuestOpenXRApp" android:hasCode="false">
		    <activity
			android:name="android.app.NativeActivity"
			android:theme="@android:style/Theme.Black.NoTitleBar.Fullscreen"
			android:launchMode="singleTask"
			android:screenOrientation="landscape"
			android:excludeFromRecents="false"
			android:configChanges="screenSize|screenLayout|orientation|keyboardHidden|keyboard|navigation|uiMode">
			<meta-data android:name="android.app.lib_name" android:value="my_vulkan_quest_openxr_app" />
			    <intent-filter>
				    <action android:name="android.intent.action.MAIN" />
				    <category android:name="com.oculus.intent.category.VR" />
				<category android:name="android.intent.category.LAUNCHER" />
			   </intent-filter>
		    </activity>
		 </application>
	     </manifest>
	    ```
	   10. Select Edit -> Find -> Replace in Files Search and Replace: **my_vulkan_quest_openxr_app** w/ your project name
	   11. Finally in `MyVulkanQuestOpenXRApp\CMakeLists.txt` change the path to where you downloaded the Oculus Quest OpenXR mobile SDK
	   ```
	   set_property(
	   TARGET
	   openxr_loader
	   PROPERTY
	   IMPORTED_LOCATION
	    C:/Users/suv/Documents/openxr_oculus_mobile_sdk_1014/OpenXR/Libs/Android/${ANDROID_ABI}/${CMAKE_BUILD_TYPE}/libopenxr_loader.so
	    )
	   ```
	   12. Run your app from Android Studio by `Shift+F10`. You should see the `hello_xr` sample app shared in the Khronos OpenXR SDK-Source repo.
	   13. Modify main.cpp in `MyVulkanQuestOpenXRApp\src` to start to customize your Vulkan app

### To run project on Rift
1. Point CMAKE to CMake.lists at root of project.
2. Generate Visual Studio x86-64 project at /build
3. Select hello_xr (or your project) as the "Startup Project" in Visual Studio
4. Add -g Vulkan as command arguments in Project properties
5. Build and run.


### To run project on Quest:
1. Open project root folder in Android Studio 
2. From command line Specify graphics plugin: adb shell setprop debug.xr.graphicsPlugin Vulkan.
3. Hit play


### License
The blank/template/starter app is the `hello_xr` sample from Khronos and is governed by their license here https://github.com/KhronosGroup/OpenXR-SDK-Source/blob/master/LICENSE. 
	
### References:
@gayanediriweera shared how to run the hello_xr sample from the  Khronos on Oculus Quest here: https://gayanediriweera.github.io/code/2021/04/06/how-to-run-helloxr-on-oculus-quest.html. 
This really helped me unpack what was going on under the hood of the gradle/CMake build process and helped me in putting this together!
