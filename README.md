## EntropyString for Swift

<p align="center">
<a href="https://github.com/Carthage/Carthage"><img
src="https://img.shields.io/badge/Carthage-compatible-4BC51D.svg?style=flat"
alt="Carthage"></a>
<a href="https://cocoapods.org/pods/EntropyString"><img src="https://img.shields.io/cocoapods/v/EntropyString.svg" alt="CocoaPods - EntropyString"></a>
</p>

EntropyString provides easy creation of randomly generated strings of specific entropy using various character sets.

## <a name="TOC"></a>
 - [Build Status](#BuildStatus)
 - [Installation](#Installation)
 - [Basic Usage](#BasicUsage)
 - [Overview](#Overview)
 - [Real Need](#RealNeed)
 - [More Examples](#MoreExamples)
 - [Character Sets](#CharacterSets)
 - [Custom Characters](#CustomCharacters)
 - [Unique Characters](#UniqueCharacters)
 - [Efficiency](#Efficiency)
 - [Secure Bytes](#SecureBytes)
 - [Custom Bytes](#CustomBytes)

## <a name="BuildStatus"></a>Build Status
[![Build Status](https://travis-ci.org/EntropyString/EntropyString-Swift.svg?branch=master)](https://travis-ci.org/EntropyString/EntropyString-Swift)

[TOC](#TOC)

## <a name="Installation"></a>Installation

### Carthage

[Carthage](https://github.com/Carthage/Carthage) is a decentralized dependency manager for Objective-C and Swift.

1. Add the project to your [Cartfile](https://github.com/Carthage/Carthage/blob/master/Documentation/Artifacts.md#cartfile).

    ```
    github "EntropyString/EntropyString-Swift.git"
    ```

2. Run `carthage update` and follow the [Carthage getting started steps](https://github.com/Carthage/Carthage#getting-started).

3. Import module EntropyString

    ```swift
    import EntropyString
    ```

### CocoaPods

[CocoaPods](https://cocoapods.org/) is a centralized dependency manager for Objective-C and Swift.

1. Add the project to your [Podfile](https://guides.cocoapods.org/using/the-podfile.html).

    ```ruby
    use_frameworks!

    pod 'EntropyString', '~> 1.0.0'
    ```

2. Run `pod install` and open the `.xcworkspace` file to launch Xcode.

3. Import module EntropyString 

    ```swift
    import EntropyString
    ```

### Swift Package Manager

##### CxNote: Linux not yet supported until random byte generation issue resolved.

The [Swift Package Manager](https://swift.org/package-manager/) is a decentralized dependency manager for Swift.

1. Add the project to your `Package.swift`.

    ```swift
    import PackageDescription

    let package = Package(
        name: "YourProject",
        dependencies: [
            .Package(url: "https://github.com/EntropyString/EntropyString-Swift.git",
                     majorVersion: 1)
        ]
    )
    ```

2. Import module EntropyString 

    ```swift
    import EntropyString
    ```

[TOC](#TOC)

## <a name="BasicUsage"></a>Basic Usage

Calculate *bits* of entropy to cover **1 million strings** with a repeat *risk* of **1 in a billion** and generate a *string* using a set of **32 characters**:

  ```swift
  import EntropyString

  let bits = Entropy.bits(for: .ten06, risk: .ten09)
  var string = RandomString.entropy(of: bits, using: .charSet32)
  ```

  > 9Pp7MDDm7b9Dhb

Generate a *string* of the same *entropy* using a set of **16 characters** (hexadecimal):

  ```swift
  string = RandomString.entropy(of: bits, using: .charSet16)
  ```

  > d33fa62f572c4cc9c8

[TOC](#TOC)

## <a name="Overview"></a>Overview

`EntropyString` provides easy creation of randomly generated strings of specific entropy using various character sets. Such strings are needed when generating, for example, random IDs and you don't want the overkill of a GUID, or for ensuring that some number of items have unique names.

A key concern when generating such strings is that they be unique. To truly guarantee uniqueness requires that each newly created string be compared against all existing strings. The overhead of storing and comparing strings in this manner is often too onerous and a different strategy is desired.

A common strategy is to replace the *guarantee of uniqueness* with a weaker but hopefully sufficient *probabilistic uniqueness*. Specifically, rather than being absolutely sure of uniqueness, we settle for a statement such as *"there is less than a 1 in a billion chance that two of my strings are the same"*. This strategy requires much less overhead, but does require we have some manner of qualifying what we mean by, for example, *"there is less than a 1 in a billion chance that 1 million strings of this form will have a repeat"*.

Understanding probabilistic uniqueness requires some understanding of [*entropy*](https://en.wikipedia.org/wiki/Entropy_(information_theory)) and of estimating the probability of a [*collision*](https://en.wikipedia.org/wiki/Birthday_problem#Cast_as_a_collision_problem) (i.e., the probability that two strings in a set of randomly generated strings might be the same).  Happily, you can use `EntropyString` without a deep understanding of these topics.

We'll begin investigating `EntropyString` by considering our [Real Need](Read%20Need) when generating random strings.

[TOC](#TOC)

## <a name="RealNeed"></a>Real Need

Let's start by reflecting on a common statement of need of developers, who might say:

*I need random strings 16 characters long.*

Okay. There are libraries available that address that exact need. But first, there are some
questions that arise from the need as stated, such as:

  1. What characters do you want to use?
  2. How many of these strings do you need?
  3. Why do you need these strings?

The available libraries often let you specify the characters to use. So we can assume for now that question 1 is answered with:

*Hexadecimal IDs will do fine*.

As for question 2, the developer might respond:

*I need 10,000 of these things*.

Ah, now we're getting somewhere. The answer to question 3 might lead to the further qualification:

*I need to generate 10,000 random, unique IDs*.

And the cat's out of the bag. We're getting at the real need, and it's not the same as the original statement. The developer needs *uniqueness* across a total of some number of strings. The length of the string is a by-product of the uniqueness, not the goal.

As noted in the [Overview](Overview), guaranteeing uniqueness is difficult, so we'll replace that declaration with one of *probabilistic uniqueness* by asking:

  - What risk of a repeat are you willing to accept?

Probabilistic uniqueness contains risk. That's the price we pay for giving up on the stronger declaration of strict uniqueness. But the developer can quantify an appropriate risk for a particular scenario with a statement like:

*I guess I can live with a 1 in a million chance of a repeat*.

So now we've gotten to the real need:

*I need 10,000 random hexadecimal IDs with less than 1 in a million chance of any repeats*.

How do you address this need using a library designed to generate strings of specified length?  Well, you don't directly, because that library was designed to answer the originally stated need, not the real need we've uncovered. We need a library that deals with probabilistic uniqueness of a total number of some strings. And that's exactly what `EntropyString` does.

Let's use `EntropyString` to help this developer:

  ```swift
  import EntropyString

  let bits = Entropy.bits(total: 10000, risk: .ten06)
  var strings = [String]()
  for i in 0 ..< 5 {
    let string = RandomString.entropy(of: bits, using: .charSet16)
    strings.append(string)
  }
  print("Strings: \(strings)")
  ```

  > Strings: ["85e442fa0e83", "a74dc126af1e", "368cd13b1f6e", "81bf94e1278d", "fe7dec099ac9"]

To generate the IDs, we first use

  ```swift
    let bits = Entropy.bits(total: 10000, risk: .ten06)
  ```

to determine the bits of entropy needed to satisfy our probabilistic uniqueness of **10,000** strings with a **1 in a million** (ten to the sixth power) risk of repeat. We didn't print the result, but if you did you'd see it's about **45.51**. Then inside a loop we used

  ```swift
    let string = RandomString.entropy(of: bits, using: .charSet16)
  ```

to actually generate random strings using hexadecimal (charSet16) characters. Looking at the IDs, we can see each is 12 characters long. Again, the string length is a by-product of the characters used to represent the entropy we needed. And it seems the developer didn't really need 16 characters after all.

Finally, given that the strings are 12 hexadecimals long, each string actually has an information carrying capacity of 12 * 4 = 48 bits of entropy (a hexadecimal character carries 4 bits). That's fine. Assuming all characters are equally probable, a string can only carry entropy equal to a multiple of the amount of entropy represented per character. `EntropyString` produces the smallest strings that *exceed* the specified entropy.

[TOC](#TOC)

## <a name="MoreExamples"></a>More Examples

In [Real Need](#RealNeed) our developer used hexadecimal characters for the strings.  Let's look at using other characters instead.

We'll start with using 32 characters. What 32 characters, you ask? Well, the [Character Sets](#CharacterSets) section discusses the default characters available in `EntropyString` and the [Custom Characters](#CustomCharacters) section describes how you can use whatever characters you want. For now we'll stick to the provided defaults.

  ```swift
  import EntropyString

  var bits = Entropy.bits(total: 10000, risk: .ten06)
  var string = RandomString.entropy(of: bits, using: .charSet32)
  print("String: \(string)\n")
  ```

  > String: PmgMJrdp9h

We're using the same __bits__ calculation since we haven't changed the number of IDs or the accepted risk of probabilistic uniqueness. But this time we use 32 characters and our resulting ID only requires 10 characters (and can carry 50 bits of entropy, which as when we used 16 characters, is more than the required 45.51).

Now let's suppose we need to ensure the names of a handful of items are unique.  Let's say 30 items. And let's decide we can live with a 1 in 100,000 probability of collision (we're just futzing with some code ideas). Using hex characters:

  ```swift
  bits = Entropy.bits(total: 30, risk: .ten05)
  string = RandomString.entropy(of: bits, using: .charSet16)
  print("String: \(string)\n")
  ```

  > String: 766923a

Using the CharSet 4 characters:

  ```swift
  string = RandomString.entropy(of: bits, using: .charSet4)
  print("String: \(string)\n")
  ```

  > String: GCGTCGGGTTTTA

Okay, we probably wouldn't use 4 characters (and what's up with those characters?), but you get the idea.

Suppose we have a more extreme need. We want less than a 1 in a trillion chance that 10 billion strings of 32 characters repeat. Let's see, our risk (trillion) is 10 to the 12th and our total (10 billion) is 10 to the 10th, so:

  ```swift
  bits = Entropy.bits(total: .ten10, risk: .ten12)
  string = RandomString.entropy(of: bits, using: .charSet32)
  print("String: \(string)\n")
  ```

   > String: F78PmfGRNfJrhHGTqpt6Hn

Finally, let say we're generating session IDs. We're not interested in uniqueness per se, but in ensuring our IDs aren't predicatable since we can't have the bad guys guessing a valid ID. In this case, we're using entropy as a measure of unpredictability of the IDs. Rather than calculate our entropy, we declare it needs to be 128 bits (since we read on some web site that session IDs should be 128 bits).

  ```swift
  string = RandomString.entropy(of: 128, using: .charSet64)
  print("String: \(string)\n")
  ```

  > String: b0Gnh6H5cKCjWrCLwKoeuN

Using 64 characters, our string length is 22 characters. That's actually 132 bits, so we've got our OWASP requirement covered! 😌

Also note that we covered our need using strings that are only 22 characters in length. So long to using GUID strings which only carry 122 bits of entropy (for the commonly used version 4 anyway) and use string representations (hex and dashes) that are 36 characters in length.

[TOC](#TOC)

## <a name="CharacterSets"></a>Character Sets

As we've seen in the previous sections, `EntropyString` provides default characters for each of the supported character sets. Let's see what's under the hood.

  ```swift
  import EntropyString

  print("CharSet 64: \(RandomString.characters(for: .charSet64))\n")
  ```

The call to `RandomString.characters(for:)` returns the characters used for any of the sets defined by the `CharSet enum`. The following code reveals all the character sets.

  ```swift
  print("CharSet 32: \(RandomString.characters(for: .charSet32))\n")
  print("CharSet 16: \(RandomString.characters(for: .charSet16))\n")
  print("CharSet  8: \(RandomString.characters(for: .charSet8))\n")
  print("CharSet  4: \(RandomString.characters(for: .charSet4))\n")
  print("CharSet  2: \(RandomString.characters(for: .charSet2))\n")
  ```

The default character sets were chosen as follows:

  - CharSet 64: **ABCDEFGHIJKLMNOPQRSTUVWXYZabcdefghijklmnopqrstuvwxyz0123456789-_**
      * The file system and URL safe char set from [RFC 4648](https://tools.ietf.org/html/rfc4648#section-5).
  - CharSet 32: **2346789bdfghjmnpqrtBDFGHJLMNPQRT**
      * Remove all upper and lower case vowels (including y)
      * Remove all numbers that look like letters
      * Remove all letters that look like numbers
      * Remove all letters that have poor distinction between upper and lower case values.
      The resulting strings don't look like English words and are easy to parse visually.

  - CharSet 16: **0123456789abcdef**
      * Hexadecimal
  - CharSet  8: **01234567**
      * Octal
  - CharSet  4: **ATCG**
      * DNA alphabet. No good reason; just wanted to get away from the obvious.
  - CharSet  2: **01**
      * Binary

You may, of course, want to choose the characters used, which is covered next in [Custom Characters](#CustomCharacters).

[TOC](#TOC)

## <a name="CustomCharacters"></a>Custom Characters

Being able to easily generate random strings is great, but what if you want to specify your own characters. For example, suppose you want to visualize flipping a coin to produce entropy of 10 bits.

  ```swift
  import EntropyString

  let randomString = RandomString()
  var flips = randomString.entropy(of: 10, using: .charSet2)
  print("flips: \(flips)\n")
  ```

  > flips: 0101001110

The resulting string of __0__'s and __1__'s doesn't look quite right. You want to use the characters __H__ and __T__ instead.

  ```swift
  try! randomString.use("HT", for: .charSet2)
  flips = randomString.entropy(of: 10, using: .charSet2)
  print("flips: \(flips)\n")
  ```

  > flips: HTTTHHTTHH

Note that setting custom characters in the above code requires using an *instance* of `RandomString`, wheras in the previous sections we used *class* functions for all calls. The function signatures are the same in each case, but you can't change the static character sets used in the class `RandomString` (i.e., there is no `RandomString.use(_,for:)` function).

As another example, we saw in [Character Sets](#CharacterSets) the default characters for CharSet 16 are **01234567890abcdef**. Suppose you like uppercase hexadecimal letters instead.

  ```swift
  try! randomString.use("0123456789ABCDEF", for: .charSet16)
  let hex = randomString.entropy(of: 48, using: .charSet16)
  print("hex: \(hex)\n")
  ```

  > hex: 4D20D9AA862C

Or suppose you want a random password with numbers, lowercase letters and special characters.

  ```swfit
  try! randomString.use("1234567890abcdefghijklmnopqrstuvwxyz-=[];,./~!@#$%^&*()_+{}|:<>?", for: .charSet64)
  let password = randomString.entropy(of: 64, using: .charSet64)
  print("password: \(password)")
  ```

  > password: }4?0x*$o_=w

Note that `randomString.use(_,for:)` can throw an `Error`. The throw is actually a `RandomStringError` and will occur if the number of characters doesn't match the number required for the CharSet or if the characters are not all unique. The section on [Unique Characters](#UniqueCharacters) discusses these errors further.

[TOC](#TOC)

## <a name="UniqueCharacters"></a>Unique Characters

As noted in [Custom Characters](#CustomCharacters), specifying the characters to use for a set can fail if the number of characters is invalid or if any character repeats.  The desire for unique characters is due to the calculation of entropy, which includes the probability of the occurrence of each character. `EntropyString` assumes the characters are unique so that each has the exact same probability of occurrence.

  ```swift
  import EntropyString

  let randomString = RandomString()
  do {
    try randomString.use("0120", for: .charSet4)
  }
  catch {
    print(error)
  }
  ```

  > error: charsNotUnique

You can force the use of repeat characters. (BTW, don't do this unless you really know what you are doing.)

  ```swift
  try! randomString.use("0120", for: .charSet4, force: true)
  ```

Now we'll create a string by specifying an __entropy__ of __128__ bits and print the result.

  ```swift
  let string = randomString.entropy(of: 128, using: .charSet4)

  print("string: \(string)\n")
  ```

  > string: 2201121012112100012022010002011020212002200212100110022121201221

Looking at the string may not reveal a problem, but the display of the various character counts sure does!

  ```swift
  let zeros = string.characters.filter { $0 == "0" }.count
  let ones = string.characters.filter { $0 == "1" }.count
  let twos = string.characters.filter { $0 == "2" }.count

  print("counts:  0 -> \(zeros)  |  1 -> \(ones)  |  2 -> \(twos)\n")
  ```

  > counts:  0 -> 32  |  1 -> 15  |  2 -> 17

The string *does not have* __128 bits of entropy__! If all 4 characters in use are unique, we would expect each character in the string to provide __2 bits__ of information (entropy).  But since the character __0__ is *twice* as likely to occur as either __1__ or __2__, the actual entropy per character has been reduced to __1.5 bits__. So the strings generated only have __96__ bits of entropy.

[TOC](#TOC)

## <a name="Efficiency"></a>Efficiency

To efficiently create random strings, `EntropyString` generates the necessary number of bytes needed
for each string and uses those bytes in a bit shifting scheme to index into a character set. For example, consider generating strings from the `.charSet32` character set. There are __32__ characters in the set, so an index into an array of those characters would be in the range `[0,31]`. Generating a random string of `.charSet32` characters is thus reduced to generating a random sequence of indices in the range `[0,31]`.

To generate the indices, `EntropyString` slices just enough bits from the array of bytes to create each index. In the example at hand, 5 bits are needed to create an index in the range `[0,31]`. `EntropyString` processes the byte array 5 bits at a time to create the indices. The first index comes from the first 5 bits of the first byte, the second index comes from the last 3 bits of the first byte combined with the first 2 bits of the second byte, and so on as the byte array is systematically sliced to form indices into the character set. And since bit shifting and addition of byte values is really efficient, this scheme is quite fast.

The `EntropyString` scheme is also efficient with regard to the amount of randomness used. Consider the following common solution to generating random strings. To generated a character, an index into the available characters is create using `arc4random_uniform`. The code looks something like:

  ```swift
  for _ in 0..<len {
    let offset = Int(arc4random_uniform(charCount))
    let index = chars.index(chars.startIndex, offsetBy: offset)
    let char = chars[index]
    string += String(char)
  }
  ```

`arc4random_uniform` generates 32 bits of randomness, returned as an UInt32. The returned value is used to create an **index**. Suppose we're creating strings of **len** 16 using a **charCount** of 32. Each **char** consumes 32 bits of randomness (generated by `archrandom_uniform` per character) while only injecting 5 bits of entropy into **string**. But a string of length 16 using 32 possible characters has an entropy carrying capacity of 80 bits. So creating each **string** requires a total of 512 bits of randomness while only actually carrying 80 bits of that entropy forward in the string itself. That means 432 bits (84% of the total) of the generated randomness is simply thrown away.

Compare that to the `EntropyString` scheme. For the example above, slicing off 5 bits at a time requires a total of 80 bits (10 bytes). Creating the same strings as above, `EntropyString` uses 80 bits of randomness per string with no wasted bits. In general, the `EntropyString` scheme can waste up to 7 bits per string, but that's the worst case scenario and that's *per string*, not *per character*!

Fortunately you don't need to really understand how the bytes are efficiently sliced and diced to get the string. But you may want to know that [Secure Bytes](#SecureBytes) are used, and that's the next topic.

[TOC](#TOC)

## <a name="SecureBytes"></a>Secure Bytes

As described in [Efficiency](#Efficiency), `EntropyString` uses an underlying array of bytes to generate strings. The entropy of the resulting strings is, of course, directly tied to the randomness of the bytes used. That's an important point. Strings are only capable of carrying information (entropy), it's the random bytes that actually provide the entropy itself.

`EntropyString` automatically generates the necessary number of bytes needed for the strings using either `SecRandomCopyBytes` or `arc4random_buf`, both of which produce cryptographically-secure random byte. `SecRandomCopyBytes` is the stronger of the two, but can fail. Rather than propagate that failure, if `SecRandomCopyBytes` fails `EntropyString` falls back and uses`arc4random_buf` to generate the bytes. Though not as secure, `arc4random_buf` does not fail.

You may, however, want to know which routine was used to generate the underlying bytes for a string. `RandomString` provides an additional `inout` parameter in the `RandomString.entropy(for:using:secure)` function for this purpose.

  ```swift
  import EntropyString

  var secure = true
  RandomString.entropy(of: 20, using: .charSet32, secure: &secure)
  print("secure: \(secure)")
  ```

  > secure: true

If `SecRandomCopyBytes` is used, the __secure__ parameter will remain `true`; otherwise it will be flipped to `false`.

You can also pass in __secure__ as `false`, in which case the `entropy` call will not attempt to use `SecRandomCopyBytes` and will use `arc4random_buf` instead.

  ```swift
  secure = false
  RandomString.entropy(of: 20, using: .charSet32, secure: &secure)
  ```

Rather than have `EntropyString` generate bytes automatically, you can provide your own [Custom Bytes](#CustomBytes) to create a string, which is the next topic.

[TOC](#TOC)

## <a name="CustomBytes"></a>Custom Bytes

As described in [Secure Bytes](#SecureBytes), `EntropyString` automatically generates random bytes using either `SecRandomCopyBuf` or `arc4random_buf`. These functions are fine, but you may have a need to provide your own btyes, say for deterministic testing or to use a specialized byte genterator. The `RandomString.entropy(of:using:bytes)` function allows passing in your own bytes to create a string.

  ```swift
  import EntropyString

  let bytes: RandomString.Bytes = [250, 200, 150, 100]
  let string = try! RandomString.entropy(of: 30, using: .charSet32, bytes: bytes)
  print("String: \(string)\n")
  ```

  > string: Th7fjL
 
The __bytes__ provided can come from any source. However, the number of bytes must be sufficient to generate the string as described in the [Efficiency](#Efficiency) section.  `RandomString.entropy(of:using:bytes)` throws `RandomString.RandomError.tooFewBytes` if the string cannot be formed from the passed bytes.

  ```swift
  do {
    try RandomString.entropy(of: 32, using: .charSet32, bytes: bytes)
  }
  catch {
    print(error)
  }
  ```

  > error: tooFewBytes

[TOC](#TOC)
