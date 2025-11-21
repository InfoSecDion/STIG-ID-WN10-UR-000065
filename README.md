# STIG-ID-WN10-UR-000065

## Synopsis
This PowerShell script configures the ‘Change the system time’ (SeSystemtimePrivilege) user right.

## Notes
- **Author**: Dion Alexander
- **LinkedIn**: 
- **GitHub**: 
- **Date Created**: 2025-11-20
- **Last Modified**: 2025-11-20
- **Version**: 1.0
- **CVEs**: N/A
- **Plugin IDs**: N/A
- **STIG-ID**: WN10-UR-000065
  
## Tested On
- **Date(s) Tested**: 
- **Tested By**: 
- **Systems Tested**: 
- **PowerShell Ver.**: 

## Usage
Put any usage instructions here.

Example syntax:

Example syntax:

```powershell
# STIG: Configure "Change the system time" (SeSystemtimePrivilege) user right

# Requires administrative privileges
if (-not ([Security.Principal.WindowsPrincipal] [Security.Principal.WindowsIdentity]::GetCurrent()
    ).IsInRole([Security.Principal.WindowsBuiltInRole]::Administrator)) {

    Write-Host "This script must be run as Administrator." -ForegroundColor Red
    exit 1
}

# Define the policy name and the groups/accounts to include
$PolicyName    = "SeSystemtimePrivilege"
$Accounts      = @("Administrators", "LOCAL SERVICE", "NT SERVICE\autotimesvc")
$AccountsValue = $Accounts -join ", "

# Define the temporary configuration file path
$configPath = "C:\Temp\secpol.cfg"

# Ensure the Temp directory exists
if (-not (Test-Path -Path "C:\Temp")) {
    New-Item -ItemType Directory -Path "C:\Temp" -Force | Out-Null
}

# Export the current security policy
Write-Host "Exporting current security policy to $configPath..."
secedit /export /cfg $configPath | Out-Null

# Verify the export was successful
if (-not (Test-Path -Path $configPath)) {
    Write-Error "Failed to export the security policy. Ensure you have the necessary permissions."
    exit 1
}

# Read the configuration file
$config = Get-Content $configPath

# Check if the policy already exists and update or add it
$policyFound = $false
for ($i = 0; $i -lt $config.Count; $i++) {
    if ($config[$i] -match "^\s*$PolicyName\s*=") {
        $config[$i] = "$PolicyName = $AccountsValue"
        $policyFound = $true
        break
    }
}

# If the policy is not found, add it to the configuration
if (-not $policyFound) {
    $config += "$PolicyName = $AccountsValue"
}

# Save the updated configuration back to the file
Set-Content -Path $configPath -Value $config

# Apply the updated security policy
Write-Host "Applying updated security policy..."
secedit /configure /db "C:\Windows\security\Database\secedit.sdb" /cfg $configPath /areas USER_RIGHTS | Out-Null

# Cleanup: Remove the temporary configuration file
if (Test-Path -Path $configPath) {
    Remove-Item -Path $configPath -Force
} else {
    Write-Warning "Temporary file '$configPath' does not exist and could not be removed."
}

# Output success message
Write-Host "Policy 'Change the system time' (SeSystemtimePrivilege) has been configured with the specified groups/accounts."
Write-Host "Running gpupdate /force to refresh policy..."
gpupdate /force | Out-Null
```
