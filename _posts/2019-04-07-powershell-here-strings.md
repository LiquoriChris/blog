---
layout: post
section-type: post
title: Using Here-Strings in PowerShell
category: powershell
tags: [ 'powershell' ]
---

Running commands in PowerShell that require a format that will not run natively in PowerShell could be a difficult task, or can it? PowerShell provides a way to store, for example, a xml or JSON payload as a string, enter here-string. A here-string is a single or double quoted string in which the quotation marks are interepted literally. A simple example is invoking a Rest API that requires a POST with a JSON body.

```
{
    "apple": [
        "red",
        "green"
    ],
    "grape": [
        "green",
        "red"
    ],
    "blueberry": "blue"
}
```

The above example is a simple collection that has the fruit as the key, and what color the fruit can be. If we tried to store this in a variable in PowerShell, will we receive an error that states: 

```
At line:2 char:12
+     "apple": [
+            ~
Unexpected token ':' in expression or statement.
At line:5 char:6
+     ],
+      ~
Missing argument in parameter list.
At line:9 char:6
+     ],
+      ~
Missing argument in parameter list.
    + CategoryInfo          : ParserError: (:) [], ParentContainsErrorRecordException
    + FullyQualifiedErrorId : UnexpectedToken
```

This happens because PowerShell is trying to execute the command it was given, which in this case is JSON format. There are two ways to get around using JSON format in PowerShell, converting the JSON to a PowerShell custom object and using a here-string. Let's take a look at both and see how they work. First, let's convert the JSON into a PowerShell custom object, PowerShell has a Cmdlet to convert JSON format to PowerShell, `ConvertFrom-Json`. 

```
$Json = Get-Content -Path C:\temp\fruit.json
$Json |ConvertFrom-Json
```

As you can see, you can take a JSON file and easily convert it to a custom object, however, if you are using a non-standard PowerShell format, just store it in a here-string variable to be used later on. For example, if we needed to update a field using a Rest API, storing the JSON payload in a here-string is much easier than converting the payload to JSON and using it in the body. Let's look at both examples:

````
$Object = @{
    apple = @(
        'red',
        'green'
    )
    grape = @(
        'green',
        'red'
    )
    blueberry = 'blue'
}

$Body = $Object |ConvertTo-Json
```

That should work, not throw any errors when using it in the body, but what if we could use the JSON directly, not having to convert it:

```
$Body = @'
{
    "apple": [
       "red",
       "green"
    ],
    "grape": [
        "green",
        "red"
    ],
    "blueberry": "blue"
}
'@
```

Notice how the here-string starts with "@'", telling PowerShell you are storing this entire string in a here-string. You can use single or double quoted here-strings, however, if you are using variables inside of the here-string, you must use double quotes. Let's see an example where we can use variables inside of the here-string: 

```
$Red = 'red'
$Green = 'green'
$Blue = 'blue'
$Body = @"
{
    "apple": [
       $Red,
       $Green
    ],
    "grape": [
        $Green,
        $Red
    ],
    "blueberry": $Blue
}
"@
```

Output:

```
{
    "apple": [
       red,
       green
    ],
    "grape": [
        green,
        red
    ],
    "blueberry": blue
}
```

Using here-strings make it easy to store complicated formats such as xml or JSON without having to convert into a readable PowerShell format. This will help when a vendor supplies a JSON payload to be used in a Rest API, all that needs to be done is substitute your values in a here-string and invoke the Rest API. As always, practice makes perfect, try running examples in the console before running in a production environment. Here-strings will save you some lines of code and time when building your PowerShell scripts.

pwshliquori