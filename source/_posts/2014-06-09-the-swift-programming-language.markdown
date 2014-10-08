---
layout: post
title: "The Swift Programming Language"
date: 2014-06-09 16:56:43 +0800
comments: true
published: false
categories: [Archives, iOS Development] 
keywords: Swift, Programming, Language
description: Introduce swift Programming Language
---
##Data Types

###Constants and Variables
``` swift
let constant = "Hello, I'm a constant."

var variable = "Hello, I'm a variable."
```

###Primitive 
Bool, Int, Double, Float, String
1)Defining;
``` swift
[let | var] identifierName{: Primitive Type} {= value}

let boolConstant: Bool = true

var intVariable: Int = 7

let realConstant: Float = 3.1415926

var realVariable: Double = 0.89727

let stringConstant: String = "String Constant"

// Swift can infer type from initial value, so we can define as follow
let boolConstantV2 = true

var intVariableV2 = 7

let realConstantV2 = 3.1415926

var realVariableV2 = 0.89727

let stringConstantV2 = "String Constant"
```

2)Declaring;
``` swift
//Declaring variables
var declaringBool: Bool

var declaringInt: Int

var declaringReal: Double

var declaringString: String
```

3)Accessing;
``` swift
boolConstantV2

intVariableV2

realConstantV2

stringVariableV2

```

###Enumerations
An enumeration defines a common type for a group of related values and enables you to work with those values in a type-safe way within you code.

Supported Features:
1)Do not have to provide a value for each member;
2)computed properties to provide additional information about the enumeration's current value;
3)Instant method to provide functionality related to the values the enumeration represents;
4)Initializers to provide an initial member value;
5)Can be extended to expand their functionality beyond their original implementation;
6)Can conform to protocols to provide standard functionality.

1)Defining;
``` swift
enum Copasspoint {
    case North

    case South

    case West

    case East
}

// Associate Values
enum Barcode {
        
        case UPCA(Int, Int, Int)

        case QRCode(String)

    }

// Raw Values
enum ASCIIControlCharacter: Character {
        case Tab = "\t"

        case LineFeed = "\n"

        case CarriageReturn = "\r"

    }
```
2)Declaring;
``` swift
let constantEnum: ASCIIControlCharacter = ASCIIControlCharacter.Tab

var variableEnum: ASCIIControlCharacter
```

3)Accessing;
``` swift
var productBarcode: Barcode = Barcode.UPCA(8, 85909_51226, 3)

productBarcode = .QRCode("ABCDEFGHIJKLMNOP")
```

###Structures
1)Defining;
``` swift
struct Resolution {
        
    var width = 0
                
    var height = 0
                        
}
```
2)Declaring;
``` swift
let constantResolution = Resolution()

var variableResolution: Resolution

variableResolution = Resolution()
```
3)Accessing;
``` swift
// You can access the properties of an instance using dot syntax.
let width: Double = variableResolution.width
```

###Collection Types
Array, Dictionary

####Array
1)Defining;
``` swift
// Empty Array
// Full Syntax
var emptyArrayForSomeInts: Array<Int> = Array<Int>()

// Shorthand Syntax 

// Array<SomeType>'s shorthand form is SomeType[]

var emptyArrayForSomeIntsV2: Int[] = Int[]()

// Because Swift can infer type, so

var emptyArrayForSomeIntsV3 = Int[]()

// Alternatively, if the context already  provides type information, such as a function argument or an already-typed variable or constant, you can create an emptye array with an empty array literal, which is written as []:

emptyArrayForSomeInts.append(3)

emptyArrayForSomeInts = []

// Non-empty Array
// Array Literals
var shoppingListDefiningWithArrayLiterals: Array<String> = ["Eggs", "Milk"]

var shoppingListDefiningWithArrayLiteralsV2: String[] = ["Eggs", "Milk"]

var shoppingListDefiningWithArrayLiteralsV3 = ["Eggs", "Milk"]
```
2)Declaring;
``` swift
var declaringArray: Array<String>

var declaringArrayV2: String[]
```
3)Accessing;
a)subscript syntax;
``` swift
var firstItem  = shoppingList[0]
```
b)properties;
``` swift
var count: Int = shoppingList.count

var status: Bool = shoppingList.isEmpty
```
c)methods
``` swift
shoppingList.append("Flour")

shoppingList.insert("Maple Syrup", atIndex: 0)
```

####Dictionary
1)Defining;
``` swift
// Empty Dictionary

var anEmptyDictionary = Dictionary<Int, String>()

// If the context already provides type information, create an empty dictionary with empty dictionary literal, which is written as [:]

anEmptyDictionary[16] = "Sixteen"

anEmptyDictionary = [:]

var definingDictionary: Dictionary<String, String> = ["TYO": "Tokyo", "DUB": "Dublin"]

var definingDictionaryV2 = ["TYO": "Tokyo", "DUB": "Dublin"]
```
2)Declaring;
``` swift
var declaringDictionary: Dictionary<String, String>
```
3)Accessing;
a)subscript syntax;
``` swift
definingDictionary["LHR"] = "London Heathrow"
```
b)properties;
``` swift
let dictionaryCount = definingDictionary.count
```
c)methods;
``` swift
definingDictionary.updateValue("Dublin International", forKey: "DUB")
```
####Mutability of Collections
1)immutable: If you assign an array or a dictionary to a constant, that array or dictionary is immutable, and its size cannot be changed. 

a)Array: You not allowed to perform any action that has the potential to change the size of an immutable array, but you are allowed to set a new value for an existing index in the array.

b)Dictionary: You cannot replace the value for an existing key in the dictionary. An immutable dictionary's contents cannot be changed once they are set.

2)mutable: If you create an array or a dictionary and assign it to a variable, the collection that is created will be mutable. This means that you can change (or mutalbe) the size of the collection after it is created by adding more items to the collection, or by removing exsting items from the ones it already contains.

The Mutability behavior of Swift's Collection Types also affects how collection types instances are assigned and modified.

a)Assignment and Copy behavior for Array
The assignment and copy behavior for Swift's Array type is more complex than for its Dictionary type. Array provide C-like performance when you work with an array's contents and copies an array's contents only when coping is neccessary.

If you assign an Array instance to a constant or varible, or pass an Array instance as an argument to a function or method call, the contents of the array are *not* copied at the point that the assignment or call take place. Instead, both arrays share the same sequence of element values. When you modify an element value through one array, the result is observable through the other.

For Arrays, copying only takes place when you perform an action that has the potential to modify the *length* of the array. This includes appending, inserting, or removing items, or using a ranged subscript to replace a range of items in the array. If and when array copying does take place, the copy behavior for an array's contents is the same as for a dictionary's keys and values.
``` swift
var testArrayA = [1, 2, 3]

var testArrayB = testArrayA

var testArrayC = testArrayB

println(testArrayA[0])
// 1
println(testArrayB[0])
// 1
println(testArrayC[0])
// 1

testArrayA[0] = 42

println(testArrayA[0])
// 42

println(testArrayB[0])
// 42

println(testArrayC[0])
// 42


testArrayA.append(4)

testArrayA[0] = 777

println(testArrayA[0])
// 777

println(testArrayB[0])
// 42

println(testArrayC[0])
// 42
```

b)Assignment and Copy behavior for Dictionaries
Whenever you assign a Dictionary instance to a constant or variable, or pass a Dictionary instance as an argument to a function or method call, the dictionary is copied at the point that the assignment or call take place.
``` swift
var basedDictionary: Dictionary<String, Int> = ["Peter": 23, "Wei": 35, "Anish": 65, "Katay": 19]


var assignmentAndCopyBehaviorForBasedDictionary = basedDictionary

assignmentAndCopyBehaviorForBasedDictionary["Peter"] = 32

println(basedDictionary["Peter"])

// prints "23"
```

###Swift's Unique Types
Optional type, tuple

####Optional 
1)Declaring;
``` swift
var optionalInteger: Optional<Int>

var optionalIntegerV2: Int?
```
2)Accessing;
``` swift
otionalInteger = 1234567890
```
####Tuple
1)Declaring;
``` swift
var tupleExample = (statusCode:404, statusMessage: "Not Found")
```

2)Accessing;
``` swift
tupleExample.statusCode = 200

tupleExample.statusMessage = "OK"

tupleExample.0 = 401

tupleExample.1 = "Not Modify"
```

###Closures, Functions

####Closure
Defining;
``` swift
{
    (parameters) -> return type  in

    statements
}

parameters: constants, variables, inout, variadic, tuple
```

####Function
Define

``` swift
func functionName(parameter1: Bool, parameter2: Int, parameter3: Double, parameter4: Array<Float>, parameter5: Dictionary<String, String>, parameter6: Int...) -> String {
    
    // Function Body

    }

// Function Without Parameters

func functionWithoutParameters() -> String {
        return "Hello Swift."
    }

// Function With Mutiple Input Parameters

func functionWithMutipleInputParameters(start: Int, end: Int) -> Int {
        
        return end - start
    }

// Variadic Parameters

func variadicParametersExample(numbers: Double...) -> Double {
        
        var total: Double = 0

        for number in numbers {
                
                total += number
            }

        return total /Double(numbers.count)
    }

// Constant and variable Parameters

func variableParametersExample(var string: String, count: Int, pad: Character) -> String {
        
        let amountToPad = count - countElements(string)

        for _ in 1...amountToPad {
                
                string = pad + string
            }

        return string
    }

// In-Out Parameters

func inOutSwapTwoInts(inout a: Int, inout b: Int) {
        
        let temporaryA = a

        a = b

        b = temporaryA
    }

// Funtion Parameter Names

// Local parameter names 

func functionParameterNames(localParameterName: Int) {
        

    }

// External Parameter Names

func externalParameterNamesExample(externalParametername localParameterName: Int) {
        
    }

// Shorthand External Parameter Names

func shorthandExternalParameterNames(#string: String, #characterToFind: Character) -> Bool {
        
        for character in string {
                
                if character == characterToFind {
                        
                        return true
                    }
            }

            return false
    }

// Default Parameter Values

func defaultParameterValues(string s1: String, toString s2: String, withJoiner joiner: String = "-") -> String {
        
        return s1 + joiner + s2
    }

// External Names for parameters with Default Values

func joinDefaultValueVersion(s1: String, s2: String, joiner: String = "-") -> String {
        
        return s1 + joiner + s2
    }

func joinFullExternalParameterDefaultValuesVersion(#string: String, #toString: String, #withJoiner: String) -> String {
        

        return string + withJoiner + toString
    }

// Function Without Return Values

func functionWithoutReturnValues(#personName: String) {
        
        println("Good Bye \(personName)")
    }

// Function with Multiple Return Values

func functionWithMultipleReturnValues(#String: String) -> (vowels: Int, consonants: Int, others: Int) {
        
        var vowels = 0, consonants = 0, others = 0

        // some code 

        return (vowels, consonants, others)
    }

```

###Generics

###Classes
``` swift
// Class

class DefineClass {
        
            
}

// Class Instance

var classInstance = DefineClass()
```
###Properites

Properties associate values with a particular class, structure, or enumeration. 

Stroed properties store constant and variable values as part of an instance. Provided only by classes and structures. 

Computed properties calculate a value. Provided by classed, structures, and enumerations.

``` swift
// Stored Properties

struct FixedLengthRange {
        
        let length: Int

        var value: Int
    }

// Lazy Stored Properties

class LazyStoredPropertiesExample {
        
        var fileName: String = "data.txt"
    }

class LazyStoredPropertiesExampleCall {
        
        @lazy var importer = LazyStoredPropertiesExample()

        var data = String[]
    }

// Computed Properties
struct Point {
        
        var x = 0.0, y = 0.0

    }

struct Size {
        
        var width = 0.0, height = 0.0

    }

struct Rect {
        
        var origin = Point()

        var size = Size()

        var center: Point {
                
                get {
                        // Get method
                    }

                set {
                        // Set method
                    }
            }
    }

// Property ObserVers

class StepCounter {
        
        var totalSteps: Int = 0 {
                
                willSet(newTotalSteps) {
                        
                        // Some code...
                    }

                didSet {
                        // Some code...
                    }
            }
    }
```
###Initialization
Initialization is the process of preparing an instance of a class, structure, or enumeration for use. This process involves setting an initial value for each stored property on that instance and performing any other setup or initialization that is required before the new instance is ready to for use.


####Class Initializtion 

All of a class’s stored properties—including any properties the class inherits from its superclass—must be assigned an initial value during initialization.

Swift defines two kinds of initializers for class types to help ensure all stored properties receive an initial value. These are known as designated initializers and convenience initializers.

Designated initializers are the primary initializers for a class. A designated initializer fully initializes all properties introduced by that class and calls an appropriate superclass initializer to continue the initialization process up the superclass chain.

Convenience initializers are secondary, supporting initializers for a class. You can define a convenience initializer to call a designated initializer from the same class as the convenience initializer with some of the designated initializer’s parameters set to default values. You can also define a convenience initializer to create an instance of that class for a specific use case or input value type.

Designated initializers must always delegate up.
Convenience initializers must always delegate across.

Class initialization in Swift is a two-phase process. In the first phase, each stored property is assigned an initial value by the class that introduced it. Once the initial state for every stored property has been determined, the second phase begins, and each class is given the opportunity to customize its stored properties further before the new instance is considered ready for use.

Unlike subclasses in Objective-C, Swift subclasses do not not inherit their superclass initializers by default. Swift’s approach prevents a situation in which a simple initializer from a superclass is automatically inherited by a more specialized subclass and is used to create a new instance of the subclass that is not fully or correctly initialized.

However, superclass initializers are automatically inherited if certain conditions are met. In practice, this means that you do not need to write initializer overrides in many common scenarios, and can inherit your superclass initializers with minimal effort whenever it is safe to do so.

Assuming that you provide default values for any new properties you introduce in a subclass, the following two rules apply:

Rule 1
If your subclass doesn’t define any designated initializers, it automatically inherits all of its superclass designated initializers.

Rule 2
If your subclass provides an implementation of all of its superclass designated initializers—either by inheriting them as per rule 1, or by providing a custom implementation as part of its definition—then it automatically inherits all of the superclass convenience initializers.

These rules apply even if your subclass adds further convenience initializers.

