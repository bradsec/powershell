# Windows PowerShell Guides and Helpers
- Most of the below examples require PowerShell run as Administrator

## Windows 7/8
- Later version of Windows have removed the ability to remove hotfixes using /quiet. So this will not work with Windows 10 onwards.  
#### Get list of KB hotfixes
`Get-Hotfix | Select-Object -Property HotFixID`  
#### Get list and remove all KB hotfixes applied to system
`Get-Hotfix | Select-Object -Property HotFixID | ForEach-Object ({Start-Process -FilePath "wusa.exe" -ArgumentList "/uninstall ($_.HotFixID -replace 'KB','/kb:') /quiet /norestart" -Wait})`
#### Get list and show output cmd.exe command prompt text to remove individual hotfixes
`Get-Hotfix | Select-Object -Property HotFixID | ForEach-Object ({"wusa /uninstall " + ($_.HotFixID -replace 'KB','/kb:') + " /quiet /norestart"})`

## Windows 10/11
#### Remove any packages containing KB
`Get-WindowsPackage -Online | Where-Object {$_.PackageName -like "*KB*"} | ForEach-Object ({Remove-WindowsPackage -PackageName $_.PackageName -Online -NoRestart})`
#### Remove any any packages of ReleaseType SecurityUpdate
`Get-WindowsPackage -Online | Where-Object {$_.ReleaseType -like "SecurityUpdate"} | ForEach-Object ({Remove-WindowsPackage -PackageName $_.PackageName -Online -NoRestart})`

## Windows 10/11 SSH
#### Get details of WindowsCapability OpenSSH
`Get-WindowsCapability -Online | Where-Object Name -like 'OpenSSH*'`
#### Check for installed
`Get-WindowsCapability -Online | Where-Object Name -like 'OpenSSH*' | ForEach-Object ({if ($_.State -eq "Installed") {$_.Name}})`
#### Add SSH server and client if not present
`Get-WindowsCapability -Online | Where-Object Name -like 'OpenSSH*' | ForEach-Object ({if ($_.State -ne "Installed") {Add-WindowsCapability -Online -Name $_.Name}})`
#### Remove SSH server and client if present
`Get-WindowsCapability -Online | Where-Object Name -like 'OpenSSH*' | ForEach-Object ({if ($_.State -eq "Installed") {Remove-WindowsCapability -Online -Name $_.Name}})`
#### Start SSH service
`Start-Service sshd`
#### Enable automatic SSH service startup
`Set-Service -Name sshd -StartupType 'Automatic'`
#### Confirm the Firewall rule is configured.
```
# Source https://docs.microsoft.com/en-us/windows-server/administration/openssh/openssh_install_firstuse

if (!(Get-NetFirewallRule -Name "OpenSSH-Server-In-TCP" -ErrorAction SilentlyContinue | Select-Object Name, Enabled)) {
    Write-Output "Firewall Rule 'OpenSSH-Server-In-TCP' does not exist, creating it..."
    New-NetFirewallRule -Name 'OpenSSH-Server-In-TCP' -DisplayName 'OpenSSH Server (sshd)' -Enabled True -Direction Inbound -Protocol TCP -Action Allow -LocalPort 22
} else {
    Write-Output "Firewall rule 'OpenSSH-Server-In-TCP' has been created and exists."
}
```
- Should return `Firewall rule 'OpenSSH-Server-In-TCP' has been created and exists.`

#### To fix issue where capability fails to install (ie. cabability still shows as NotPresent)
- Edit group policy  
- `Computer Configuration` > `Administrative Templates` > `System`  
- Set values in `Specify settings for optional component installation and component repair`  
- - Enabled
- - Download repair content and optional features directly from Windows Update instead of Windows Server Update Services (WSUS)
- Reboot system and try Add-WindowsCapability again.
