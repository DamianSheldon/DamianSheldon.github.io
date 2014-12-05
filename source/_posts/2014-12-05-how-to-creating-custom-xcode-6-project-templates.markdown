---
layout: post
title: "如何创建自定义的Xcode 6 工程模板"
date: 2014-12-05 14:34:04 +0800
comments: true
categories: [Archives, iOS Development]
keywords: Xcode 6 Project Templates
discription: How to creating custom Xcode 6 Project Templates
---

使用Xcode 6新建工程时，Apple准备了好些模板，这些模板写个Demo还是没有问题的，但是用来组织项目文件还是太弱了，所以情况经常是不得不每次去新建各种目录，这种重复性的劳动一来乏味，二来浪费时间。那么我们能不像创建自己的模板呢？这样新建的工程就能按自己的想法包含各种目录和文件。好消息是可以，坏消息是Apple没有提供相应的文档。虽然没有文档，还是试着来创建一个模板，每次都重复实在太烦（就是这么任性）。

既然没有文档，我们就把Apple的模板复制一份，在它的基础上修改成我们需要的样子。**/Applications/Xcode.app/Contents/Developer/Platforms/iPhoneOS.platform/Developer/Library/Xcode/Templates/Project\ Templates/iOS/Application/**有iOS所有工程模板。用户自定义的模板建议放到**~/Library/Developer/Xcode/Templates/**，目录如果不存在就创建。模板至少要包含两部分：一是扩展名为**.xctemplate**的文件夹；二是名称为**TemplateInfo.plist**的属性列表文件。好了，我们来创建一个自定义模板：

```
// Step 1:
$ mkdir ~/Library/Developer/Xcode/Templates/CocoaBite.xctemplate/

// Step 2:
$ cp /Applications/Xcode.app/Contents/Developer/Platforms/iPhoneOS.platform/Developer/Library/Xcode/Templates/Project\ Templates/iOS/Application/Single\ View\ Application.xctemplate/* ~/Library/Developer/Xcode/Templates/CocoaBite.xctemplate/

```
<!-- more -->

现在我们有了一个和Single View Application一样的模板，但这和我们目标还相差很远。接下来我们要做就是修改**TemplateInfo.plist**，让模板为我们做更多准备工作。

| Keys | Advice |
| ---- | ------ |
| Ancestors   | No          | Import settings from another Project Template.
| Concrete    | Recommended | Visible or hide Template form New Project Window.
| Definitions | No          | Work with workplace. Can write to file example source code.
| Description | Recommended | New Project Window - Project Template Description.
| Identifier  | Yes         | Project Template Unique Identifier.
| Kind        | Yes         | XCode Template Kind. Project or File.
| Nodes       | Recommended | Create or Copy Files to Project. Copy works
| Options     | Recommended | New Project Wizard >> Choose Options for Project. Add Text Fields, Combo Boxes.
| Platforms   | Recommended | Set Platform.
| Project     | Yes         | Set Project Build Settings.
| Targets     | Recommended | Set Build Settings, Build Phases for Targets. Link Libraries.

上面列出了TemplateInfo.plist大部分键，详细介绍在[这里][1]。

[1]: https://snipt.net/yonishin/about-xcode-4-project-template/

我自己新建的模板主要用到Definitions和Nodes，它们俩组合起来可以控制模板会新建哪些文件。例如我想让模板包含Models目录：

```
// Step 1:
$ cd ~/Library/Developer/Xcode/Templates/CocoaBite.xctemplate/

// Step 2:
$ mkdir -p Models

// Step 3: 编辑TemplateInfo.plist 如下图所示。

```

<div style="text-align: center" markdown="1">

	<img name="PropertyList" src="/images/PropertyList.png" width="623" height="836">

</div>

完整的模板放在[这里](https://github.com/DamianSheldon/Xcode-6-Project-Templates)。

##Reference

[Creating Custom Xcode 4 Project Templates](http://meandmark.com/blog/2011/12/creating-custom-xcode-4-project-templates/)  
[About XCode 4 Project Template (How To Create Custom Project Template)](https://snipt.net/yonishin/about-xcode-4-project-template/)  
[Xcode-6-Project-Templates](https://github.com/reidmain/Xcode-6-Project-Templates)  
