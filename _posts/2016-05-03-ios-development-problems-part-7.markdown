---
layout: post
title: "iOS App 开发问题汇总（七）"
date: 2016-05-03 17:00:04 +0800
comments: true
categories: [Archives, iOS Development]
keywords: iOS
description: iOS development problems
---

### 1.如何设置透明的导航栏并且去掉其底部灰色的分隔线？
A:可以通过设置导航栏的背景图和阴隐图为透明的图片来实现。

```
- (void)viewDidAppear:(BOOL)animated
{
    [super viewDidAppear:animated];
    
    self.translucent = self.navigationController.navigationBar.isTranslucent;
    self.navigationController.navigationBar.translucent = YES;
    
    self.previousImage = [self.navigationController.navigationBar backgroundImageForBarMetrics:UIBarMetricsDefault];
    
    UIImage *newImage = [UIImage imageNamed:@"TransparentPixel"];
    [self.navigationController.navigationBar setBackgroundImage:newImage forBarMetrics:UIBarMetricsDefault];
    
    self.previousShadowImage = self.navigationController.navigationBar.shadowImage;
    self.navigationController.navigationBar.shadowImage = newImage;
}

- (void)viewWillDisappear:(BOOL)animated
{
    [super viewWillDisappear:animated];
    
    self.navigationController.navigationBar.translucent = self.translucent;
    
    [self.navigationController.navigationBar setBackgroundImage:self.previousImage forBarMetrics:UIBarMetricsDefault];
    self.navigationController.navigationBar.shadowImage = self.previousShadowImage;
}
```

Reference:Apple's sample code Customizing UINavigationBar  
[Separator between navigation bar and view - iOS 7](http://stackoverflow.com/questions/18921082/separator-between-navigation-bar-and-view-ios-7)

<!-- more -->

### 2. 自定义的View设置cornerRadius不生效

### 3. CocoaPods 升级到1.0，Xcode 一直报 header file not found.
A: 仔细查看了 Xcode 的编译过程，头文件的路径是包含了的，于是怀疑是Xcode的自身可能存在问题。经过实验，可以用下面的方法解决：

1. Clean project
2. Window > Projects > Your Project > Delete Derived Data
3. Build

### 4. No codesigning identities (i.e. certificate and private key pairs) that match the team ID: xxx
A:

* Go to Xcode preferences, view details of the Apple ID, and delete the provisioning file that's complaining, redownload.
* Go to the Keychain Access, and delete the development certificate that's related to the provisioning file you just deleted.

Reference:[iOS 7.0 No code signing identities found](http://stackoverflow.com/questions/19197497/ios-7-0-no-code-signing-identities-found)

### 5. How to load image from CocoaPods resource bundle ?
A:

```
  s.resource_bundles = {
    'DMLSelector' => ['DMLSelector/Assets/*.png']
  }
  
  NSBundle *bundle = [NSBundle bundleForClass:[self class]];
  self.imageView.image = [UIImage imageNamed:@"DMLSelector.bundle/Checked" inBundle:bundle compatibleWithTraitCollection:nil];
```

Reference:[Load image from CocoaPods resource bundle](http://stackoverflow.com/questions/23835052/load-image-from-cocoapods-resource-bundle)

### 6. How to specify NSLineBreakMode in boundingRectWithSize?
A: Use NSParagraphStyleAttributeName & NSParagraphStyle:

```
NSMutableParagraphStyle *paragraph = [[NSMutableParagraphStyle alloc] init];
paragraph.lineBreakMode = NSLineBreakByWordWrapping; //e.g.

CGSize size = [label.text boundingRectWithSize: constrainedSize options:NSStringDrawingUsesLineFragmentOrigin attributes: @{ NSFontAttributeName: label.font, NSParagraphStyleAttributeName: paragraph } context: nil].size;
```
Reference:[How to specify NSLineBreakMode in boundingRectWithSize?](http://stackoverflow.com/questions/20631464/how-to-specify-nslinebreakmode-in-boundingrectwithsize)

### 7.

```
INFO [2016-06-01 15:42:30.78]: ---------------------------------------------------------------------
INFO [2016-06-01 15:42:30.78]: --- Step: fir p ../BBCApp.ipa -T token ---
INFO [2016-06-01 15:42:30.78]: ---------------------------------------------------------------------
INFO [2016-06-01 15:42:30.78]: $ fir p ../BBCApp.ipa -T c977500277789d01cc67d82750057858
INFO [2016-06-01 15:42:34.31]: ▸ /System/Library/Frameworks/Ruby.framework/Versions/2.0/usr/lib/ruby/2.0.0/rubygems/dependency.rb:296:in `to_specs': Could not find 'fir-cli' (>= 0) among 206 total gem(s) (Gem::LoadError)
INFO [2016-06-01 15:42:34.31]: ▸ from /System/Library/Frameworks/Ruby.framework/Versions/2.0/usr/lib/ruby/2.0.0/rubygems/dependency.rb:307:in `to_spec'
INFO [2016-06-01 15:42:34.31]: ▸ from /System/Library/Frameworks/Ruby.framework/Versions/2.0/usr/lib/ruby/2.0.0/rubygems/core_ext/kernel_gem.rb:47:in `gem'
INFO [2016-06-01 15:42:34.31]: ▸ from /usr/local/bin/fir:22:in `<main>'
```

A: 问题的原因是找不到fir这个命令，可以直接指定绝对路径解决。

```
sh "/Library/Ruby/Gems/2.0.0/gems/fir-cli-1.5.0/bin/fir p ../BBCApp.ipa -T token"
```

### 8. 怎么获取iPhone上已安装App的ipa文件？
A: 可以利用iTunes下载已购的功能，虽然 12.7 版本不支持，但老版本支持，所以可以安装老版本。

You can have iTunes 12.7 and iTunes 12.6.3 installed side-by-side.

1.	Install iTunes 12.6.3 from the links above.
2.	Copy the iTunes app and paste it to where you want, rename if necessary.
3.	Hold Option and double click iTunes. Create a new library in a separate location. iTunes 12.6.3 and iTunes 12.7 can't use the same library because iTunes 12.7 will update the file and iTunes 12.6.3 won't be able to understand it.
4.	Manually download the latest iTunes from Apple's website and run the installer to update your main copy of iTunes.
5.	(Optional) Change your iTunes 12.6.3 icon to a different icon to help distinguish between the two. (Right-click, Get Info, drag new icon onto existing icon).
Important: You will need to Option-Open each time you switch versions so that you can select the correct library and don't update the older library to 12.7 format.

Reference:[How do I download an iOS App (IPA) file to my Mac after iTunes 12.7 update?](https://apple.stackexchange.com/questions/298391/how-do-i-download-an-ios-app-ipa-file-to-my-mac-after-itunes-12-7-update)  

### 9. UICollectionView's cellForItemAtIndexPath is not being called
A: 

```
self.automaticallyAdjustsScrollViewInsets = NO;
```

Reference:[UICollectionView's cellForItemAtIndexPath is not being called](http://stackoverflow.com/questions/14668781/uicollectionviews-cellforitematindexpath-is-not-being-called?rq=1)

### 10. iOS supported audio formats?
A:

```
@"aac",@"adts",@"ac3",@"aif",@"aiff",@"aifc",@"caf",@"mp3",@"mp4",@"m4a",@"snd",@"au",@"sd2",@"wav"
```

Reference:[Supported Audio file formats in iPhone](http://stackoverflow.com/questions/1761460/supported-audio-file-formats-in-iphone)  
[Audio Tutorial for iOS: File and Data Formats](https://www.raywenderlich.com/69365/audio-tutorial-ios-file-data-formats-2014-edition)

### 11. How do I put the image on the right side of the text in a UIButton?

A:

```
CGFloat spacing          = 3;
    CGFloat insetAmount      = 0.5 * spacing;

    // First set overall size of the button:
    button.contentEdgeInsets = UIEdgeInsetsMake(0, insetAmount, 0, insetAmount);
    [button sizeToFit];

    // Then adjust title and image insets so image is flipped to the right and there is spacing between title and image:
    button.titleEdgeInsets   = UIEdgeInsetsMake(0, -button.imageView.frame.size.width - insetAmount, 0,  button.imageView.frame.size.width  + insetAmount);
    button.imageEdgeInsets   = UIEdgeInsetsMake(0, button.titleLabel.frame.size.width + insetAmount, 0, -button.titleLabel.frame.size.width - insetAmount);
```

Reference:[How do I put the image on the right side of the text in a UIButton?](http://stackoverflow.com/questions/7100976/how-do-i-put-the-image-on-the-right-side-of-the-text-in-a-uibutton)

extension:How to position image top title bottom in a UIButton?
A:

```
// Objc
-(void)imageTopTitleBottom:(UIButton *)button
{
    // the space between the image and text
    CGFloat spacing = 3.0;

    // lower the text and push it left so it appears centered
    //  below the image
    CGSize imageSize = button.imageView.image.size;
    button.titleEdgeInsets = UIEdgeInsetsMake(0.0, - imageSize.width, - (imageSize.height + spacing), 0.0);

    // raise the image and push it right so it appears centered
    //  above the text
    CGSize titleSize = [button.titleLabel.text sizeWithAttributes:@{NSFontAttributeName: button.titleLabel.font}];
    button.imageEdgeInsets = UIEdgeInsetsMake(- (titleSize.height + spacing), 0.0, 0.0, - titleSize.width);
}

// Swift
func imageTopTitleBottom() -> Void {
    if let buttonImage = self.image(for: .normal), let title = self.title(for: .normal) {
        let spacing: CGFloat = 2.0

            let imageSize = buttonImage.size

            self.titleEdgeInsets = UIEdgeInsetsMake(0, -imageSize.width, -(imageSize.height + spacing), 0)

            let titleString = title as NSString

            let stringSize = titleString.size(attributes: [NSFontAttributeName: self.titleLabel!.font])

            self.imageEdgeInsets = UIEdgeInsetsMake(-(stringSize.height + spacing), 0, 0, -stringSize.width)
    }
}
```

### 12. duplicate symbol _iToastDuration

```
enum iToastDuration {
	iToastDurationLong = 10000,
	iToastDurationShort = 1000,
	iToastDurationNormal = 3000
}iToastDuration;
```
A: 

```
typedef enum iToastDuration {
	iToastDurationLong = 10000,
	iToastDurationShort = 1000,
	iToastDurationNormal = 3000
}iToastDuration;
```

### 13. Detecting taps on attributed text in a UITextView in iOS
A:

```
import UIKit
class ViewController: UIViewController, UIGestureRecognizerDelegate {

    @IBOutlet weak var textView: UITextView!

    override func viewDidLoad() {
        super.viewDidLoad()

        // Create an attributed string
        let myString = NSMutableAttributedString(string: "Swift attributed text")

        // Set an attribute on part of the string
        let myRange = NSRange(location: 0, length: 5) // range of "Swift"
        let myCustomAttribute = [ "MyCustomAttributeName": "some value"]
        myString.addAttributes(myCustomAttribute, range: myRange)

        textView.attributedText = myString

        // Add tap gesture recognizer to Text View
        let tap = UITapGestureRecognizer(target: self, action: #selector(myMethodToHandleTap(_:)))
        tap.delegate = self
        textView.addGestureRecognizer(tap)
    }

    func myMethodToHandleTap(sender: UITapGestureRecognizer) {

        let myTextView = sender.view as! UITextView
        let layoutManager = myTextView.layoutManager

        // location of tap in myTextView coordinates and taking the inset into account
        var location = sender.locationInView(myTextView)
        location.x -= myTextView.textContainerInset.left;
        location.y -= myTextView.textContainerInset.top;

        // character index at tap location
        let characterIndex = layoutManager.characterIndexForPoint(location, inTextContainer: myTextView.textContainer, fractionOfDistanceBetweenInsertionPoints: nil)

        // if index is valid then do something.
        if characterIndex < myTextView.textStorage.length {

            // print the character index
            print("character index: \(characterIndex)")

            // print the character at the index
            let myRange = NSRange(location: characterIndex, length: 1)
            let substring = (myTextView.attributedText.string as NSString).substringWithRange(myRange)
            print("character at index: \(substring)")

            // check if the tap location has a certain attribute
            let attributeName = "MyCustomAttributeName"
            let attributeValue = myTextView.attributedText.attribute(attributeName, atIndex: characterIndex, effectiveRange: nil) as? String
            if let value = attributeValue {
               print("You tapped on \(attributeName) and the value is: \(value)")
            }

        }
    }
}
```
Reference:[Detecting taps on attributed text in a UITextView in iOS](http://stackoverflow.com/questions/19332283/detecting-taps-on-attributed-text-in-a-uitextview-in-ios?rq=1)

### 14. iOS 开发中如何获取农历？
A: iOS 8 以后 Apple 内置了农历支持，可以得到类似**2016丙申年五月十七星期二**的信息，如果你只需要这些信息就够了，那使用内置的农历就可以了。如果还想获取类似**壬辰年癸卯月庚辰日**，那可以用[LunarCalendar](https://github.com/autopear/LunarCalendar)。

```
    NSCalendar *chineseCalendar = [[NSCalendar alloc] initWithCalendarIdentifier:NSCalendarIdentifierChinese];

    NSDateFormatter *dateFormatter = [NSDateFormatter new];
    dateFormatter.locale = [NSLocale localeWithLocaleIdentifier:@"zh_Hans_CN"];
    dateFormatter.dateStyle = NSDateFormatterFullStyle;
    dateFormatter.calendar = chineseCalendar;
    
    NSLog(@"%@", [dateFormatter stringFromDate:[NSDate date]]);
    
    // Output:2016丙申年五月十七星期二
```

Reference:[What does Apple mean when it says iOS 8 will include lunar calendar support?](https://www.quora.com/What-does-Apple-mean-when-it-says-iOS-8-will-include-lunar-calendar-support)   
[NSCalendarIdentifierRepublicOfChina vs NSCalendarIdentifierChinese?](http://stackoverflow.com/questions/27878696/nscalendaridentifierrepublicofchina-vs-nscalendaridentifierchinese)  

### 15. How to fix tableView's tableHeaderView overlap first cell?
A: The core of solution is when you change the frame of tableHeaderView, you need reassign it to tableView.

```
    UIView *headerView = self.tableHeaderView;

    [headerView setNeedsLayout];
    [headerView layoutIfNeeded];
    CGFloat height = [headerView systemLayoutSizeFittingSize:UILayoutFittingCompressedSize].height;

    headerView.frame = ({
        CGRect headerFrame = headerView.frame;
        headerFrame.size.height = height;
        headerFrame;
    });

    self.tableHeaderView = headerView;
```

Reference:[tableHeaderView on top of first cell](http://stackoverflow.com/questions/28782319/tableheaderview-on-top-of-first-cell)  

