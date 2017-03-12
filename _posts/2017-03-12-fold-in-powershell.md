---
layout: post
title: Fold in PowerShell
---

Oftentimes, when working with data, you might find yourself needing to process or transform it in some way to continue. Duh, no kidding, you might say. Are you really writing a blog post about this? Run with me here, I promise there's gold at the end of the rainbow.

Anyway. You might say "Sure, I'll use Excel and apply a function to my dataset". To which I'll reply "Good luck applying a regular expression to a range", or, "Good luck reading a nested conditional", and walk away.

My tool of choice for doing any sort of data munging is PowerShell. The above problem is as simple as follows:

```powershell
$list | % { $_ -match "(.*)" ; $matches[1] }
``` 

Naturally, you can make that script block do whatever task you want, including be readable. PowerShell 1, Excel 0.

By the way, that little *%* symbol is actually an alias for *Foreach-Object*, a cute function/cmdlet that applies your block to every item in your list. Very neat!

Some of you might point out "hey, that's a map!". And you would be completely spot on. Although it doesn't admit it, PowerShell actually has a lot of functional structures that you can take advantage of. Map and filter are practically first-class citizens, with dedicated functions and aliases to perform these operations over lists.

One of the key advantages of functional programming, I find, is the provision of many standard utility functions, with well-understood behaviours, to save you on reimplementing these basic tasks over and over. That and partial application. And a usable type system. Sadly, we don't get those last two with PowerShell, but we're in a command shell, not writing F# or Haskell.

So instead of writing this code twenty times a day...

```powershell
$results = @()
foreach ($item in $list) {
  if ($item.property -eq $desiredValue) {
    $results += $item
  }
}
```

You can write this!

```powershell
$list | ? { $_.property -eq $desiredValue }
```

Or this!

```powershell
$list | ? -Property property -eq -Value $desiredValue
```

The *?* symbol is the alias for *Where-Object*, which is clearly the filter function.

However, one very notable omission is the fold. 

Where map is used for applying some function to every item in a list, and filter is used to select every item in a list according to some function, fold basically collapses every item in a list into a single value, according to some function.

Having found it useful in my day to day activities, I have these neat functions lying around.

```powershell
function foldl {
    param(
        $baseValue, $f, $list
    )

    if ($list.length -gt 1) {
        foldl $(& $f $baseValue $list[0]) $f $list[1..$($list.length)]
    } else {
        & $f $baseValue $list[0]
    }
}

function foldr {
    param($baseValue, $f, $list)

    if ($list.length -gt 0) {
        & $f $list[0] (foldr -f $f -baseValue $baseValue -list $list[1..($list.length)])
    } else {
        $baseValue
    }
}

```

"What the heck did I just read?!", you may ask. Let's break it down a bit.

*fold* starts off with a base value, and applys its function to every item in the list, "adding" the result to its base value and continuing. The fold-left version works slightly differently to the fold-right version - left will apply the function to the left-most values first, and the right version will apply it from the right. The difference is completely non-existent in PowerShell, so best not to worry about it.

So what do you even use this for? Here's a simple example that sums a list.

```powershell
PS /home/kbstarmie> foldl -baseValue 0 -f {param($1,$2) $1 + $2} -list @(1..100)
5050
PS /home/kbstarmie> foldr -baseValue 0 -f {param($1,$2) $1 + $2} -list @(1..100)
5050

```

You're basically limited by your imagination (and requirements) in folding things. You can use it to check if the property of every object is true, like so.

```powershell
foldl -baseValue $true -f {param($1,$2) $1.property -and $2.property} -list $someObjectList
```

Obviously, you will need to ensure your objects have the right properties so the system doesn't complain, but it's a small price to pay for the convenience of a useful PowerShell helper function!

I appreciate that some of this content might go right over some readers who haven't been acquainted with functional programming fundamentals. In that case, I wholeheartedly recommend spending an afternoon or evening with [Learn You A Haskell For Great Good!](http://learnyouahaskell.com/) to be introduced to the basics. It's fun, trust me.

Anyway, I just wanted to introduce this small function and I hope that you all find it useful in your day to day data munging, scripting, or whatever! I've made a script [available in my PowerShell repository](https://github.com/kbstarmie/powershell-snippets/blob/master/functions/fold.ps1) if you're rushing to apply it to your profile loaded code.

Until next time *~Kira*

