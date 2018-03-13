<!-- .slide: id="remoting" -->

## Remoting

Executing commands on remote hosts

--

<!-- .slide: id="invoke_command" -->

## Invoke-Command

Passing variables into remote scriptblocks is tricky:

```powershell
$Data = 'Import value'
Invoke-Command {
    $Using:Data
}
```

When `Invoke-Command` is started with `-AsJob` it becomes a remote [job](#/jobs)

--

<!-- .slide: id="credssp" -->

## CredSSP

The Credential Security Support Provider (CredSSP) offers forwarding of credentials to remote hosts

Only CredSSP allows for multi-hop authentication

Forwarding must be configured using [group policy](https://msdn.microsoft.com/de-de/library/windows/desktop/bb204773%28v=vs.85%29.aspx?f=255&MSPPError=-2147217396) or the local policy

```powershell
$InvokeCommandParameters = @{
    ComputerName = MyRemoteHost
    Authentication = Credssp
    Credential  =(Get-Credential)
    Scriptblock = { hostname }
}
Invoke-Command @InvokeCommandParameters
```

--

<!-- .slide: id="trustedhosts" -->

## WinRM TrustedHosts

The client must either use HTTPS or allow forwarding credentials using `TrustedHosts` (elevation required):

```powershell
Get-Item WSMan:\localhost\Client\TrustedHosts

Set-Item WSMan:\localhost\Client\TrustedHosts -Value 'machineA,machineB'

Set-Item WSMan:\localhost\Client\TrustedHosts -Value 'machineC' -Concatenate

Set-Item WSMan:\localhost\Client\TrustedHosts -Value '10.0.*'

Set-Item WSMan:\localhost\Client\TrustedHosts -Value '*'
```

Module [psTrustedHosts](https://github.com/jasonmcboyd/psTrustedHosts) managed this list

--

<!-- .slide: id="pssessionoption" -->

## PSSessionOption

By default, sessions receive a default configuration (see `New-PSSessionOption`)

Sessions can be started with a modified configuration:

```powershell
# idle timeout of 5 minutes
$Option = New-PSSessionOption -IdleTimeout 300000
New-PSSession -SessionOption $Option
```

--

<!-- .slide: id="reconnect" -->

## Reconnect session

Clients can disconnect and reconnect with a session

```powershell
# connect to a host
$Session = New-PSSession -ComputerName 'RemoteHost'

# disconnect the session
Disconnect-PSSession -Session $Session

# reconnect a disconnected session
Connect-PSSession -Session $Session

# retrieve a session
Get-PSSession -ComputerName 'RemoteHost'
```

If this fails, the session has ended and you need to specify a [PSSessionOption](#/pssessionoption)

--

<!-- .slide: id="remoting_proxy" -->

## Remoting through a proxy

Only works through HTTPS proxy to prevent MitM attacks:

```powershell
$Option = New-PSSessionOption -ProxyAccessType AutoDetect
New-PSSession -SessionOption $Option
```

--

<!-- .slide: id="remote_module" -->

## Remote module

PowerShell can even import a remote [module](#/sharing)

```powershell
$InvokeCommandParameters = @{
    Session = $Session
    Scriptblock = { Import-Module 'Helpers' }
}

Invoke-Command @InvokeCommandParameters

$RemoteModuleParameters = @{
    -Session        = $Session
    -Module         = $ModuleName
    -AllowClobber   = $true
    -FormatTypeName = *
}
$RemoteModule = Import-PSSession @RemoteModuleParameters

Import-Module -Name $RemoteModule -Global
```

See [`Import-RemoteModule`](https://github.com/nicholasdille/PowerShell-Helpers/blob/master/Helpers/Public/Import-RemoveModule.ps1)

--

<!-- .slide: id="non_admin" -->

## Remoting as non-admin

By default, only admins are able to use remoting

Controlled by the local `Remote Management Users` group

Alternatively, change the session configuration:

```powershell
Set-PSSessionConfiguration -Name Microsoft.PowerShell -ShowSecurityDescriptorUI
```

Or create a new endpoint using [PSSessionConfiguration](#/pssessionconfiguration)

--

<!-- .slide: id="LocalSubnet" -->

## WinRM outside local subnet

By default, WinRM on public networks is only available from the same subnet:

```powershell
PS> $PublicProfile = Get-NetFirewallProfile -Name Public

PS> $Rule = $PublicProfile |
        Get-NetFirewallRule |
        Where-Object Name -eq 'WINRM-HTTP-In-TCP'

PS> $Filter = $Rule | Get-NetFirewallAddressFilter

PS> $Filter.RemoteAddress
LocalSubnet
```

Fix this:

```powershell
$Rule | Set-NetFirewallRule -RemoteAddress Any
```

--

<!-- .slide: id="pssessionconfiguration" -->

## PSSessionConfiguration (1)

PowerShell support multiple [endpoints for remoting](https://docs.microsoft.com/en-us/powershell/module/microsoft.powershell.core/about/about_session_configurations?view=powershell-6)

Endpoints can have different permissions and capabilities

```powershell
# create a new endpoint
Register-PSSessionConfiguration -Name MyNewEndpoint

# configure endpoint
Set-PSSessionConfiguration -Name MyNewEndpoint #...

# use endpoint
Enter-PSSession -Configuration Name MyNewEndpoint

# remove endpoint
Unregister-PSSessionConfiguration -Name MyNewEndpoint
```

--

## PSSessionConfiguration (2)

Endpoints can be restricted:

```powershell
# create session configuration file
New-PSSessionConfigurationFile -Path .\MyNewEndpoint.pssc

# edit file

# register endpoint
$RegisterPsSessionParameters = @{
    Name = MyNewEndpoint
    Path = .\MyNewEndpoint.pssc
}
Register-PSSessionConfiguration @RegisterPsSessionParameters
```

--

## PSSessionConfiguration (3)

`LanguageMode` defines which language features are available

- `FullLanguage` for no restrictions

- [`RestrictedLanguage`](https://docs.microsoft.com/en-us/powershell/module/microsoft.powershell.core/about/about_language_modes?view=powershell-6) only offers simple comparison operators and few builtin varialbes

- `NoLanguage` only allows existing cmdlets to be executed

--

## PSSessionConfiguration (4)

`SessionType` prepares a session configuration:

- `Default` offers `FullLanguage` and adds snap-in [`Microsoft.PowerShell.Core`](https://docs.microsoft.com/en-us/powershell/module/microsoft.powershell.core/?view=powershell-5.1)

- `Empty` does not load any snap-ins (therefore, no `Import-Module` and `Add-PSSnapin`) and offers `NoLanguage`

- `RestrictedRemoteServer` adds very few cmdlets (e.g. `Exit-PSSession`) and offers `NoLanguage`
