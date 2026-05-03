# PowerShell Automation: Bulk User Creation

This section documents the PowerShell script I wrote to automate
large-scale user provisioning — simulating a real enterprise onboarding event.

---

## The Problem This Solves

In a real company, when a new branch opens or a merger happens,
IT may need to create hundreds of accounts in Active Directory fast.
Doing this manually (right-click → New User, repeat 500 times) is
slow, error-prone, and doesn't scale. PowerShell scripting is the
industry-standard solution.

---

## What the Script Does

1. Reads a plain text file (`names.txt`) containing first and last names
2. For each name, generates:
   - A username in format `firstname.lastname` (e.g. `sarah.johnson`)
   - A standardized initial password (`Password1`)
3. Creates the user in Active Directory under the `_USERS` OU
4. Sets the password and enables the account
5. Logs the result to the terminal

**Total users created in this simulation: 1,000+**

---

## The Script

```powershell
# ----- BULK USER CREATION SCRIPT -----
# Reads names from names.txt and creates AD users automatically

# --- CONFIGURATION — Edit these values ---
$PASSWORD_FOR_USERS   = "Password1"
$USER_FIRST_LAST_LIST = Get-Content .\names.txt
$password = ConvertTo-SecureString $PASSWORD_FOR_USERS -AsPlainText -Force
$OU = "ou=_USERS,dc=mydomain,dc=com"

# Import AD module
Import-Module activedirectory

# --- MAIN LOOP ---
foreach ($n in $USER_FIRST_LAST_LIST) {
    $first = $n.Split(" ")[0].ToLower()
    $last  = $n.Split(" ")[1].ToLower()
    $username = "$($first.Substring(0,1))$($last)".ToLower()

    Write-Host "Creating user: $($username)" -BackgroundColor Black -ForegroundColor Cyan

    New-AdUser `
        -AccountPassword $password `
        -GivenName $first `
        -Surname $last `
        -DisplayName $username `
        -Name $username `
        -EmployeeID $username `
        -PasswordNeverExpires $true `
        -Path $OU `
        -Enabled $true
}
```

> **Note:** `PasswordNeverExpires $true` is used here for lab convenience only.
> In a production environment this would be `$false` to enforce the 90-day rotation GPO.

---

## The Input File (names.txt)

The script reads from a plain text file with one full name per line:

```
John Smith
Sarah Johnson
Mohammed Boudieb
Alice Martinez
David Chen
...
```

<!-- YOU WRITE HERE — Describe how you generated your names list.
Example:
"I used an online name generator to create a list of 1000 random names
and saved them to names.txt in the same folder as the script." -->

---

## How to Run the Script

**Prerequisites:**
- Must be run on the Domain Controller (or a machine with RSAT installed)
- Must be run as a Domain Administrator
- `names.txt` must be in the same directory as the script
- The `_USERS` OU must already exist in AD before running

**Steps:**
1. Open PowerShell ISE as Administrator
2. Set execution policy: `Set-ExecutionPolicy Unrestricted`
3. Navigate to the script folder: `cd C:\Scripts`
4. Run: `.\bulk-create-users.ps1`

<!-- YOU WRITE HERE — Describe what you actually saw when you ran it.
Example:
"The terminal printed each username in cyan as it was created.
The whole process took about 2-3 minutes for 1000 users.
I then opened AD Users and Computers to verify — all users appeared
in the _USERS OU correctly." -->

---

## Screenshots

**Script running in PowerShell — usernames printing in real time:**

![Script Running](./screenshots/script-running.png)

**AD Users and Computers — 1,000+ users created in the _USERS OU:**

![Users Created in AD](./screenshots/users-created-ad.png)

---

## What This Demonstrates

- **PowerShell scripting** for IT automation
- **Active Directory user provisioning** at scale
- **String manipulation** (parsing names, generating usernames)
- **Secure password handling** with `ConvertTo-SecureString`
- **Real-world IT workflow** — bulk onboarding is a standard enterprise task
