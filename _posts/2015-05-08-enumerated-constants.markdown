---
layout: post
title: "Enumerated constants"
date: 2015-05-08 16:38:20 +0800
comments: true
categories: [Archives, iOS Development]
keywords: Objective c, Enumerated constants
description: 
---

Objective-C中的枚举常量大致有以下四种情形：

```
// 1. unnamed enumerations
enum {
    NSBorderlessWindowMask      = 0,
    NSTitledWindowMask          = 1 << 0,
    NSClosableWindowMask        = 1 << 1,
    NSMiniaturizableWindowMask  = 1 << 2,
    NSResizableWindowMask       = 1 << 3
 
};

`- (void)setStyleMask:(NSUInteger)styleMask`

// 2 
typedef enum _NSMatrixMode {
    NSRadioModeMatrix           = 0,
    NSHighlightModeMatrix       = 1,
    NSListModeMatrix            = 2,
    NSTrackModeMatrix           = 3
} NSMatrixMode;

// 3
typedef NS_ENUM(NSInteger, UITableViewCellStyle) {
    UITableViewCellStyleDefault,
    UITableViewCellStyleValue1,
    UITableViewCellStyleValue2,
    UITableViewCellStyleSubtitle
};

// 4
typedef NS_OPTIONS(NSUInteger, NSLayoutFormatOptions) {
    NSLayoutFormatAlignAllLeft = (1 << NSLayoutAttributeLeft),
    NSLayoutFormatAlignAllRight = (1 << NSLayoutAttributeRight),
    NSLayoutFormatAlignAllTop = (1 << NSLayoutAttributeTop),
    NSLayoutFormatAlignAllBottom = (1 << NSLayoutAttributeBottom),
    NSLayoutFormatAlignAllLeading = (1 << NSLayoutAttributeLeading),
    NSLayoutFormatAlignAllTrailing = (1 << NSLayoutAttributeTrailing),
    NSLayoutFormatAlignAllCenterX = (1 << NSLayoutAttributeCenterX),
    NSLayoutFormatAlignAllCenterY = (1 << NSLayoutAttributeCenterY),
    NSLayoutFormatAlignAllBaseline = (1 << NSLayoutAttributeBaseline),
    NSLayoutFormatAlignAllLastBaseline = NSLayoutFormatAlignAllBaseline,
    NSLayoutFormatAlignAllFirstBaseline NS_ENUM_AVAILABLE_IOS(8_0) = (1 << NSLayoutAttributeFirstBaseline),
    
    NSLayoutFormatAlignmentMask = 0xFFFF,
    
    /* choose only one of these three
     */
    NSLayoutFormatDirectionLeadingToTrailing = 0 << 16, // default
    NSLayoutFormatDirectionLeftToRight = 1 << 16,
    NSLayoutFormatDirectionRightToLeft = 2 << 16,  
    
    NSLayoutFormatDirectionMask = 0x3 << 16,  
};

```

情形一相当于定义了常量，但不定义类型;  
情形二定义了一个NSMatrixMode类型;  

情形三定义了一个NSInteger的UITableViewCellStyle类型;  
情形四支持C++的枚举特性。

>The NS_OPTIONS macro is defined in different ways if compiling as C++ or not. If it’s not C++, it’s expanded out the same as NS_ENUM. However, if it is C++, it’s expanded out slightly differently. Why? The C++ compiler acts differently when two enumeration values are bitwise OR’ed together. This is something, as shown earlier, that is commonly done with the options type of enumeration. When two values are OR’ed together, C++ considers the resulting value to be of the type the enumeration represents: NSUInteger. It also doesn’t allow the implicit cast to the enumeration type.

Reference:  
Effective Objective-C 2.0  
[NS_ENUM & NS_OPTIONS](http://nshipster.cn/ns_enum-ns_options/)  
