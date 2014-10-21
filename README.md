# Basic Filtering

## What Problem Are We Solving?

As with most data, when we present it to the user, we will want only the relevant data to be shown. If you have used any sort of restaurant review site in the past ten years, you'll have had this experience. It would be inefficient to have to look through every restaurant in the world all at once. Instead, we provide the software with our preferences for location, price, cuisine, and more. In return, the software provides us with a cultivated list of restaurants we might want to know more about. 

There are many ways to filter data in Cocoa. In this reading, we will teach you the most common and simplest way to filter data stored in an `NSArray` object.

## What is NSPredicate?

The most common filtering mechanism in Cocoa with Objective-C is `NSPredicate`. With it we can do quite a bit and with ease. Here is the syntax for a basic `NSPredicate`.

###### Example

```objc

NSArray *testArray = @[@"John",@"Mary",@"Margaret",@"Joshua",@"Biff",@"Ezekiel" ];
    
NSPredicate *allJNames = [NSPredicate predicateWithFormat:@"self BEGINSWITH %@",@"J"];
    
NSArray *filteredTestArray = [testArray filteredArrayUsingPredicate:allJNames];
    
NSLog(@"%@",filteredTestArray);

```

This will result in the following output:

```
2014-10-20 15:40:40.498 filteringTests[36619:1980980] (
    John,
    Joshua
)

```

`NSPredicate` is one of the frameworks in Cocoa that has its own sort of 'sub language' of format strings to learn. We'll get into the specifics of this later, but for now, the most important things to take away from the above example are:

1) An `NSPredicate` requires a kind of string that defines how the data will be filtered as its argument.

2) An `NSPredicate` is applied to a collection (i.e. `NSArray`, `NSSet`).

######Example

```objc

    NSDictionary *johnDict = @{@"name":@"John",@"age":@18};
    NSDictionary *maryDict = @{@"name":@"Mary",@"age":@39};
    NSDictionary *margDict = @{@"name":@"Margaret",@"age":@24};
    NSDictionary *joshuaDict = @{@"name":@"Joshua",@"age":@21};
    NSDictionary *biffDict = @{@"name":@"Biff",@"age":@21};
    NSDictionary *ezekDict = @{@"name":@"Ezekiel",@"age":@400};
    
    NSArray *ourCharacters = @[johnDict, maryDict, margDict, joshuaDict, biffDict, ezekDict];
    
    NSPredicate *allJNames = [NSPredicate predicateWithFormat:@"self.name BEGINSWITH %@",@"J"];

    NSArray *filteredTestArrayWithDictionaries = [ourCharacters filteredArrayUsingPredicate:allJNames];
    
    NSLog(@"%@",filteredTestArrayWithDictionaries);

```
Here we have filtered on the same information, but filtering on an `NSArray` of `NSDictionary` objects.

The output of this code will result in the following output in our debug console:

```
2014-10-20 15:55:42.752 filteringTests[36700:1987740] (
        {
        age = 18;
        name = John;
    },
        {
        age = 21;
        name = Joshua;
    }
)
```

You'll notice that in the above code snippet, the only difference besides the data in the `NSArray` itself is the language used in the predicate. Given we are attempting to filter our array on an `NSDictionary` key, we specify the key with dot notation (i.e. `self.name`).

These are the basics of creating an `NSPredicate`. But in order to fully take advantage of them, we must be more familiar with the language of `NSPredicate`: the format string.

###Format Strings


####Variables

Our format strings traditionally begin with the variable we want to filter our data, and generally end with the value / argument by which we wish to filter. In our examples thus far, that has been names of people and the letter "J". When the input is a simple `NSArray` of names, we can just use `self` to refer to the data as we did in our first example. However, when filtering an `NSDictionary` we can get more dynamic and specify an argument instead of `self.name` as we did above. For example:

```objc
    NSString *fieldToFilterOn = @"name";
    
    NSPredicate *someAlphabeticFieldToFilterOn = [NSPredicate predicateWithFormat:@"self.%K BEGINSWITH %@",fieldToFilterOn,@"J"];

    NSArray *filteredTestArrayWithDictionaries = [ourCharacters filteredArrayUsingPredicate:someAlphabeticFieldToFilterOn];
    
    NSLog(@"%@",filteredTestArrayWithDictionaries);
```

With our same dataset, this would evaluate to just our `johnDict` and `joshDict` again.

#### Basic Comparison

You can compare the left and right arguments in an NSPredicate with standard comparators: ==, >= (or =>), <= (or =<), <, >, != (or <>).

In addition, BETWEEN is also worth noting, as it allows you to compare one value to an array of values. For instance:

```objc
NSPredicate *betweenPredicate =
    [NSPredicate predicateWithFormat: @"@123 BETWEEN %@", @[@100, @150]];
```

#### String Comparison

##### BEGINSWITH / ENDSWITH

###### Example 

Well, every example we have given so far has used this comparator. So we won't waste your time!


##### CONTAINS

Ensures the left hand expression contains the right hand expression.

###### Example:
```objc

    NSDictionary *johnDict = @{@"name":@"John",@"age":@18};
    NSDictionary *maryDict = @{@"name":@"Mary",@"age":@39};
    NSDictionary *margDict = @{@"name":@"Margaret",@"age":@24};
    NSDictionary *joshuaDict = @{@"name":@"Joshua",@"age":@21};
    NSDictionary *biffDict = @{@"name":@"Biff",@"age":@21};
    NSDictionary *ezekDict = @{@"name":@"Ezekiel",@"age":@400};
    
    NSArray *ourCharacters = @[johnDict, maryDict, margDict, joshuaDict, biffDict, ezekDict];
    
    NSPredicate *allJNames = [NSPredicate predicateWithFormat:@"self.name CONTAINS %@",@"a"];

    NSArray *filteredTestArrayWithDictionaries = [ourCharacters filteredArrayUsingPredicate:allJNames];
    
    NSLog(@"%@",filteredTestArrayWithDictionaries);

```

The output of this code will result in the following output in our debug console, as all of the below names contain the letter "a".

```
2014-10-20 16:49:29.111 filteringTests[36700:1987740] (
        {
        age = 39;
        name = Mary;
    },
        {
        age = 24;
        name = Margaret;
    },
       {
        age = 21;
        name = Joshua;
    }
)
```

##### LIKE

Ensures the left hand expression is "like" the right hand expression. 

Okay, so that is not particularly self-explanatory is it? The `LIKE` keyword opens up a can of worms because it allows us to take advantage of regular expression matchers `*` and `?`. Here is an example, but if you don't understand this just yet, come back to it after you read up on regular expressions.



######Example

```objc

NSDictionary *johnDict = @{@"name":@"John",@"age":@18};
NSDictionary *maryDict = @{@"name":@"Mary",@"age":@39};
NSDictionary *margDict = @{@"name":@"Margaret",@"age":@24};
NSDictionary *fargDict = @{@"name":@"Fargaret",@"age":@31};
NSDictionary *flargDict = @{@"name":@"Flargaret",@"age":@35};
NSDictionary *joshuaDict = @{@"name":@"Joshua",@"age":@21};
NSDictionary *biffDict = @{@"name":@"Biff",@"age":@21};
NSDictionary *ezekDict = @{@"name":@"Ezekiel",@"age":@400};
    
NSArray *ourCharacters = @[johnDict, maryDict, margDict, fargDict, flargDict, joshuaDict, biffDict, ezekDict];
    
NSPredicate *allNamesThatStartWithALetterAndThenTheLettersARAndThenAnythingElse = [NSPredicate predicateWithFormat:@"self.name LIKE %@",@"?ar*"];

NSArray *filteredTestArrayWithDictionaries = [ourCharacters filteredArrayUsingPredicate:allNamesThatStartWithALetterAndThenTheLettersARAndThenAnythingElse];
    
NSLog(@"%@",filteredTestArrayWithDictionaries);

```

The output of this code will result in the following output in our debug console, as all of the below names contain the letters "ar", may have letters after them, and have one character in front of them. Here is the result:

```
2014-10-20 16:49:29.111 filteringTests[36700:1987740] (
        {
        age = 39;
        name = Mary;
    },
        {
        age = 24;
        name = Margaret;
    },
       {
        age = 21;
        name = Joshua;
    }
)
```

##### Modified String Operators

When using string comparators, you should expect your results to be case / diacritical mark (aka accent marks) sensitive. You can make your results case and diacritical mark insensitive by adding [c] and/or [d] after your comparator. 

######Example

```objc
    NSDictionary *johnDict = @{@"name":@"John",@"age":@18};
    NSDictionary *maryDict = @{@"name":@"Mary",@"age":@39};
    NSDictionary *margDict = @{@"name":@"Margaret",@"age":@24};
    NSDictionary *fargDict = @{@"name":@"Fargaret",@"age":@31};
    NSDictionary *flargDict = @{@"name":@"flargaret",@"age":@35};
    NSDictionary *joshuaDict = @{@"name":@"Joshua",@"age":@21};
    NSDictionary *biffDict = @{@"name":@"Biff",@"age":@21};
    NSDictionary *ezekDict = @{@"name":@"Ezekiel",@"age":@400};
    
    NSArray *ourCharacters = @[johnDict, maryDict, margDict, fargDict, flargDict, joshuaDict, biffDict, ezekDict];
    
    NSPredicate *allNamesBeginningWithCaseInsensitiveF = [NSPredicate predicateWithFormat:@"self.name BEGINSWITH[c] %@",@"F"];

    NSArray *filteredTestArrayWithDictionaries = [ourCharacters filteredArrayUsingPredicate:allNamesBeginningWithCaseInsensitiveF];
    
    NSLog(@"%@",filteredTestArrayWithDictionaries);
```

In the above our NSPredicate would log all names that start with capital "F" or lowercase "f".

Here are the standard string matchers you will see / use regularly.

[c] - case insensitive matching

[d] - diacritical mark insensitive matching (for letters with accent marks on them)

Note: String matchers may be used together, like so:

[cd] - would match on strings that are both case and diacritical mark insensitive

###### Compounds Predicates

Predicates may be put together using the keywords `AND` (or `&&`), `OR` (or '||'), and `NOT` (or '!').


######Example
```objc
    NSDictionary *johnDict = @{@"name":@"John",@"age":@18};
    NSDictionary *maryDict = @{@"name":@"Mary",@"age":@39};
    NSDictionary *margDict = @{@"name":@"Margaret",@"age":@24};
    NSDictionary *fargDict = @{@"name":@"Fargaret",@"age":@31};
    NSDictionary *flargDict = @{@"name":@"flargaret",@"age":@35};
    NSDictionary *joshuaDict = @{@"name":@"Joshua",@"age":@21};
    NSDictionary *biffDict = @{@"name":@"Biff",@"age":@21};
    NSDictionary *ezekDict = @{@"name":@"Ezekiel",@"age":@400};
    
    NSArray *ourCharacters = @[johnDict, maryDict, margDict, fargDict, flargDict, joshuaDict, biffDict, ezekDict];
    
    NSPredicate *allNamesBeginningWithCaseInsensitiveF = [NSPredicate predicateWithFormat:@"self.name BEGINSWITH[c] %@ OR self.name BEGINSWITH %@",@"F",@"J"];

    NSArray *filteredTestArrayWithDictionaries = [ourCharacters filteredArrayUsingPredicate:allNamesBeginningWithCaseInsensitiveF];
    
    NSLog(@"%@",filteredTestArrayWithDictionaries);
```

This will log all names that start with uppercase "F", lowercase "F", and uppercase "J".

With this, you have all the basics of `NSPredicate`-based filtering at your fingertips. There are some more advanced moves you can make, but get these down first and then read up on Advanced Filtering. Filtering will not only help you filter standard `NSArray` and `NSDictionary` objects, but also data stored in Core Data, so remember to come back for a refresher when you get there!