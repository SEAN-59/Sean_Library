# Foundation / Numbers, Data, and Basic Values
## 1. Numbers [Document Link](https://developer.apple.com/documentation/foundation/numbers_data_and_basic_values?language=objc)
### NSInteger : 정수 [Document Link](https://developer.apple.com/documentation/objectivec/nsinteger?language=objc)
- Describes an integer.   
### NSUInteger : 음수 없는 정수 [Document Link](https://developer.apple.com/documentation/objectivec/nsuinteger?language=objc)
- Describes an unsigned integer.
### NSDecimal : 10진수 숫자 [Document Link](https://developer.apple.com/documentation/foundation/nsdecimal?language=objc)
- A structure representing a base-10 number.   
### NSDecimalNumber : 10진수 숫자 [Document Link](https://developer.apple.com/documentation/foundation/nsdecimalnumber?language=objc)
###### 그런데 뭔가 메소드가 더 많음
- An object for representing and performing arithmetic on base-10 numbers.   
### NSNumber : 스칼라 값에 대한 객체  [Document Link](https://developer.apple.com/documentation/foundation/nsnumber?language=objc)
- An object wrapper for primitive scalar numeric values.   
### NSNumberFormatter :
- A formatter that converts between numeric values and their textual representations.   
<br>
<br>
---
## 2. Binary Data
### NSData
- A static byte buffer in memory.
### NSMutableData
- An object representing a dynamic byte buffer in memory.
<br>
<br>
---
## 3.URLs
### NSURL
- An object that represents the location of a resource, such as an item on a remote server or the path to a local file.
### NSURLComponents
- An object that parses URLs into and constructs URLs from their constituent parts.
### NSURLQueryItem
- An object representing a single name/value pair for an item in the query portion of a URL.
<br><br>
---
## 4.Unique Identifiers
### NSUUID
- A universally unique value that can be used to identify types, interfaces, and other items.
<br><br> 
---
## 5.Geometry
### CGFloat
- The basic type for all floating-point values.
### NSPoint
- A point in a Cartesian coordinate system.
### NSSize
- A two-dimensional size.
### NSRect
- A rectangle.
### NSAffineTransform
- A graphics coordinate transformation.
### NSEdgeInsets
- A description of the distance between the edges of two rectangles.
<br><br>
---
## 6.Ranges
### NSRange
- A structure used to describe a portion of a series, such as characters in a string or objects in an array.