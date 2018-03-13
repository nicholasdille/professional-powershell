<!-- .slide: id="pipelines" -->

## Pipelines

To stream or not to stream...

Only use stream processing if you can operate on individual objects independent of one another

Good idea: Substitute in a list of strings

Bad idea: Calculate the mean of a list of values

Always a good idea: Accept parameters from the pipeline, e.g. files to process:

```powershell
Get-ChildItem | Remove-Item
```

--

<!-- .slide: id="pipeline_functions" -->

## Pipelines with functions (1)

```powershell
function Do-Something
{
    [CmdletBinding()]
    param(
        [Parameter(ValueFromPipeline)]
        [string[]]
        $InputObject
    )
    process
    {
        "InputObject: $InputObject"
    }
}
```

`ValueFromPipeline` enables the pipeline

--

## Pipelines with functions (2)

`process` block is called for every input object

```powershell
PS> Do-Something 'a', 'b', 'c'
InputObject: a b c

PS> 'a', 'b', 'c' | Do-Something
InputObject: a
InputObject: b
InputObject: c
```

Loop through `$InputObject`:

```powershell
$InputObject | ForEach-Object
{
    "item: $_"
}
```

--

<!-- .slide: id="ByProperty" -->

## Objects on the pipeline (1)

Instead of using two [parameter sets](#/parameter_sets), extract the parameter from an object on the [pipeline](#/pipelines):

```powershell
function Do-Something
{
    [CmdletBinding()]
    param(
        [Parameter(Mandatory,
            ValueFromPipeline,
            ValueFromPipelineByPropertyName
        )]
        [int]$Id
    )
    $Id
}
6 | Do-Something
[pscustomobject]@{Id = 6} | Do-Something
```

--

## Objects on the pipeline (2)

Access a different property by parameter aliases

```powershell
function Do-Something
{
    [CmdletBinding()]
    param(
        [Parameter(Mandatory,
            ValueFromPipeline,
            ValueFromPipelineByPropertyName
        )]
        [Alias('Name')]
        [int]$Id
    )
    $Id
}
6 | Do-Something
[pscustomobject]@{Name = 6} | Do-Something
```

--

<!-- .slide: id="debugging_pipelines" -->

## Debugging pipelines

`-OutVariable` can be used to obtain result of individual steps:

```powershell
Get-ChildItem -OutVariable gci |
    Where-Object { Length -ne $null } -OutVariable where |
    Group-Object -Property Extension -OutVariable group |
    Sort-Object -Property Count -OutVariable sort |
    Select-Object -Last 1
```

A single execution suffices to debug all steps of the pipeline
