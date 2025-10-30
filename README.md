Allow this PowerShell session to run anything + force TLS 1.2

Set-ExecutionPolicy -Scope Process Bypass -Force — lets the current PowerShell process only run scripts/modules without policy prompts (doesn’t change machine/user policy permanently).

ServicePointManager.SecurityProtocol = Tls12 — forces TLS 1.2 for any HTTPS calls (needed to talk to PSGallery/NuGet on modern endpoints).

Prep package sources (NuGet + PowerShell Gallery)

Install-PackageProvider -Name NuGet ... — ensures the NuGet provider exists so PowerShell can download modules.

Set-PSRepository -Name PSGallery -InstallationPolicy Trusted — marks PSGallery as “trusted” to avoid interactive prompts when installing.

Remove any old/broken PSWindowsUpdate installs

Get-InstalledModule PSWindowsUpdate ... | Uninstall-Module ... — uninstalls all versions of the module.

Remove-Item ... \Modules\PSWindowsUpdate (both ProgramFiles and User profile) — force-deletes residual folders so you start clean.

Freshly install and load PSWindowsUpdate

Install-Module PSWindowsUpdate -Scope AllUsers -Force — installs the latest module for all users on the machine.

Import-Module PSWindowsUpdate -Force — loads it into the current session.

Unblock module files (defense-in-depth) and re-import

Enumerates the module folder and runs Unblock-File on every file (avoids “downloaded from the internet” blocks).

Re-imports the module to ensure everything’s loadable.

Verify the module is present/usable

Get-Module PSWindowsUpdate -ListAvailable | Select Name,Version,ModuleBase — shows what’s installed and where.

Get-Command -Module PSWindowsUpdate | Select Name — lists available cmdlets from the module.

Dry-run core cmdlets (no installs yet)

Get-WindowsUpdate — queries available updates (from Windows Update/WSUS depending on policy) and lists them.

Get-WURebootStatus — reports if a reboot is pending from prior operations.

Perform updates

Get-WindowsUpdate -MicrosoftUpdate -AcceptAll -Install

Switches the catalog to include Microsoft Update (Office, etc.),

auto-accepts all found updates,

and installs them.
Expect downloads, installs, and possibly a reboot requirement at the end.

Remove GUI-blocking Windows Update policy and refresh

reg delete "HKLM\SOFTWARE\Policies\Microsoft\Windows\WindowsUpdate" /f — deletes the whole policy key that can disable or redirect Windows Update GUI (e.g., enforced WSUS/deferral settings).

gpupdate /force — reapplies Group Policy; if a GPO reenforces that key, it may reappear after this.

(After reboot) Check if the policy came back

reg query HKLM\SOFTWARE\Policies\Microsoft\Windows\WindowsUpdate /s — verifies whether any policy values were recreated by GPO.

Kick a Windows Update scan via USO client

usoclient StartScan — tells the Windows Update Orchestrator to start a detection cycle (useful after policy changes).
