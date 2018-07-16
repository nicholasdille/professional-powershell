<!-- .slide: id="hints" -->

## Hints

Useful topics

--

<!-- .slide: id="errors" -->

## Hints: Non-terminating errors

Some errors do not terminate execution

Force termination `-ErrorAction Stop`

`Write-Error` does not terminate

`throw` terminates

--

<!-- .slide: id="progress" -->

## Hints: Progress

PowerShell has an integrated progress bar controlled by `Write-Progress`:

- Text-based on the console
- Graphical in PowerShell ISE

Note that you will have to calculate the percentage yourself

Progress bars can be stacked using `-Id` and `-ParentId`

See also progress for [jobs](#/parallelization)

--

<!-- .slide: id="types" -->

## Hints: Custom types (1)

Any hashtable converted to an object:

```powershell
PS> @{ a = 1; b = 2 }

Name Value
---- -----
a    1
b    2

PS> [pscustomobject]@{ a = 1; b = 2 }

a b
- -
1 2
```

Use hashtables for random access

Use `pscustomobject` for arrays of objects

--

## Hints: Custom types (2)

Use `pscustomobject` to create a named custom type

```powershell
PS> [pscustomobject]@{
    Name = 'test'
    PSTypeName = 'MyType'
} | Get-Member

    TypeName: MyType

Name        MemberType   Definition
----        ----------   ----------
Equals      Method       bool Equals(System.Object obj)
GetHashCode Method       int GetHashCode()
GetType     Method       type GetType()
ToString    Method       string ToString()
Name        NoteProperty string Name=test
```

--

## Hints: Custom types (3)

Custom types are extended by `Add-Member`

```powershell
PS> $Item = [pscustomobject]@{ Name = 'test' }
PS> Add-Member -InputObject $Item -MemberType NoteProperty -Name 'Id' -Value 3
PS> $Item

Name Id
---- --
test  3
```

[All about custom objects by Kevin Marquette](https://kevinmarquette.github.io/2016-10-28-powershell-everything-you-wanted-to-know-about-pscustomobject/)

--

<!-- .slide: id="custom_formats" -->

## Hints: Custom formats (1)

Define how to display an object:

- Whether to use list or table view
- Which properties to display

Can be used with [custom types](#/types)

Store XML data in `MyTypeName.Format.ps1xml`

Import as [module](#/sharing) member or `Update-FormatData`

Override by piping to `Format-List *`

--

## Hints: Custom formats (2)

```powershell
<?xml version="1.0" encoding="utf-8" ?>
<Configuration><ViewDefinitions><View>
<Name>DefaultView</Name>
<ViewSelectedBy><TypeName>MyTypeName</TypeName></ViewSelectedBy>
<TableControl><AutoSize/>
<TableHeaders><TableColumnHeader>
<Width>10</Width>
<Alignment>Right</Alignment>
</TableColumnHeader></TableHeaders>
<TableRowEntries><TableRowEntry>
<TableColumnItems><TableColumnItem><PropertyName>Id</PropertyName></TableColumnItem></TableColumnItems>
</TableRowEntry></TableRowEntries>
</TableControl>
</View></ViewDefinitions></Configuration>
```

See also the [official documentation](https://docs.microsoft.com/en-us/powershell/module/microsoft.powershell.core/about/about_format.ps1xml?view=powershell-6)

Display loaded formats using `Get-FormatData`

--

<!-- .slide: id="quoting" -->

## Hints: Quoting

Single quotes do not evaluate variables

Double quotes evaluate variables

Here strings allow for multi-line string:

```powershell
$Data = @"
Enter your multi
line string here
line breaks will
be preserved
"@
```

Stick to single quotes whenever possible to avoid substitution

--

<!-- .slide: id="encoding" -->

## Hints: Encoding

Be careful when reading data from files or writing data to files

```powershell
Get-Content -Path myfile.txt -Encoding ASCII
```

ASCII is slowly disappearing, so you will have to use Unicode or UTF-8 more often

--

<!-- .slide: id="serialization" -->

## Hints: Serialization

Data structure can be serialized using `Export-CliXml` and `Import-CliXml`

Or you can convert to/from [JSON](#/json)

--

<!-- .slide: id="json" -->

## Hints: JSON

Data structures can be converted to/from JSON using `ConvertTo-Json` and `ConvertFrom-Json`

Note that `ConvertTo-Json` is limited to a `-Depth` of 3 by default

--

<!-- .slide: id="requires" -->

## Hints: Requirements

Place `#requires` in first line of script

Expect script to run as administrator:

```powershell
#requires -RunAsAdministrator
```

Expect modules to be installed:

```powershell
#requires -Modules pester
```

Expect PowerShell version:

```powershell
#requires -Version 5.0
```

--

<!-- .slide: id="pson" -->

## Hints: PSON (1)

PowerShell Object Notation (PSON) is a PowerShell specific alternative to JSON

The [module](#/sharing) manifest (`.psd1`) is expressed in PSON

```powershell
@{
    Name = 'Test'
    Tags = @('a', 'b', 'c')
}
```

Import into variable `$Data`:

```powershell
$Params = @{
    BindingVariable = 'Data'
    BaseDirectory   = '-'
    FileName        = 'test.psd1'
}
Import-LocalizedData @Params
```

--

## Hints: PSON (2)

PSON data structure with code must be parsed:

```powershell
@{
    Config = @(
        Path = (Join-Path -Path 'a' -ChildPath 'b')
    )
}
```

```powershell
# if stored in ps1 file
$Data = $(. '.\data.ps1')

# if stored in psd1 file
$Data = Get-Content -Raw -Path '.\data.psd1' | Invoke-Expression
```

--

<!-- .slide: id="psdefaultparametervalues" -->

## Hints: Default parameters

PowerShell can add parameters to cmdlets implicitly

This can be very useful for setting a proxy

```powershell
$PSDefaultParameterValues = @{
    'Install-Module:Proxy' = 'http://proxy.mydomain.com:3128'
}
```

...or adding credentials:

```powershell
$Cred = Get-Credential
$PSDefaultParameterValues = @{
    'New-PSSession:Credential' = $Cred
}
```

This applies to all calls in the same session

--

<!-- .slide: id="culture" -->

## Hints: Culture

Culture describes how data is displayed

Formatting of `[datetime]` according to a specific culture:

```powershell
$Culture = [System.Globalization.CultureInfo]'en-US'
Get-Date -Format $Culture.DateTimeFormat.ShortDatePattern
```

The same can be helpful for `[int]`:

```powershell
$Culture = [System.Globalization.CultureInfo]'de-DE'
$Thread = [System.Threading.Thread]::CurrentThread
$Thread.CurrentCulture = $Culture
$Thread.CurrentUICulture = $Culture

Invoke-Command { 1.4 }
```

--

<!-- .slide: id="datetime" -->

## Hints: Date/time parsing

Sometimes the following fails:

```powershell
Get-Date '24122014_022257'
```

Then you need to describe the format of the date:

```powershell
$usdate = '24122014_022257'
[datetime]::ParseExact($usdate, 'ddMMyyyy_HHmmss', $null)
```

`ParseExact` accepts a [list of standard string](https://docs.microsoft.com/en-us/dotnet/standard/base-types/standard-date-and-time-format-strings)

--

<!-- .slide: id="credentials" -->

## Hints: Credentials

Create a new credential object:

```powershell
$pw = ConvertTo-SecureString "P@ssw0rd!" -AsPlainText -Force
New-Object PSCredential -ArgumentList "username", $pw
```

Export and import credentials:

```powershell
# export
$Credential | Export-CliXml -Path .\cred.clixml

# import
$Credential = Import-CliXml -Path .\cred.clixml
```

Note that the password is stored as a `[SecureString]` which cannot be imported on other hosts

--

<!-- .slide: id="securestring" -->

## Hints: Plaintext from SecureString

Extract the plaintext password from `[SecureString]`:

```powershell
$BSTR = [System.Runtime.InteropServices.Marshal]::SecureStringToBSTR($SecureString)
[System.Runtime.InteropServices.Marshal]::PtrToStringAuto($BSTR)
```

--

<!-- .slide: id="regex" -->

## Hints: Regular expressions

Enable extended mode `(?x)` to use line breaks and comments:

```powershell
if ($Line -match '(?x)
    ^                           # Beginning of the line
    (?<SourceIp>\S+) \s         # IP address
    \S+ \s                      # deprecated
    (?<User>\S+) \s             # username (HTTP auth)
    \[(?<Timestamp>[^]]+)\] \s  # date enclosed in brackets
    "(?<Request>.+)" \s         # request (quoted)
    (?<Code>\d+) \s             # HTTP return code
    (?<Size>\d+|-) \s           # Size of response
    $                           # End of line
    ')
    {}
```

--

<!-- .slide: id="helpers" -->

## Hints: Useful functions

Many PowerShell enthusiasts have created their own library

Mine is called [Helpers](https://github.com/nicholasdille/PowerShell-Helpers)

--

<!-- .slide: id="gridview" -->

## Hints: Tabular Data

Tabular data can be visualized by using `Out-GridView`:

```powershell
Get-ChildItem | Out-GridView
```

`Out-GridView` can also be used as a simple dialog:

```powershell
Get-WindowsOptionalFeature -Online |
    Out-GridView -Title 'Select new features' -OutputMode Multiple |
    Enable-WindowsOptionalFeature -Online
```

--

<!-- .slide: id="elevation" -->

## Hints: Auto-Elevation

Some script require an elevation

```powershell
using namespace System.Security.Principal
$myWindowsPrincipal = New-Object WindowsPrincipal([WindowsIdentity]::GetCurrent())

# Check to see if we are currently running "as Administrator"
if (-not $myWindowsPrincipal.IsInRole([WindowsBuiltInRole]::Administrator)) {
    # We are not running "as Administrator" - so relaunch as administrator
    $newProcess = New-Object System.Diagnostics.ProcessStartInfo "PowerShell";
    $newProcess.Arguments = $myInvocation.MyCommand.Definition;
    $newProcess.Verb = "runas";
    [System.Diagnostics.Process]::Start($newProcess);
    # Exit from the current, unelevated, process
    exit
}

# Run your code that needs to be elevated here
```