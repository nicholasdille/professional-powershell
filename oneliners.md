<!-- .slide: id="oneliner" -->

## Avoid oneliners

Honour line breaking to improve readibility

Refactor to multiple lines

--

## Avoid oneliners

Avoid scriptblocks in a pipeline:

```powershell
# this is bad
Get-ChildItem `
    | Select-Object -First 10 `
    | ForEach-Object { $_.FullName } `
    | Where-Object { $_ -like '*.exe' }

# this is good
$FilesTop10 = Get-ChildItem | Select-Object -First 10
foreach ($File in $FilesTop10)
{
    if ($File.FullName -like '*.exe')
    {
        $File.FullName
    }
}
```

This includes `Where-Object`, `ForEach-Object`