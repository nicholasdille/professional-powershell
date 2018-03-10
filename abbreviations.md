<!-- .slide: id="abbreviations" -->

## Do not abbreviate

Abbreviations make code harder to read

--

<!-- .slide: id="aliases" -->

## Do not abbreviate

Do not use aliases:

```powershell
Get-ChildItem      # not gci
```

--

<!-- .slide: id="parameters" -->

## Do not abbreviate

Do not use abbreviated parameters:

```powershell
Get-ChildItem -Recurse      # not -rec
```

--

<!-- .slide: id="language" -->

## Natural language

You can design [cmdlets](functions) to sound similar to natural language

```powershell
Move-Item -From $a -To $b

# or
Move-Item -FromPath $a -ToPath $b

# or
Move-Item -SourcePath $a -DestinationPath $b
```