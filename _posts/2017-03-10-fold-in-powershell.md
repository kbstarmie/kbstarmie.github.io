---
layout: post
title: Fold in PowerShell
---

# {{ page.title }}

As far as programming languages go, functional languages 

Nevertheless, the concept of stateless processing is useful when structuring 

PowerShell actually has a lot of functional structures that you can take advantage of. Map and filter are practically first-class citizens, with dedicated syntax and aliases to perform these operations over lists.

However, one very notable omission is the fold.

Having found it useful in my day to day activities, whether, I have this neat function sitting around.

```
function foldl {
    param(
        $baseValue, $f, $list
    )

    if ($list.length -gt 1) {
        foldl $(& $f $baseValue $list[0]) $f $list[1..$($list.count)]
    } else {
        & $f $baseValue $list
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
