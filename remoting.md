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

## CredSSP (1)

The Credential Security Support Provider (CredSSP) offers forwarding of credentials to remote hosts

Only CredSSP allows for multi-hop authentication

Forwarding must be configured using [group policy](https://msdn.microsoft.com/de-de/library/windows/desktop/bb204773%28v=vs.85%29.aspx?f=255&MSPPError=-2147217396) or the local policy

```powershell
Invoke-Command `
    -ComputerName MyRemoteHost `
    -Authentication Credssp `
    -Credential (Get-Credential)
    -Scriptblock { hostname }
```

--

## CredSSP (2)

The client must allow forwarding credentials using `TrustedHosts` (elevation required):

```powershell
Get-Item WSMan:\localhost\Client\TrustedHosts
Set-Item WSMan:\localhost\Client\TrustedHosts `
    -Value 'machineA,machineB'
Set-Item WSMan:\localhost\Client\TrustedHosts `
    -Value 'machineC' -Concatenate
Set-Item WSMan:\localhost\Client\TrustedHosts `
    -Value '10.0.*'
Set-Item WSMan:\localhost\Client\TrustedHosts `
    -Value '*'
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
Invoke-Command `
    -Session $Session `
    -Scriptblock { Import-Module 'Helpers' }
$RemoteModule = Import-PSSession `
    -Session $Session `
    -Module $ModuleName `
    -AllowClobber `
    -FormatTypeName *
Import-Module `
    -Name $RemoteModule `
    -Global
```

See [`Import-RemoteModule`](https://github.com/nicholasdille/PowerShell-Helpers/blob/master/Helpers/Public/Import-RemoveModule.ps1)

--

<!-- .slide: id="non_admin" -->

## Remoting as non-admin

XXX

XXX see [PSSessionConfiguration](#/pssessionconfiguration)

--

<!-- .slide: id="LocalSubnet" -->

## WinRM outside local subnet

XXX

--

<!-- .slide: id="pssessionconfiguration" -->

## PSSessionConfiguration

XXX