# Install Merlin Operations Suite

## Download and install

1. On a Windows x64 workstation, open [Merlin GitHub Releases](https://github.com/atelierzane/Merlin2026-Releases/releases/latest).
2. Download `MerlinOperationsSuite-win-Setup.exe` for an interactive installation, or `MerlinOperationsSuite-win.msi` for deployment by an administrator. Download the installer assets, rather than GitHub's source-code ZIP.
3. Run the installer. It installs all six applications and the suite launcher together. Allow elevation when requested. The installer can download Microsoft Visual C++ 2022 x64 and Edge WebView2 if they are missing. The .NET runtime is already included.
4. Launch **Merlin Operations Suite** from the desktop or Start menu. First-run setup guides database connection, store/register enrollment, administrator creation, and the required imports. A cancelled setup resumes on the next launch.
5. After setup, the launcher opens the business day through Polling Agent when required, then starts Point of Sale.

The installer can be unsigned when the release was built without a signing certificate. Verify its origin and compare `Get-FileHash <installer> -Algorithm SHA256` with the release's `SHA256SUMS` before deployment.

## Database connection

Merlin requires a reachable SQL Server database. SQL Server is separate from the application installer. On the store server, install SQL Server Express or use an existing SQL Server instance. New-store setup can create and migrate the database when the connected account has permission.

For a single machine, the default is `.\SQLEXPRESS` and database `MerlinDatabase_Base` with Windows authentication. Setup also accepts another server, database, or supported authentication configuration.

For multiple workstations, create the store on the server first, then choose **Join Existing Store** on each register and use the same server/database. Configure SQL Server TCP/IP, firewall access, and account permissions for those machines. The optional `support/Configure-MerlinStoreServer.ps1` helper in the installed package runs from an elevated PowerShell session on the server; it changes network settings only after confirmation. It does not install SQL Server or grant database logins.

Blank CSV import templates are included under `current/support/import-templates` in an installed package. `current/support/FIELD-GUIDE.md` explains field requirements and import order.

## Managed or offline deployment

The MSI installs per machine under Program Files. An elevated deployment tool can run:

```powershell
msiexec.exe /i MerlinOperationsSuite-win.msi /qn /norestart /L*v Merlin-install.log
```

Check the Windows Installer exit code through your deployment tool (`0` is success; `3010` is success with a restart required). Preinstall Visual C++ 2022 x64 and Edge WebView2 for unattended or offline deployments. SQL Server must also be available before setup can finish. No .NET SDK, Visual Studio, developer checkout, or separate .NET installation is needed for the packaged applications.

The portable ZIP includes the same self-contained binaries but does not register installation, shortcuts, or the normal installed update lifecycle. Use Setup or MSI for production workstations.

## Upgrades and troubleshooting

All applications update as one suite. The update host checks the GitHub feed, downloads and verifies the new package, and asks before restarting applications to apply it. Deploy newer versions monotonically; do not reset the suite to a version below an installed release.

Version 1.1.0 moves downloads and updates to the public releases-only repository. To migrate an older installation whose feed points to the private `Merlin2026` repository, run the 1.1.0 installer once. It preserves user configuration and the SQL database, then future updates use the public feed.

Store configuration and logs are under `%LocalAppData%\Merlin` for each Windows user. Configuration survives upgrades and reinstallations. Each Windows user on a shared machine must enroll their own client profile. Business data remains in SQL Server. Keep database backups as part of store operations.

If launch or setup fails, inspect `%LocalAppData%\Merlin\Logs\startup.log`; update diagnostics are under `%LocalAppData%\Merlin\Updates\Logs`. Check SQL Server connectivity and permissions first. Do not delete configuration to repair an application installation; use the setup/recovery choices provided by the launcher.
