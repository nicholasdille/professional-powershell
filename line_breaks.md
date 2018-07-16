<!-- .slide: id="line_breaks" -->

## Break long lines

Do not use the backtick (`) to break a line, use one of the other [line continuation characters](https://get-powershellblog.blogspot.co.uk/2017/07/bye-bye-backtick-natural-line.html) instead

--

## Break long lines

Break at operators and indent:

```powershell
if ($InputObject -eq 'ThisValue' -or
    $InputObject -eq 'OtherValue') {
    Do-Something
}
```

--

## Break long lines

Break at pipeline steps and indent:

```powershell
Get-ChildItem -Path c:\ -Recurse |
    Select-Object -First 10
```

--

<!-- .slide: id="splatting" -->

## Break long lines

Use [splatting](https://docs.microsoft.com/en-us/powershell/module/microsoft.powershell.core/about/about_splatting?view=powershell-6#splatting-with-hash-tables) to break long statements:

```powershell
$Params = @{
    Path = 'c:\'
    Recurse = $true
}
Get-ChildItem @Params
```

Splatting can be used to build parameters dynamically
