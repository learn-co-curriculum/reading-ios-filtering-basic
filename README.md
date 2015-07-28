# Introduction To Filtering (`NSPredicate`)

*Far over the misty mountains cold  
To dungeons deep and caverns old  
We must away ere break of day  
To seek the pale enchanted gold.*  
—from *The Hobbit* by J.R.R. Tolkien

## Objectives

1. Recognize when to employ a filter.
2. Review localized filtering with loops and conditionals.
3. Create a filter using `NSPredicate` and its format string.
4. Apply the filter and print the resulting array.
5. Learn the basic comparators.
6. Learn the string comparators.
7. Use the wildcards `?` and `*` correctly when using the `LIKE` comparator.
8. Select the case-insensitive `[c]` and diactric-insensitive `[d]` options for string comparators.
9. Combine or invert predicates with `AND`, `OR`, and `NOT` operators.

## The Usefulness Of Filtering

![](https://curriculum-content.s3.amazonaws.com/ios/reading-ios-filtering-basic/pileOfDwarves.gif)

Anytime that we begin interacting with a large set of data, filtering out the relevant pieces of information becomes increasingly important. Imagine the real-world case of needing to sift through thousands of contacts to find only those with a specific zip code or telephone area code. Constructing a filter is the typical way to get the computer to do the sifting for you. In fact, this is how "search" bars typically work.

In this reading, we'll be working with a nested data structure (an array with 14 dictionaries) containing information (name, age, and height) on each of the fourteen adventuring heroes from J.R.R. Tolkien's *The Hobbit*. This set of sample data is sufficiently large to meaningfully apply a few different filters, and should be inferred as being "in scope" for all of the examples throughout this reading:

```objc
NSArray *middleEarthers = @[ @{ @"name"   : @"Bilbo"  ,
                                @"age"    : @50       ,
                                @"height" : @1.27     } ,
                             @{ @"name"   : @"Thorin" ,
                                @"age"    : @195      ,
                                @"height" : @1.49     } ,
                             @{ @"name"   : @"Balin"  ,
                                @"age"    : @178      ,
                                @"height" : @1.38     } ,
                             @{ @"name"   : @"Dwalin" ,
                                @"age"    : @169      ,
                                @"height" : @1.50     } ,
                             @{ @"name"   : @"Bifur"  ,
                                @"age"    : @155      ,
                                @"height" : @1.35     } ,
                             
                             @{ @"name"   : @"Bofur"  ,
                                @"age"    : @155      ,
                                @"height" : @1.45     } ,
                             @{ @"name"   : @"Bombur" ,
                                @"age"    : @155      ,
                                @"height" : @1.35     } ,
                             @{ @"name"   : @"Fíli"   ,
                                @"age"    : @82       ,
                                @"height" : @1.35     } ,
                             @{ @"name"   : @"Kíli"   ,
                                @"age"    : @77       ,
                                @"height" : @1.43     } ,
                             @{ @"name"   : @"Glóin"  ,
                                @"age"    : @158      ,
                                @"height" : @1.41     } ,
                             
                             @{ @"name"   : @"Óin"    ,
                                @"age"    : @167      ,
                                @"height" : @1.45     } ,
                             @{ @"name"   : @"Dori"   ,
                                @"age"    : @155      ,
                                @"height" : @1.36     } ,
                             @{ @"name"   : @"Ori"    ,
                                @"age"    : @155      ,
                                @"height" : @1.35     } ,
                             @{ @"name"   : @"Gandalf",
                                @"age"    : @2019     ,
                                @"height" : @1.80     }
                             ];
```
**A Note On The Data For Fellow Tolkien Nerds:** *The characters' [heights][heights] are extrapolated from the heights of the actors cast in the Peter Jackson films, while the characters' [ages][ages] are based on their descriptions from the book. For the dwarves without an explicitly stated age, they have been listed as being 155 years old—the median age of the possible range. [Gandalf's listed age][Gandalf_age] of 2,019 years is that of his physical form in Middle Earth; as a Maiar, he is some 11,000 years old, having been created before recorded history.*

## Filtering With Loops

It's likely that you've already used loops and conditionals to manually construct a filter for a specific situation. Filtering this way is entirely acceptable for simple searches. However, this approach has its limitations regarding search complexity and code readability.

Let's write a filter using a loop and a conditional to find the oldest (and wisest) adventurers in the party:

```objc
NSMutableArray *elders = [ @[] mutableCopy ];

for (NSDictionary *character in middleEarthers) {
    if ([character[@"age"] integerValue] > 155 ) {
        [elders addObject:character];
    }
}

for (NSDictionary *character in elders) {
    NSLog(@"%@ is %@ years old.", character[@"name"], character[@"age"]);
}
```
This will print, in some order:

```
Thorin is 195 years old.
Balin is 178 years old.
Dwalin is 169 years old.
Glóin is 158 years old.
Óin is 167 years old.
Gandalf is 2019 years old.
```

But that's a lot of code to write that isn't very readable; it also isn't portable—we'd have to copy and paste our filter if we wanted to access it somewhere else in our code, or we could write it into a custom method. Yuck! Fortunately, Objective-C has a built-in foundation class for creating filter as objects. Welcome to the world of `NSPredicate`.

## Filtering With `NSPredicate`

![](https://curriculum-content.s3.amazonaws.com/ios/reading-ios-filtering-basic/dinnerMusic.gif)

The class' name is derived from the field of "[predicate logic][predicate_logic]", though the strict definition of that phrase is somewhat ambiguous. Using `NSPredicate` instead of a loop allows us to not only accomplish the same filter in two lines of code every time, but it also allows us to access the same filter parameters from elsewhere in our code by passing around the `NSPredicate` object that we created (this is most commonly accomplished through using properties which we'll discuss in the next topic).

Let's rewrite our filter from above using an `NSPredicate`:

```objc
NSPredicate *elderPredicate = [NSPredicate predicateWithFormat:@"age > 155"];
NSArray *elders = [middleEarthers filteredArrayUsingPredicate:elderPredicate];

for (NSDictionary *character in elders) {
    NSLog(@"%@ is %@ years old.", character[@"name"], character[@"age"]);
}
```
This will print, in some order:

```
Thorin is 195 years old.
Balin is 178 years old.
Dwalin is 169 years old.
Glóin is 158 years old.
Óin is 167 years old.
Gandalf is 2019 years old.
```
Exactly the same!

So we got the same result as our loop filter, but what exactly did we just do?

### Breaking Down The Syntax

Up above we wrote two magical lines that created and utilized a filter with `NSPredicate`. Since `NSPredicate` uses its own internal syntax passed to it through an interpolated string, let's walk through how to set one up. This time, however, we're going to filter for the youngest adventurers in the party.

#### Creating the Predicate Variable

As with every variable, it's important to give it a descriptive name. We'll call this one `youthPredicate`. The basic initializer for predicates is the `predicateWithFormat:` class method on `NSPredicate`. There are other, more complex initializers, but this is the only one we're going to discuss:

```objc
NSPredicate *youthPredicate = [NSPredicate predicateWithFormat:<#formatString#>];
```
#### Composing the Formatted String

The formatted string is how `NSPredicate` understands its instructions. There is a specific syntax just for `NSPredicate` that requires three components:

  * The key path,
  * The comparator, and
  * The object value

arranged in the manner of: `@"%K comparator %@", keyPath, objectValue`. Both the key path and object value *can* use format specifiers, but aren't required to. 

The **key path** contains instructions for accessing the value to compare. When accessing a dictionary like we have in the examples here, this is the name of the appropriate key.

**Note:** *The word* `self` *can be submitted to the keypath when filtering arrays or sets of basic objects such as* `NSString` *and* `NSNumber` *variables;* `self` *filters objects based upon their entirety rather than by a value-for-key or value-of-property basis.*

The **comparator** is the kind of comparison we want our filter to make. There is rather extensive list of valid comparators, a few of which we'll cover in this reading. In the example above, it is the mathematical comparator `>` ("greater than").

The **object value** is the desired search criteria we are making a comparison against. This can either be hardcoded into the format string before compiling, or it can be interpolated into the format string programmatically using a format specifier.

In the case of our `youthPredicate`, we want to find the adventurers who are younger than 100 years old; `age` will be our key path again, we'll use the `<` ("less than") comparator this time, and `100` will be the new object value. *Notice that we can write the digital value inside the format string without passing in an* `NSNumber` *object. The predicate syntax implicitly handles the conversion of primitives.* So, our format string should look like this: `@"age < 100"`.

Passing this format string into the initializer method will give us this first of our two lines to filter our data:

```objc
NSPredicate *youthPredicate = [NSPredicate predicateWithFormat:@"age < 100"];
```
#### Utilizing the Filter

The second line of code in our filtering operation (which does **not** need to *immediately* follow the predicate definition) submits our predicate to do some work for us. This is accomplished with the `filteredArrayUsingPredicate:` instance method on `NSArray`. We send this method call to the array whose contents we wish to filter, incorporate our predicate as the method argument, and receive a new `NSArray` object which contains the results.

**Advanced:** `NSSet` *and* `NSOrderedSet` *also have built-in predicate methods.*

To call this method on our `middleEarthers` array by submitting our new `youthPredicate` as the argument; let's capture its return in an array appropriately named `youth`:

```objc
NSArray *youth = [middleEarthers filteredArrayUsingPredicate:youthPredicate];
```
Great! Now let's print our results using a similar `for-in` loop to the one we used above:

```objc
for (NSDictionary *character in youth) {
    NSLog(@"%@ is only %@ years old.", character[@"name"], character[@"age"]);
}
```
This will print, in some order:

```
Bilbo is only 50 years old.
Fíli is only 82 years old.
Kíli is only 77 years old.
```
That's correct!

### Basic Comparators

The typical mathematical comparators are valid in the `NSPredicate` syntax, but they obey slightly different conventions than when writing directly within Objective-C. 

| Comparator | Meaning                  |
|:----------:|:-------------------------|
| `>`        | Greater than             |
| `<`        | Less than                |
| `>=`or`=>` | Greater than or equal to |
| `<=`or`=<` | Less than or equal to    |
| `=`or`==`  | Is equal to              |
| `!=`or`<>` | Is NOT equal to          |

**Advanced:** *Additionally,* `BETWEEN` *allows you to submit a range of two numbers in the form of* `keyPath BETWEEN { lower, upper }`, *such as* `age BETWEEN { 132, 178 }`.

Let's write a new predicate to filter for the tallest adventures. We'll need to submit `height` as the key path, and let's use the `>=` ("greater than or equal to") comparator to search for anyone at least 1.45 meters tall:

```objc
NSPredicate *tallPredicate = [NSPredicate predicateWithFormat:@"height >= 1.45"];
NSArray *tallCharacters = [middleEarthers filteredArrayUsingPredicate:tallPredicate];

for (NSDictionary *character in tallCharacters) {
    NSLog(@"%@ is %@ meters tall.", character[@"name"], character[@"height"]);
}
```
This will print, in some order:

```
Thorin is 1.49 meters tall.
Dwalin is 1.5 meters tall.
Bofur is 1.45 meters tall.
Óin is 1.45 meters tall.
Gandalf is 1.8 meters tall.
```
### String Comparators

There is a set of comparators specifically for comparing strings. The simplest ones to employ are:

| Comparator  | Meaning |
|:-----------:|:--------|
| `BEGINSWITH`| The key path's value must begin with the object value. |
| `ENDSWITH`  | The key path's value must end with the object value.
| `CONTAINS`  | The key path's value must contain the object value.
| `LIKE`      | The key path's value must match the object value, which can employ the use of wildcard characters `?` and `*`. |

**Note:** *Typing string comparators in either UPPERCASE or lowercase is equally valid predicate syntax, however, you'll most often see them written in UPPERCASE because it helps to visually distinguish the comparators from their associated key path and object value in the format string. We suggest typing them in UPPERCASE yourself.*

When predicating with string comparators, the object value (which would be a string) must either be wrapped inside single quotes (`'``'`) to distinguish its bounds from the bounds of the format string, or interpolated into the format string using a format specifier.

#### Using Wildcards in the `LIKE` Comparator

When using `LIKE`, the object value can be written with `?`s ("question marks") and `*`s ("stars"). These "wildcard" characters allow you to tell the predicate to permit either:

  * `?`: any **single** character in the held place, or 
  * `*`: any substring of **any length** (including zero).

Let's use the `LIKE` comparator with some wildcards to look for any adventurers who spell their names with an "li":

```objc
NSPredicate *liNamesPredicate = [NSPredicate predicateWithFormat:@"name LIKE '*li*'"];
NSArray *liNames = [middleEarthers filteredArrayUsingPredicate:liNamesPredicate];

for (NSDictionary *character in liNames) {
    NSLog(@"%@ spells his name with an 'li'.", character[@"name"]);
}
```
This will print, in some order:

```
Balin spells his name with an 'li'.
Dwalin spells his name with an 'li'.
Fíli spells his name with an 'li'.
Kíli spells his name with an 'li'.
```
Did you notice that using the `*` wildcard permitted Fíli's and Kíli's name to pass through even though the "li" in their names are the last two characters? If we were to change that second `*` to a `?`, the predicate would expect a single character after the "li" instead of "any substring or nothing":

```objc
NSPredicate *liNamesPredicate = [NSPredicate predicateWithFormat:@"name LIKE '*li?'"];
NSArray *liNames = [middleEarthers filteredArrayUsingPredicate:liNamesPredicate];

for (NSDictionary *character in liNames) {
    NSLog(@"%@ spells his name with an 'li'.", character[@"name"]);
}
```
This will print, in some order:

```
Balin spells his name with an 'li'.
Dwalin spells his name with an 'li'.
```
Conversely, if we instead replace the first `*` with `??`, the predicate will look for names with exactly two characters preceding the "li" and any substring following it:

```objc
NSPredicate *liNamesPredicate = [NSPredicate predicateWithFormat:@"name LIKE '??li*'"];
NSArray *liNames = [middleEarthers filteredArrayUsingPredicate:liNamesPredicate];

for (NSDictionary *character in liNames) {
    NSLog(@"%@ spells his name with an 'li'.", character[@"name"]);
}
```
This will print, in some order:

```
Balin spells his name with an 'li'.
Fíli spells his name with an 'li'.
Kíli spells his name with an 'li'.
```
Notice that Dwalin's name no longer appears since his name has three characters preceding the "li". Be careful when selecting which wildcard to use—you may not get the results you expect if you don't think through your wildcards.

**Advanced:** *The* `LIKE` *comparator is a light form of checking against a regular expression, or "RegEx"; the* `MATCHES` *comparator is used for full RegEx functionality.*

#### String Comparator Modifiers: `[c]`, `[d]`, and `[cd]`

*Namárië! Nai hiruvalyë Valimar.  
Nai elyë hiruva. Namárië!*  
—from *The Lord of the Rings* by J.R.R. Tolkien

These string comparators can also be modified to be insensitive to case (`[c]`), insensitive to diacritical characters (`[d]`), or insensitive to both (`[cd]`). These options, written inside square brackets `[``]`, can be appended to the selected string comparator within the predicate format string in the manner of `CONTAINS[cd]`. 

Case insensitivity treats uppercase letters equally to lowercase letters. Diacritical insensitivity treats letters with a diacritical mark as their base letter; [diacritical marks][diacritical_marks] are letters with special accents. As an example, the above excerpt from *Galadriel's Lament* in *The Lord of the Rings* written in the fictional elven language *Quenya* uses the diacritics `á` and `ë` in several places. Selecting the diacritic-insensitive option `[d]` will allow "Namárië" to pass as "Namarie".

##### Using String Comparator Modifiers

Let's use a string comparator to search among our adventurers from Middle Earth who spell their name with an "o". We should use the `CONTAINS` comparator on the `name` key path with the object value `o`:

```objc
NSPredicate *namesWithOPredicate = [NSPredicate predicateWithFormat:@"name CONTAINS 'o'"];
NSArray *namesWithO = [middleEarthers filteredArrayUsingPredicate:namesWithOPredicate];

for (NSDictionary *character in namesWithO) {
    NSLog(@"%@ spells his name with an 'o'.", character[@"name"]);
}
```
This will print, in some order:

```
Bilbo spells his name with an 'o'.
Thorin spells his name with an 'o'.
Bofur spells his name with an 'o'.
Bombur spells his name with an 'o'.
Dori spells his name with an 'o'.
```
Hm, well we got the names of five our adventurers, but we're missing Dori's brother Ori. Let's add the case insensitive option `[c]` to our `CONTAINS` comparator:

```objc
NSPredicate *namesWithOPredicate = [NSPredicate predicateWithFormat:@"name CONTAINS[c] 'o'"];
NSArray *namesWithO = [middleEarthers filteredArrayUsingPredicate:namesWithOPredicate];

for (NSDictionary *character in namesWithO) {
    NSLog(@"%@ spells his name with an 'o'.", character[@"name"]);
}
```
This will print, in some order:

```
Bilbo spells his name with an 'o'.
Thorin spells his name with an 'o'.
Bofur spells his name with an 'o'.
Bombur spells his name with an 'o'.
Dori spells his name with an 'o'.
Ori spells his name with an 'o'.
```
Ori's name was included in this new list! You may have noticed, however, that the brothers Óin and Glóin spell their names with a diacritical mark (as do Fíli and Kíli). That's why they're not coming up in the current search. Let's add the diacritical insensitive option `[d]` to our `CONTAINS` comparator:

```objc
NSPredicate *namesWithOPredicate = [NSPredicate predicateWithFormat:@"name CONTAINS[cd] 'o'"];
NSArray *namesWithO = [middleEarthers filteredArrayUsingPredicate:namesWithOPredicate];

for (NSDictionary *character in namesWithO) {
    NSLog(@"%@ spells his name with an 'o'.", character[@"name"]);
}
```
This will print, in some order:

```
Bilbo spells his name with an 'o'.
Thorin spells his name with an 'o'.
Bofur spells his name with an 'o'.
Bombur spells his name with an 'o'.
Glóin spells his name with an 'o'.
Óin spells his name with an 'o'.
Dori spells his name with an 'o'.
Ori spells his name with an 'o'.
```
Now all eight adventurers with an 'o' in their name correctly show up in the search!

### Basic Compound Predicates

Predicates can be combined or inverted within the format string by using the basic logical operators below that you're already familiar with:

| Operator   | Description                       |
|:----------:|:----------------------------------|
| `AND`,`&&` | Requires both predicates to pass  |
| `OR`,`||`  | Requires either predicate to pass |
| `NOT`,`!`  | Requires the predicate to fail    |

However, groups of predicates can only be combined using the *same* compound operator. *You cannot mix* `AND` *predicates with* `OR` *predicates.*

Let's use a basic compound predicate to search for adventurers in our party who are both large in stature (over 1.45 meters) and large in years (older than 170):

```objc
NSPredicate *tallAndWisePredicate = [NSPredicate predicateWithFormat:@"height > 1.45 AND age >= 170"];
NSArray *tallAndWise = [middleEarthers filteredArrayUsingPredicate:tallAndWisePredicate];

for (NSDictionary *character in tallAndWise) {
    NSLog(@"%@ is %@ meters tall and %@ years old.", character[@"name"], character[@"height"], character[@"age"]);
}
```
This will print, in some order:

```
Thorin is 1.49 meters tall and 195 years old.
Gandalf is 1.8 meters tall and 2019 years old.
```
Thorin and Gandalf! The two leaders of the adventuring party who are among both the tallest and the oldest members of the group.

## The Road Goes Ever On And On

*It's a dangerous business, Frodo, going out your door. You step onto the road, and if you don't keep your feet, there's no knowing where you might be swept off to.*  
—from *The Lord of the Rings* by J.R.R. Tolkien

There is a wealth of deep functionality provided by `NSPredicate`. The options discussed in this reading barely scratch the surface of a truly wonderful and powerful utility class. When you reference Apple's [Predicate Programming Guide][predicate_guide] or look for help and examples online, there will be a lot of use cases and syntax that you won't understand. This is normal. Programmers can spend years continuing to learn advanced features and uses of `NSPredicate` alone, but yet the very basics in this reading provide us with an amazing toolset. 

![](https://curriculum-content.s3.amazonaws.com/ios/reading-ios-filtering-basic/adventure.gif)


Matt Thompson wrote it beautifully on his blog [NSHipster](http://nshipster.com/nspredicate/):

>`NSPredicate` is, and I know this is said a lot, truly one of the jewels of Cocoa. Other languages would be lucky to have something with half of its capabilities in a third-party library—let alone the standard library. Having it as a standard-issue component affords us as application and framework developers an incredible amount of leverage in working with data.
>
>Together with `NSExpression`, `NSPredicate` reminds us what a treat Foundation is: a framework that is not only incredibly useful, but meticulously architected and engineered, to be taken as inspiration for how we should write our own code.

Keep your feet out there!

[heights]: http://24.media.tumblr.com/fe736cede58224929734a827ba260f76/tumblr_mhos2lSLs51rmwdaro1_500h.jpg
[ages]: http://lotrproject.com/blog/2014/06/07/character-age-at-the-time-of-the-hobbit/
[Gandalf_age]: http://scifi.stackexchange.com/questions/58952/how-old-is-gandalf

[predicate_logic]: https://en.wikipedia.org/wiki/Predicate_logic
[predicate_guide]: https://developer.apple.com/library/ios/documentation/Cocoa/Conceptual/Predicates/Articles/pSyntax.html
[diacritical_marks]: https://en.wikipedia.org/wiki/English_terms_with_diacritical_marks
