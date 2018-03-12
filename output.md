<!-- .slide: id="output" -->

## Output

PowerShell supports several stream for different types of information:

- Output
- Error
- Warning
- Verbose
- Debug
- Information (since PowerShell 5)

[Redirect](https://docs.microsoft.com/de-de/powershell/module/microsoft.powershell.core/about/about_redirection?view=powershell-6&viewFallbackFrom=powershell-Microsoft.PowerShell.Core) to merge streams

--

<!-- .slide: id="write_host" -->

## Output: Write-Host

Avoid `Write-Host` because it does not add to a stream

`Write-Host` Output is only sent to the console to display, if no console is there (such as running in a background process) then no output is displayed

--

<!-- .slide: id="verbose_debug" -->

## Output: Verbose and debug

Use `Write-Verbose` to allow users to follow your code

Use `Write-Debug` to allow users to debug your code

Which output is displayed is controlled with by [preference variables](https://docs.microsoft.com/en-us/powershell/module/microsoft.powershell.core/about/about_preference_variables?view=powershell-6) or by parameters in [advanced functions](#/advanced_functions)

--

<!-- .slide: id="information_stream" -->

## Output: Information stream

New in PowerShell 5.0 to informational (non-verbose) messages

```powershell
Write-Information 'message'
```

Capturing the information stream:

```powershell
Get-ChildItem -InformationAction Continue -InformationVariable InfoStream
$InfoStream
```

Note: The above will not produce any useful output
