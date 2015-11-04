---
layout: post
title: "iOS App 开发问题汇总（三）"
date: 2015-07-30 11:36:57 +0800
comments: true
categories: [Archives, iOS Development]
keywords: iOS App
discription: iOS App 开发问题汇总
---

### 1.问题描述：Checking the Entitlements for an iOS app Submission to the App Store

Making an Inspectable .ipa file  
In the Xcode Organizer, instead of Submit to the iOS App Store, do Save for Enterprise or Ad-Hoc Deployment. This will create a local copy of the .ipa file that would be submitted to the App Store.  
When asked to choose the provisioning profile to sign with, select the same distribution profile you use when submitting to the App Store. Take a screenshot of your choice (command-shift-3) so you can verify this step later. During submission, this screenshot will be the only record you have identifying which profile was used to sign the app.  
When asked to save the package, uncheck Save for Enterprise Distribution, then save the .ipa file.  
Checking the Entitlements of an .ipa file  
Find the .ipa file and change its the extension to .zip.  
Expand the .zip file. This will produce a Payload folder containing your .app bundle.  
Use the codesign tool to check the entitlements on the .app bundle like this:
$ codesign -d --entitlements :- "Payload/YourApp.app"
where YourApp.app is the actual name of your .app bundle.  
Use the security tool to check the entitlements of the app's embedded provisioning profile:
// should add cms, Apple may be a typo
$ security cms -D -i "Payload/YourApp.app/embedded.mobileprovision"

where YourApp.app is the actual name of your .app bundle.
<!-- more -->
### 2.问题描述：`<PBXGroup path=`Vendors` UUID=`4931326E1B4A0EBF00741B49`>` attempted to initialize an object with an unknown UUID. `4920B77E1B58900100C9789C` for attribute: `children`. This can be the result of a merge and  the unknown UUID is being discarded.

解决办法：
1. I opened the .pbxproj file in a text editor.  
a. Go to your .xcodeproj  
b. Right click -> Show package contents  
c. Open .pbxproj with a text editor.  
2. I searched for the UUID.  
3. Turns out that is a static library folder already removed from reference.  
4. I readd folder that contain the static library.  
5. Rerun pod installation.  
6. Issue not happening anymore! :D

Reference:https://github.com/CocoaPods/CocoaPods/issues/1822

### 3.How do I return to an older version of our code in Subversion?

解决办法:

```
svn update
svn merge -r 150:140 .
svn commit -m "Rolled back to r140"
```
Reference:http://stackoverflow.com/questions/814433/how-do-i-return-to-an-older-version-of-our-code-in-subversion

### 4. Add top layout guide in NIB file

解决办法：

```
-(void)viewDidLoad {
      if ([self respondsToSelector:@selector(edgesForExtendedLayout)])
         self.edgesForExtendedLayout = UIRectEdgeNone;
}
```
Reference:http://stackoverflow.com/questions/17074365/status-bar-and-navigation-bar-appear-over-my-views-bounds-in-ios-7

### 5. How do I set the height of tableHeaderView (UITableView) with autolayout?

Reference:http://stackoverflow.com/questions/20982558/how-do-i-set-the-height-of-tableheaderview-uitableview-with-autolayout

### 6. How to resize superview to fit all subviews with autolayout?

Reference:http://stackoverflow.com/questions/18118021/how-to-resize-superview-to-fit-all-subviews-with-autolayout/18155803#18155803

### 7.How to use c++ and objective-c together in XCode 4.2

```
1. Rename any .m file to .mm file;
2. If you look at the "Build Settings" there is a place written "Compile Sources As". There is a dropdown menu there where you can select Objective-C++. In the clang/gcc commandline I think it is "-x objective-c++".
```

Reference:http://stackoverflow.com/questions/9250655/how-to-use-c-and-objective-c-together-in-xcode-4-2

### 8.How do I change the status bar content to white?

Here are the steps involved in changing the contents of the status bar (battery, time, signal) to white:

Starting with iOS 8, you need to set the status bar color in the Project Editor, otherwise the status bar flashes from black to white when your app first starts:
Select the very first node in the Project Navigator to display the Project Editor.
Select your app under Targets.
Select the General tab and under the Deployment Info section, set Status Bar Style to Light.

Next, you need to modify your project's .plist file.
In the Project Navigator, expand the Supporting Files node.
Select the <My Project Name>-Info.plist file.
Click on the very first entry (Information Property List).
Click the plus (+) sign that appears in the first column.
At the bottom of the popup list, select View controller-based status bar appearance.
Make sure the Value column is set to NO.
If you want to change the status bar content color for all scenes in your app, add the following code to the AppDelegate class's application:didFinishLaunchingWithOptions method (add it before the return YES statement), or you can add it to an individual view controller's viewDidLoad method:
In Swift:

```
UIApplication.sharedApplication().statusBarStyle = UIStatusBarStyle.LightContent
```
In Objective-C:

```
[[UIApplication sharedApplication] setStatusBarStyle:UIStatusBarStyleLightContent];
```
If you are hiding the status bar in some scenes in your app, and you have a root navigation controller, you may need to add the following code to the root navigation controller:

In Swift:

```
override func preferredStatusBarStyle() -> UIStatusBarStyle {
        return UIStatusBarStyle.LightContent
}
```
In Objective-C:

```
- (UIStatusBarStyle)preferredStatusBarStyle
{
    return UIStatusBarStyleLightContent;
}
```
Reference:http://iosappsfornonprogrammers.com/forum/index.php?topic=1554.0

###9.Expression

```
(lldb) expr NSString *$str = (NSString *)[[NSString alloc] initWithData:data encoding:4]
(lldb) po $str

(lldb) e NSArray *$array = @[ @"Saturday", @"Sunday", @"Monday" ]
(lldb) p [$array count]
```

###10.fastlane add Slack Notifications
Create an Incoming WebHook and export this as SLACK_URL. Can send a message to #channel (by default), a direct message to @username or a message to a private group group with success (green) or failure (red) status.

Slack > Menu > API > Incoming Webhooks > Set up an incoming webhook integration in your Slack team to try it out.

Reference:https://github.com/KrauseFx/fastlane/blob/master/docs/Actions.md#notifications

###11.directory not found for option '-F/Applications/Xcode-beta.app/Contents/Developer/Platforms/iPhoneOS.platform/Developer/SDKs/iPhoneOS9.0.sdk/Developer/Library/Frameworks' "

Solution:Further to a migration of my Xcode project, from Xcode 6.4 to Xcode 7, I get the warning message below (after compilation) for the Test target :

directory not found for option '-F/Applications/Xcode.app/Contents/Developer/Platforms/iPhoneSimulator.platform/Developer/SDKs/iPhoneSimulator9.0.sdk/Developer/Library/Frameworks'
Actually I found something when comparing a new project vs an older one...

In the old project, the warning was only being produced by the test target of my projects. Under 'Search Paths', I found it was including two items under 'Framework Search Paths':

$(SDKROOT)/Developer/Library/Frameworks
$(inherited)
The new project kept the 'Framework Search Paths' empty.

Deleting those entries in my older project then removed the warning.

Reference:http://stackoverflow.com/questions/30827022/xcode-7-beta-library-search-path-warning

###12.ld: '/Users/dongmeiliang/Documents/ZHYJApp/ZHYJApp/ZHYJApp/Vendors/baiduMap/Release-iphoneos/BaiduMapAPI.framework/BaiduMapAPI(BMAddrList.o)' does not contain bitcode. You must rebuild it with bitcode enabled (Xcode setting ENABLE_BITCODE), obtain an updated library from the vendor, or disable bitcode for this target. for architecture arm64

Solution:disable bitcode for this target.
App target > Build Settings > Enable Bitcode > NO

Reference:http://stackoverflow.com/questions/30848208/new-warnings-in-ios9

###13.*** Assertion failure in +[AAPLListUtilities sharedApplicationGroupContainer], /Users/dongmeiliang/Downloads/ListerforwatchOSiOSandOSX/Objective-C/ListerKit/AAPLListUtilities.m:33
2015-09-22 15:31:20.966 Lister[84089:1396115] *** Terminating app due to uncaught exception 'NSInternalInconsistencyException', reason: 'The shared application group container is unavailable. Check your entitlements and provisioning profiles for this target. Details on proper setup can be found in the PDFs referenced from the README.'

Solution:Target > Lister > iCloud > Fix issue
			Target > App Group > Fix issue
			
Reference:http://stackoverflow.com/questions/27008730/new-lister-app-error-the-shared-application-group-container-is-unavailable-che

###14.How can I conditionally include a file based on build configuration in Xcode?

For each target for which you want to conditionally include the settings bundle, choose its Project from the source list, choose the target, and switch to the "Build Phases" tab.

Click the "Add Build Phase" button and choose "Add Run Script".

Then enter the following for the script:

```
if [ "${CONFIGURATION}" == "Debug" ]; then
    cp -r "${PROJECT_DIR}/Settings.bundle" "${BUILT_PRODUCTS_DIR}/${PRODUCT_NAME}.app"
fi
```
Reference:http://stackoverflow.com/questions/8496476/how-can-i-conditionally-include-a-file-based-on-build-configuration-in-xcode
### 15.Automatic Preferred Max Layout Width is not available on iOS versions prior to 8.0
Solution: 1. Go to Issue Navigator (CMD+8) and Select latest built with the warning 
2. Locate the warning(s) (search for "Automatic Preferred Max Layout") and press expand button on the right
3. Find the Object ID of the UILabel
4. Open the Storyboard and SEARCH (CMD+f) for the object. It will SELECT AND HIGHLIGHT the UILabel
5. Explictit set preferred layout width

Reference:http://stackoverflow.com/questions/25398312/automatic-preferred-max-layout-width-is-not-available-on-ios-versions-prior-to-8

