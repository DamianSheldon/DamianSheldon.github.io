---
layout: post
title: "Swift Learning Notes -- Properties"
date: 2014-06-24 15:28:20 +0800
comments: true
categories: [Archives, iOS Development] 
---

###Properties
“Properties associate values with a particular class, structure, or enumeration. ” -- Apple

####Type Properties
属于类型本身的properties称为Type Property.  

1)Value Type Properties (keyword: static)  
structrue, enumeration可以定义Stroed 和Computed type properties.

Stored type properties for value types can be variables or constants.  
NOTE:Unlike stored instance properties, you must always give stored type properties a default value. This is because the type itself does not have an initializer that can assign a value to a stored type property at initialization time.

Computed type properties are always declared as variable properties, in the same way as computed instance properties.

2)Reference Type Properties (keyword: class)  
class只可以定义Computed type properties.

####Instance Properties
1)Stroed Properties
In its simplest form, a stored property is a constant or variable that is stored as part of an instance of a particular class or structure. Stored properties can be either variable stored properties (introduced by the var keyword) or constant stored properties (introduced by the let keyword).

P.S:A lazy stored property is a property whose initial value is not calculated until the first time it is used. You indicate a lazy stored property by writing the @lazy attribute before its declaration.

2)Computed Properties
computed properties, which do not actually store a value. Instead, they provide a getter and an optional setter to retrieve and set other properties and values indirectly.

###Properties Observer
Property observers observe and respond to changes in a property’s value. 

You have the option to define either or both of these observers on a property:

willSet is called just before the value is stored.  
didSet is called immediately after the new value is stored.

