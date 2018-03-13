<!-- .slide: id="code_quality" -->

## Code Quality

Run script analysis to understand how Microsoft and the community expect code to be written

Write unit tests for your functions

Get someone to review your code

--

<!-- .slide: id="script_analysis" -->

## Script analysis (1)

The [PSScriptAnalyzer]() checks code for typical mistakes:

```powershell
PS> Get-Content -Path .\New-Credential.ps1
function New-Credential
{
    param($User, $Pass)
    $SPass = ConvertTo-SecureString $Pass -AsPlainText -Force
    New-Object PSCredential -ArgumentList $User, $SecPass
}

PS> Invoke-ScriptAnalyzer .\New-Credential.ps1 |
    Select-Object -ExpandProperty RuleName
PSAvoidUsingUserNameAndPassWordParams
PSAvoidUsingConvertToSecureStringWithPlainText
PSAvoidUsingPlainTextForPassword
PSUseShouldProcessForStateChangingFunctions
```

Also see the [official documentation](https://github.com/PowerShell/PSScriptAnalyzer#usage)

--

## Script analysis (2)

Suppressing issues:

```powershell
function New-Credential
{
    [Diagnostics.CodeAnalysis.SuppressMessageAttribute(
        'PSUseShouldProcessForStateChangingFunctions',
        '',
        Justification='No state is changed'
    )]
    param($User, $Pass)
    $SPass = ConvertTo-SecureString $Pass -AsPlainText -Force
    New-Object PSCredential -ArgumentList $User, $SecPass
}
```

Show suppressed issues:

```powershell
Invoke-ScriptAnalyzer .\New-Credential.ps1 -SuppressedOnly
```

--

<!-- .slide: id="unit_tests" -->

## Unit tests

[Pester](https://github.com/pester/Pester/wiki/Pester) is a framework for BDD

```powershell
. .\New-Credential.ps1

Describe 'New-Credential' {
    It 'Produces PSCredential object' {
        New-Credential -User a -Pass b |
            Should BeOfType PSCredential
    }
    It 'Embed correct user' {
        New-Credential -User a -Pass b |
            Select-Object -ExpandProperty UserName |
            Should Be 'a'
    }
}
```

--

<!-- .slide: id="code_coverage" -->

## Code coverage

Pester can also calculate code coverage

```powershell
PS> Invoke-Pester .\New-Credential.ps1 -CodeCoverage .\New-Credential.ps1
Executing all tests in .\New-Credential.ps1

Executing script .\New-Credential.ps1

Describing New-Credential
    [+] Produces PSCredential object 191ms
    [+] Embed correct user 30ms
Tests completed in 221ms
Tests Passed: 2 Failed: 0 Skipped: 0 Pending: 0 Inconclusive: 0

Code coverage report:
Covered 100,00 % of 10 analyzed Commands in 1 File.
```

The result is also exposed when adding `-PassThru`

--

<!-- .slide: id="cross_platform" -->

## Cross platform

With [PowerShell Core](https://github.com/Powershell/powershell) 6, Microsoft published a cross platform version available on <i class="fab fa-windows"></i> <i class="fab fa-linux"></i> <i class="fab fa-apple"></i>

All code should be tested against PowerShell Core
