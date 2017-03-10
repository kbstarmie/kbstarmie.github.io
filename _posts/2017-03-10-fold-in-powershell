---
layout: post
title: Fold in Powershell
---

# {{ page.title }}

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
```
