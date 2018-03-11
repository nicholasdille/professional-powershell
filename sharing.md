<!-- .slide: id="sharing" -->

## Sharing is caring

Code is shared using modules

The official module repository is [PowerShell Gallery](https://www.powershellgallery.com/)

Managing modules has become easier in PowerShell 5:

```powershell
# search
Find-Module -Name pester

# install for all users
Install-Module -Name pester

# install for current user
Install-Module -Name pester -Scope CurrentUser
```

Learn how to [import a module using remoting](#/remote_module)

--

<!-- .slide: id="testing_modules" -->

## Testing modules

Modules should be tested using `Import-Module`

Set the module search path to the directory above your module:

```powershell
$env:PSModulePath = "c:\dev\modules;$env:PSModulePath"
Import-Module -Name MyModule
```

Using the following breaks some members like [formats](#/custom_formats):

```powershell
Import-Module c:\dev\modules\MyModule\MyModule.psm1
```

--

<!-- .slide: id="module_layout" -->

## Module directory layout

Module name: `MyModule`

```
\MyModule
    |
    +- MyModule.psd1   # module manifest
    +- MyModule.psm1   # root module
```

The module manifest is expressed in [PSON](#/pson)

Create a new module manifest: `New-ModuleManifest`

Place [custom formats](#/custom_formats) in the module root

--

<!-- .slide: id="module_variables" -->

## Module private variables

Declare variables in `MyModule\MyModule.psm1`:

```powershell
$script:MyVariable = @{}
```

Export variable:

```powershell
Export-ModuleMember -Variable 'MyVariable'
```

--

<!-- .slide: id="ci" -->

## Continuous integration

Run [script analysis](#/script_analysis) first

Run [unit tests](#/unit_tests)

Check [code coverage](#/code_coverage)

[My build scripts](https://github.com/nicholasdille/PowerShell-Build) are independent of the CI/CD tool but offer an example configuration for [AppVeyor](https://www.appveyor.com/)