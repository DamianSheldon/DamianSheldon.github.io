---
layout: post
title: "iOS App 开发问题汇总（五）"
date: 2016-01-08 21:32:22 +0800
comments: true
categories: [Archives, iOS Development]
keywords: 
discription: 
---
### 1.On iOS 7 and later, how do I take a snapshot of my view and save the result in a UIImage?
Solution:

```
- (UIImage *)snapshot:(UIView *)view
{
    UIGraphicsBeginImageContextWithOptions(view.bounds.size, YES, 0);
    [view drawViewHierarchyInRect:view.bounds afterScreenUpdates:YES];
    UIImage *image = UIGraphicsGetImageFromCurrentImageContext();
    UIGraphicsEndImageContext();
    
    return image;
}
```
Reference:https://developer.apple.com/library/ios/qa/qa1817/_index.html

### 2.How to present a view controller on iOS7 without the status bar overlapping?

Solution:The easiest workaround I've found is to wrap the view controller you want to present inside a navigation controller, and then present that navigation controller.

```
MyViewController *vc = [MyViewController new];
UINavigationController *nvc = [[UINavigationController alloc] 
    initWithRootViewController:vc];
[self presentViewController:nvc animated:YES completion:nil];
```

### 3.代码创建 UITableView 时如何使用各种系统样式的 UITableViewCell?
Solution:

```
- (UITableViewCell *)tableView:(UITableView *)tableView cellForRowAtIndexPath:(NSIndexPath *)indexPath
{
    static NSString *sCellID = @"CellID";

    UITableViewCell *cell = [tableView dequeueReusableCellWithIdentifier:sCellID];
    if (!cell) {
        cell = [[UITableViewCell alloc] initWithStyle:UITableViewCellStyleDefault reuseIdentifier:sCellID];
    }
    // Configure the cell...
    
    return cell;
}
```
<!-- more -->
### 4.Settings bundle not showing up in iPhone settings

Solution:iOS 9.0.x - 9.2 Settings App Bug, Using the App Switch UI (double-press Home button) to kill the Settings App.

Reference:http://stackoverflow.com/questions/2683793/settings-bundle-not-showing-up-in-iphone-settings

### 5.单元测试的 Target 在测试导航栏里是灰色的不能点击测试。

Solution: Edit Scheme ... > Test > + > Select your test target.

### 6.How to change UITableViewCell Image to Circle in UITableView
Solution:

```
// Subclass UITableViewCell override layoutSubviews method

- (void)layoutSubviews
{
    [super layoutSubviews];
    
    self.imageView.layer.cornerRadius = CGRectGetHeight(self.imageView.frame) * 0.5;
    self.imageView.layer.masksToBounds = YES;
}
```

Reference:http://stackoverflow.com/questions/23350728/how-to-change-uitableviewcell-image-to-circle-in-uitableview

###7.How to downgrading Cocoapods?
A:

```
// First you can figure out which version of Cocoapods you are on with the command:
$ pod --version

// You can also see all the version of Cocoapods you have installed with this command:
$ sudo gem list cocoapods

// Next uninstall Cocoapods. If you have multiple version, you will have the choice of uninstalling all or a specific version.
$ sudo gem uninstall cocoapods

// Finally you can install the specific version with this command:
$ sudo gem install cocoapods -v 0.39.0
```

Reference:https://diego.org/2015/09/24/downgrading-cocoapods/

###8.How to get UISearchDisplayController to search only after search button is pressed？
A:Setup a delegate for the search bar and implement the searchBarSearchButtonClicked: method. Do your searching from that method. Just return NO from the shouldReloadTableForSearchString method.

Reference:http://stackoverflow.com/questions/18563529/how-to-get-uisearchdisplaycontroller-to-search-only-after-search-button-is-press

###9.Warning: the running version of Bundler is older than the version that created the lockfile. We suggest you upgrade to the latest version of Bundler by running `gem install bundler`.

A:

```
$ sudo gem install bundler
$ bundle install

...
An error occurred while installing git (1.2.9.1), and Bundler cannot continue.
Make sure that `gem install git -v '1.2.9.1'` succeeds before bundling.

$ sudo gem install git -v '1.2.9.1'
$ sudo gem install lowdown -v '0.0.5' --verbose
...
ERROR:  Error installing lowdown:
	lowdown requires Ruby version >= 2.1.0.
	
$ ruby --version
ruby 2.0.0p645 (2015-04-13 revision 50299) [universal.x86_64-darwin15]
// On OS X machines, you can use third-party tools (rbenv and RVM).
// Install RVM:
$ \curl -sSL https://get.rvm.io | bash -s stable --ruby=2.1.8

* To start using RVM you need to run `source /Users/dongmeiliang/.rvm/scripts/rvm`
    in all your open shell windows, in rare cases you need to reopen all shell windows.
$ source /Users/dongmeiliang/.rvm/scripts/rvm

// Check rvm has been installed successfully
$ rvm --help

$ bundle install
/System/Library/Frameworks/Ruby.framework/Versions/2.0/usr/lib/ruby/2.0.0/rubygems/dependency.rb:296:in `to_specs': Could not find 'bundler' (>= 0) among 13 total gem(s) (Gem::LoadError)
	from /System/Library/Frameworks/Ruby.framework/Versions/2.0/usr/lib/ruby/2.0.0/rubygems/dependency.rb:307:in `to_spec'
	from /System/Library/Frameworks/Ruby.framework/Versions/2.0/usr/lib/ruby/2.0.0/rubygems/core_ext/kernel_gem.rb:47:in `gem'
	from /usr/local/bin/bundle:22:in `<main>'
	
We need reinstall bundler after update ruby:
$ gem install bundler
Fetching: bundler-1.11.2.gem (100%)
Successfully installed bundler-1.11.2
Parsing documentation for bundler-1.11.2
Installing ri documentation for bundler-1.11.2
Done installing documentation for bundler after 6 seconds
1 gem installed

//bundle - Ruby Dependency Management
//gem -- RubyGems program, RubyGems is a sophisticated package manager for Ruby.
//rvm - The Ruby Version Manager
```

###10.Attempt to set a non-property-list object
```
Attempt to set a non-property-list object <CFBasicHash 0x15809b8d0 [0x19ec23b68]>{type = immutable dict, count = 12,

entries =>

	0 : id = <CFNumber 0xb000000000000043 [0x19ec23b68]>{value = +4, type = kCFNumberSInt64Type}

	1 : <CFString 0x156dda8d0 [0x19ec23b68]>{contents = "allFreeLifeId"} = <CFNull 0x19ec23ea8 [0x19ec23b68]>

	5 : <CFString 0x156fd04d0 [0x19ec23b68]>{contents = "portraitUri"} = <CFNull 0x19ec23ea8 [0x19ec23b68]>

	6 : mobile = <CFString 0x156df0dd0 [0x19ec23b68]>{contents = "18670789602"}

	11 : ryUserId = <CFString 0x156fbef80 [0x19ec23b68]>{contents = "AFL2016020310001627518670789602"}

	13 : birthDay = <CFNull 0x19ec23ea8 [0x19ec23b68]>

	14 : signature = <CFString 0x19a51d480 [0x19ec23b68]>{contents = ""}

	15 : token = <CFString 0x1580a3cc0 [0x19ec23b68]>{contents = "vL9oen0gHgJF7sRMWdaacY9LZ0Y0py8pcDvpcwTzcsCvfbB0kFB1PaDbWZ1c1GGZ8/jRvJ5KfX2oT3Sv+7Wb75r8W7FhoewF9l1Si6GJa2B48Xpc3rElo2a6FVG/X29zQOiOTHRjfiM="}

	16 : password = <CFString 0x1580d5050 [0x19ec23b68]>{contents = "E10ADC3949BA59ABBE56E057F20F883E"}

	20 : email = <CFNull 0x19ec23ea8 [0x19ec23b68]>

	21 : name = Julian

	22 : gender = <CFNull 0x19ec23ea8 [0x19ec23b68]>

}

 as an NSUserDefaults/CFPreferences value for key UserInfo
```

A:

```
// Write
NSData *dataFromUserInfo = [NSKeyedArchiver archivedDataWithRootObject:userInfo];
[[NSUserDefaults standardUserDefaults] setObject:dataFromUserInfo forKey:sUserInfoKey];

// Read
NSData *dataFromUserInfo = [[NSUserDefaults standardUserDefaults] objectForKey:sUserInfoKey];
NSDictionary *ret = [NSKeyedUnarchiver unarchiveObjectWithData:dataFromUserInfo];
```
Reference:http://stackoverflow.com/questions/19720611/attempt-to-set-a-non-property-list-object-as-an-nsuserdefaults

###11.Missing iOS Distribution signing identity
A:KeyChain Access > select the System keychain > View > Show Expired Certificates > delete the expired version of the Apple Worldwide Developer Relations Certificate Authority Intermediate certificate (expired on February 14, 2016)

Reference:http://stackoverflow.com/questions/32821189/xcode-7-error-missing-ios-distribution-signing-identity-for

###12. FMDB unable to open db under Application Support directory.
A:Unlike the Documents directory, the Application Support directory does not exist in the app's sandbox by default. You need to create it before you can use it.

```
NSURL * ApplicationSupportDirectory()
{
    NSURL *applicationSupportDir = URLOfDirectory(NSApplicationSupportDirectory);
    
    if (applicationSupportDir) {
        // If the directory does not exist, this method creates it.
        // This method is only available in OS X v10.7 and iOS 5.0 or later.
        [[NSFileManager defaultManager] createDirectoryAtURL:applicationSupportDir withIntermediateDirectories:YES attributes:nil error:nil];
    }
    
    return applicationSupportDir;
}
```

Reference:http://stackoverflow.com/questions/16204988/ios-cant-save-file-to-application-support-folder-but-can-to-documents
http://www.cocoawithlove.com/2010/05/finding-or-creating-application-support.html

###13.How to NSLog a Call stack when a program is running?
A:

```
// 1. Cocoa framework
NSLog(@"%@", [NSThread callStackSymbols]);

// 2. C API
#include <execinfo.h>

int size = 256;
void *stack[size];
size = backtrace(stack, size);

char **syms = backtrace_symbols(stack, size);
for (int i = 0; i < size; i++) {
    printf("Frame #%d: %s\n", i, syms[i]);
}
free(syms);
```

Reference:http://stackoverflow.com/questions/13319551/how-to-nslog-a-call-stack-when-a-program-is-running

###14.How does programmatically construct to-one and to-many relationship in Core Data?
A:

```
NSRelationshipDescription *department = [NSRelationshipDescription new];
department.name = @"department";
department.deleteRule = NSNullifyDeleteRule;
department.minCount = 0;
department.maxCount = 1;// max = 1 for to-one relationship
    
NSRelationshipDescription *employees = [NSRelationshipDescription new];
employees.name = @"employees";
employees.deleteRule = NSNullifyDeleteRule;
employees.minCount = 0;
employees.maxCount = 0;// max = 0 for to-many relationship
```

Reference:http://stackoverflow.com/questions/13743242/adding-relationships-in-nsmanagedobjectmodel-to-programmatically-created-nsentit


