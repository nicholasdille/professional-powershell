<!-- .slide: id="caveats" -->

## Caveats

The concepts Powershell is based on have some consequences that may seem surprising

--

<!-- .slide: id="pipeline_array" -->

## Caveats: Arrays and the pipeline

Array elements are passed into the pipeline individually

```powershell
PS> $Array = @('a', 'b', 'c')
PS> $Array.GetType()

IsPublic IsSerial Name     BaseType
-------- -------- ----     --------
True     True     Object[] System.Array

PS> $Array | ForEach-Object { $_ }
a
b
c
```

--

<!-- .slide: id="array_filters" -->

## Caveats: Filtering arrays

If filtering only returns one object, it is not treated as an array:

```powershell
$Data = @('foobar', 'blarg', 'arg', 'blubb')

# array
$Items = $Data | Where-Object { $_ like '*arg' }

# not an array
$Items = $Data | where-Object { $_ -like '*bar' }
```

Solved by wrapping result into array:

```powershell
$Items = @($Data | where-Object { $_ -like '*bar' })
```

--

<!-- .slide: id="array_performance" -->

## Caveats: Performance of arrays

For large arrays use `System.Collections.ArrayList`:

```powershell
# slow
$Data = @()
1..1mb | ForEach-Object { $Data += $_ }

# fast
$Data = New-Object -TypeName System.Collections.ArrayList
1..1mb | ForEach-Object { $Data.Add($_) }
```

--

<!-- .slide: id="parameter_prompt" -->

## Caveats: Avoid parameter prompts

If missing, PowerShell will prompt for [mandatory parameters](#/parameter_validation)

Use the following instead to throw:

```powershell
function Do-Something {
    [CmdletBinding()]
    param(
        [Parameter()]
        [ValidateNotNullOrEmpty()]
        [string]
        $Path = $(throw 'Please specify mandatory parameter "Path"')
    )
}
```

--

<!-- .slide: id="basic_authentication" -->

## Caveats: HTTP Basic auth

Before [PowerShell Core](#/cross_platform) 6, `Invoke-WebRequest` does not support authentication

```powershell
$AuthBytes = [System.Text.Encoding]::ASCII.GetBytes('user:pass')
$AuthString = [System.Convert]::ToBase64String($AuthBytes)
Invoke-WebRequest -Headers @{ Authorization = "Basic $AuthString" }
```

See also [ConvertTo-Base64](https://github.com/nicholasdille/PowerShell-Helpers/blob/master/Helpers/Public/ConvertTo-Base64.ps1) based on [ConvertTo-ByteArray](https://github.com/nicholasdille/PowerShell-Helpers/blob/master/Helpers/Public/ConvertTo-ByteArray.ps1)

My module [WebRequest](https://github.com/nicholasdille/PowerShell-WebRequest) introduces overrides to `Invoke-WebRequest` and `Invoke-RestMethod`

--

<!-- .slide: id="tls" -->

## Caveats: TLS 1.1/1.2

By default, Windows PowerShell 5 supports TLS 1.0 for `Invoke-WebRequest` and `Invoke-RestMethod`

The following code enables TLS 1.1 and TLS 1.2:

```powershell
$Backup = [System.Net.ServicePointManager]::SecurityProtocol
[System.Net.ServicePointManager]::SecurityProtocol = 'Tls11,Tls12'

# code requiring TLS 1.1 and/or TLS 1.2

[System.Net.ServicePointManager]::SecurityProtocol = $Backup
```
