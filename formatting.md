<!-- .slide: id="formatting" -->

## Formatting

Proper formatting increases readability and enables code [reuse](#/scope)

--

<!-- .slide: id="case" -->

## Formatting

Use CamelCase for everything

```powershell
$MyVariable = ''
function Get-MyData {}
```

--

<!-- .slide: id="brackets" -->

## Formatting

Place curly brackets on a new line:

```powershell
if ($true)
{
    'true'
}
```

Except when a scriptblock is used as a parameter value:

```powershell
Invoke-Command -Scriptblock { whoami }
```

--

<!-- .slide: id="properties" -->

## Accessing properties

Hard to read:

```powershell
$Path = (Get-ChildItem | Select -First 1).FullName
```

Better:

```powershell
$Path = Get-ChildItem | Select -First 1 -ExpandProperty FullName
```

Best because you continue with an object:

```powershell
$Path = $Get-ChildItem | Select -First 1
$Path.FullName
```

The following line are equivalent:

```powershell
Get-ChildItem | Select -ExpandProperty FullName
Get-ChildItem | ForEach-Object { $_.FullName }
```

--

<!-- .slide: id="verbs" -->

## Approved verbs

Microsoft provides a list of approved verb: `Get-Verb`

`New` create a new object, e.g. `New-Item`

`Add` adds something to an existing object, e.g. `Add-Content`

`Import`/`Export` operate on files or external systems, e.g. `Import-CliXml`

`ConvertTo`/`ConvertFrom` operate in memory, e.g. `ConvertFrom-Json`