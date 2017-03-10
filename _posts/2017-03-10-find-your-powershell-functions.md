---
title: Finding your PowerShell functions with AST parsing
layout: post
---

I don't know how wise it is to admit this on a public blog, but personal file organisation is not one of my strong points. The pros and (mostly) cons of hierarchical file systems aside, maintaining files and folders, with their rigid categories and topics, just isn't what I focus on. As a consequence, my user directory looks like the temp folder, and the file names are almost as descriptive. 😵

In my day to day work, I might find a useful snippet and save it in one of five or six places, with a name that may or may not be helpful. Or I might add a new function to one of my scripts. Which is fine, I'll be able to find it later.`*

Most of the time it's fine. But sometimes, I'll just struggle to find it, and we all know how reliable the operating system search is. So what to do?

Well, one day, late in the evening, I wondered if you could monkey patch PowerShell, and how you might do it. I'm a little bit of a nerd, I guess. Anyway, and I did some research, and there's an entire .NET class devoted to the PowerShell  


```
$allPowershellFunctions = @($files)| % {
    $_ | % { 
        $parser = [System.Management.Automation.Language.Parser]::ParseFile("$_", [ref]$null, [ref]$null)
        $functions = $parser.findall({ $args[0] -is [System.Management.Automation.Language.FunctionDefinitionAst]}, $true)

        [pscustomobject]@{
            Path = $_ ;
            Functions = $functions | % {
                [pscustomobject]@{
                    Name = $_.name ;
                    Block = $_.extent.text ; 
                }
            }
        }
    }
}

$allPowershellFunctions | % {
    write-output ""
    write-output "# $_.path" 
    write-output ""
    $_.functions | % {
        write-output $_.block
    }

}
```

`* This scenario may or may not happen, but I thought it was an entertaining angle to present this code, which, admittedly, was initially written to index all the PowerShell code available to me. That index is still on the wish list. 🙏
