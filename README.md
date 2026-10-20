# Nodren 1.1.3

This folder contains the Nodren 1.1.3 Windows x64 release: the desktop
application, CLI, Worker, native Core library, and installers.

## Choose an installer

- **`Install-Nodren-1.1.3.exe`** installs the desktop application, its managed
  local Controller, CLI, Worker, and native Core.
- **`Install-Nodren-Worker-1.1.3.exe`** installs only the Worker and native
  Core for a separate machine.

Run the installer and follow its prompts. The desktop application manages its
local Controller; no separate Controller service is installed. The Worker-only
installer does not start a service, open firewall ports, or start the Worker
automatically. Both installers register a Windows uninstaller.

The portable release files are `NodrenApp.exe`, `nodren.exe`,
`nodren-worker.exe`, and `nodren-core.dll`. For normal installation, use the
installer rather than launching individual files from this folder.

## Connect a Worker

The Worker must be configured with the Controller address, a Worker ID, and a
matching unique token issued by the Controller administrator. From PowerShell
in the Worker installation directory:

```powershell
$secureToken = Read-Host "Worker token" -AsSecureString
$env:NODREN_WORKER_TOKEN = [System.Net.NetworkCredential]::new("", $secureToken).Password
Remove-Variable secureToken
.\Worker.exe --controller <controller-host>:9000 --id <worker-id>
```

Use the actual Controller host and assigned Worker ID in place of the
placeholders. The token is held in the current PowerShell session; close that
session when finished. Worker TCP traffic is authenticated and integrity
protected, but not encrypted. Use a trusted private network or encrypted
tunnel. Do not put real tokens in documentation, command history, or shared
files.

## Supported workloads and limitations

This release distributes the built-in native `sum`, `xor`, and `dot_product`
workloads. Process, command, script, AI, and extension-code execution are
disabled. The desktop application's numeric `.txt` and `.csv` inputs are
parsed locally and submitted only as data for those built-in workloads.

This is a Windows x64 application release. A Linux Nodren application release
is not included.

## Verify the files

`SHA256SUMS.txt` lists hashes for the packaged installers, binaries, and
`build-info.json`; it does not hash itself or this README. In PowerShell,
verify the listed files from this directory:

```powershell
foreach ($line in Get-Content .\SHA256SUMS.txt) {
    if ($line -match '^([0-9a-f]{64}) \*([^*]+)$') {
        $expected = $Matches[1]
        $name = $Matches[2]
        $actual = (Get-FileHash -LiteralPath $name -Algorithm SHA256).Hash.ToLowerInvariant()
        if ($actual -ne $expected) {
            throw "Checksum mismatch: $name"
        }
        Write-Output "Verified $name"
    } else {
        throw "Invalid checksum entry: $line"
    }
}
```

`build-info.json` records the release version, architecture, build
configuration, source commit, source-tree state, source fingerprint, and
payload hashes. This build is Nodren **1.1.3**, Windows **x64**, **Release**;
its source commit is `64b1f6bc8d3abe275770e2c247785b9a9f15cba2`. The metadata
records that the source tree was dirty when built, so the commit alone does
not identify every source change included in these binaries.

## Upgrade and uninstall

Use the corresponding Nodren installer to install or upgrade. User settings
and Controller state are stored separately from program files. Uninstallation
removes installed program files and shortcuts while preserving user data.
Review the installer's prompts and the project's installer documentation
before changing an existing installation.
