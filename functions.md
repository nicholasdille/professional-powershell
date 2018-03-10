<!-- .slide: id="functions" -->

## Functions

Isolate functionality into functions and modules

--

<!-- .slide: id="parameter_validation" -->

## Functions: Parameter validation

Use integrated parameter validation

```powershell
function Do-Something
{
    param(
        [Parameter(Mandatory)]
        [ValidateNotNullOrEmpty()]
        [string]
        $Path
    )

    #
}
```

More about [parameters and validation](https://docs.microsoft.com/en-us/powershell/module/microsoft.powershell.core/about/about_functions_advanced_parameters?view=powershell-6)

--

<!-- .slide: id="get_help" -->

## Functions: Get-Help

Add comment based help

```powershell
<#
.SYPNOSIS
Short description
#>
function Do-Something {}
```

Supported [keywords in comment based help](https://docs.microsoft.com/en-us/powershell/module/microsoft.powershell.core/about/about_comment_based_help?view=powershell-6#comment-based-help-keywords)

--

<!-- .slide: id="advanced_functions" -->

## Advanced functions

Promote to cmdlet because...

- Advanced feature like [ShouldProcess](#/shouldprocess) and `-WhatIf` and `-Confirm`

- Parameters for controlling verbosity (`-Verbose` and `-Debug`)

```powershell
function Do-Something
{
    [CmdletBinding()]
    param()

    #
}
```

--

<!-- .slide: id="return_values" -->

## Functions: Return values

Be careful of additional output

```powershell
PS> function Test-Output
{
    'string output'
    42
}
PS> Test-Output | ForEach-Object { "object: $_" }
string output
42
```

Use `Write-Information`, `Write-Verbose` or `Write-Debug`

--

<!-- .slide: id="valuefrompipeline" -->

## Functions: Pipeline

See [pipelines](#/pipelines)

--

<!-- .slide: id="parameter_sets" -->

## Parameter sets (1)

Functions can accept multiple sets of parameters

```powershell
function Do-Something
{
    param(
        [Parameter(ParameterSetName='ById')]
        [int]$Id,
        [Parameter(ParameterSetName='ByObject')]
        [object]$Object,
        [Parameter(ParameterSetName='ById')]
        [Parameter(ParameterSetName='ByObject')]
        [switch]$PassThru
    )
    if ($PSCmdlet.ParameterSetName -ieq 'ByObject') {}
}
```

--

<!-- .slide: id="defaultparameterset" -->

## Parameter sets (2)

PowerShell cannot determine the current set if now parameters are supplied

The `DefaultParameterSetName` helps PowerShell to resolve this by assuming one by default:

```powershell
function Do-Something
{
    [CmdletBinding(DefaultParameterSetName='ById')]
    param(
        [Parameter(ParameterSetName='ById')]
        [int]$Id,
        [Parameter(ParameterSetName='ByObject')]
        [object]$Object
    )
}
```

<!-- .slide: id="shouldprocess" -->

## Functions: Confirmation (1)

Advanced functions support `ShouldProcess` to ask for user confirmation

```powershell
function Do-Something
{
    [CmdletBinding(
        SupportsShouldProcess,
        ConfirmImpact='Low'
    )]
    param($InputObject)

    if ($PSCmdlet.ShouldProcess($InputObject))
    {
        'do it'
    }
}
```

--

## Functions: Confirmation (2)

`ConfirmImpact` determines if a confirmation is necessary based on [`$ConfirmationPreference`](https://docs.microsoft.com/en-us/powershell/module/microsoft.powershell.core/about/about_preference_variables?view=powershell-6)

`ShouldProcess` enables new parameters...

- ...for testing (`-WhatIf`) and...
- ...controlling confirmation (`-Confirm`)

A detailled example can be found [here](http://dille.name/blog/2017/08/27/how-to-use-shouldprocess-in-powershell-functions/)

--

<!-- .slide: id="dynamic_functions" -->

## Dynamic functions

Functions are expose by `Function:\`

```powershell
$Code = {
    param(
        [Parameter(Mandatory)]
        [ValidateNotNullOrEmpty()]
        [string]
        $Path
    )

    if (-not (Test-Path -Path $Path)) {
        throw "Path '$Path' does not exist"
    }
}

New-Item -Path Function:\Do-Something -Value $Code -Force
```

--

<!-- .slide: id="dynamic_parameters" -->

## Dynamic parameters (1)

Parameters can be [created and populated during runtime](https://docs.microsoft.com/en-us/powershell/module/microsoft.powershell.core/about/about_functions_advanced_parameters?view=powershell-6#dynamic-parameters)

I recommend to use [`New-DynamicParameter`](https://github.com/beatcracker/Powershell-Misc/blob/master/New-DynamicParameter.ps1):

```powershell
function Do-Something
{
    DynamicParam {
        @(
            [psobject]@{
                Name = 'Item'
                Type = [string]
                Mandatory = $true
                ValidateSet = (Get-ChildItem).Name
            }
        ) | New-DynamicParameter
    }
}
```

--

## Dynamic parameters (2)

`New-DynamicParameter` also creates proper variables:

```powershell
function Do-Something
{
    DynamicParam {
        #...
    }

    New-DynamicParameter -CreateVariables -BoundParameters $PSBoundParameters
}
```