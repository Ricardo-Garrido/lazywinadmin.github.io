---
layout: single
title: PowerShell/Pester - Make sure your Comment Based Help is not indented
excerpt: 
permalink: /2016/08/powershellpester-make-sure-your-comment.html
tags: 
- adsips
- pester
- powershell
published: true
comments: true
---
{% include base_path %} 
{% include toc title="Table of content" icon="file-text" %}
<a href="{{ base_path }}/images/2016/20160822_PowerShellPester_-_Make_sure_your_Comment_Based_Help_is_not_indented/pester_logo__1425385033__-400x400.png" imageanchor="1" style="clear: left; float: left; margin-bottom: 1em; margin-right: 1em;"><img border="0" height="120" src="{{ base_path }}/images/2016/20160822_PowerShellPester_-_Make_sure_your_Comment_Based_Help_is_not_indented/pester_logo__2058964902__-200x200.png" width="120" /></a>
While writing some PowerShell Pester tests for my module <a href="https://www.powershellgallery.com/packages/AdsiPS/1.0.0.2" target="_blank">AdsiPS</a>, I wanted to make sure for each functions that all the help keywords of the comment based help were not indented.

Depending of the script editor you are using, the number of spaces or tabs might differ and once published on GitHub the code might looks not so pretty.


My goal is just to keep my code clean and enforce the same practice across all the functions.


## Example
Here for example, <b>I don't want this</b>:

<img border="0" src="{{ base_path }}/images/2016/20160822_PowerShellPester_-_Make_sure_your_Comment_Based_Help_is_not_indented/PowerShellPester-Indented_commentbasedhelp01__2057622999__-511x384.png" />

Instead, <b>I want all the comment based help in that format</b>

<img border="0" src="{{ base_path }}/images/2016/20160822_PowerShellPester_-_Make_sure_your_Comment_Based_Help_is_not_indented/PowerShellPester-Indented_commentbasedhelp02__99513925__-475x378.png" />

(Help keywords directly at the beginning of the line)

## Code
This can be accomplish by something like that:


```powershell
[CmdletBinding()]
PARAM (
    $ModuleName = "ADSIPS",
    $GithubRepository = "github.com/lazywinadmin/"
)

# Make sure one or multiple versions of the module are note loaded
Get-Module -Name $ModuleName | remove-module

# Find the Manifest file
$ManifestFile = "$(Split-path (Split-Path -Parent -Path $MyInvocation.MyCommand.Definition))\$ModuleName\$ModuleName.psd1"

# Import the module and store the information about the module
$ModuleInformation = Import-module -Name $ManifestFile -PassThru

# Get the functions present in the Manifest
$ExportedFunctions = $ModuleInformation.ExportedFunctions.Values.name

# Testing the Module
Describe "$ModuleName Module - HELP" -Tags "Module" {
    #$Commands = (get-command -Module ADSIPS).Name
    
    FOREACH ($funct in $ExportedFunctions)
    {
        # Retrieve the content of the current function
        $FunctionContent = Get-Content function:$funct
        
        Context "$funct - Comment Based Help - Indentation Checks"{
            
            # Validate Help start at the beginning of the line
            It "Help - Starts at the beginning of the line"{
                <span style="background-color: yellow;">$Pattern = ".Synopsis"
                <span style="background-color: yellow;">(<span style="background-color: yellow;">$FunctionContent -split '\r\n' |
                <span style="background-color: yellow;">    <span style="background-color: yellow;">select-string $Pattern).line -match "^$Pattern" | Should Be $true
            }
        }
    }
}

```

## Step By Step
<b>So what is happening here ?</b>

<b>#1</b>- I get the content of the function using ```Get-Content```

```powershell
$FunctionContent = Get-Content function:$funct
```

<b>#2</b>- I define the pattern that I want to retrieve, here I'm just looking for the help keyword Synopsis

```powershell
$Pattern = ".Synopsis"
```
I guess i could check each help keywords as an improvement of the current code.

<b>#3</b>- I split the content of the Function file on each Carriage Return character and look for the pattern I define in #2. This will give me any lines that contains '```.Synopsis```' (We should only have one)

```powershell
$FunctionContent -split '\r\n' | select-string $Pattern
```

<b>#4</b>- Using regex, we specify the caret character '```^```' which will matches the beginning of a line with the pattern we defined. Finally we validate that it is $true using the Pester syntax '```Should Be```'

```powershell
line -match "^$Pattern" | Should Be $true
```

<img border="0" src="{{ base_path }}/images/2016/20160822_PowerShellPester_-_Make_sure_your_Comment_Based_Help_is_not_indented/PowerShellPester-Indented_commentbasedhelp03__40354873__-834x329.png"/>


## Related posts

* <a href="{{ base_path }}/2016/05/using-pester-to-test-your-comment-based.html" target="_blank">Using Pester to test your Comment Based Help</a>

* <a href="{{ base_path }}/2016/05/using-pester-to-test-your-manifest-file.html" target="_blank">Using Pester to test your Manifest File</a>

* <a href="{{ base_path }}/2016/08/powershellpester-make-sure-your.html" target="_blank">Make sure your parameters are separated by an empty line</a>
