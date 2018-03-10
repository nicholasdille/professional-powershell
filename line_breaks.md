<!-- .slide: id="line_breaks" -->

## Break long lines

Break lines at 70 characters

Configure your editor to replace a tabstop with four spaces

Use the backtick (`) to break a line

--

## Break long lines

Break at parameters and indent:

```powershell
Get-ChildItem `
    -Path c:\ `
    -Recurse
```

--

## Break long lines

Break at pipeline steps and indent:

```powershell
Get-ChildItem -Path c:\ -Recurse `
    | Select-Object -First 10
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