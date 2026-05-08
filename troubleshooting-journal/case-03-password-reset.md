# Case 03 — Expired Password Preventing Domain Login

**Category:** Active Directory  
**Difficulty:** Easy  
**Time to Resolve:** ~8 minutes  
**osTicket Ticket:** #1003


## Situation

A domain user (`sarah.johnson` in the HR OU) submitted a ticket reporting
she could not log in to her workstation. The error message was:

> *"Your password has expired and must be changed."*

She had been on annual leave for 3 weeks and returned to find she
could not access her machine. The 90-day password rotation GPO
had expired her password during her absence.


## Task

Reset the user's password, ensure they can log in, and force them
to set a new password themselves on first login (so the IT admin
never knows their permanent password).

---

## Action

**Step 1 - Acknowledge and triage the ticket**

Opened the osTicket (#1003). This is a standard password expiry -
no security concern, no lockout involved.
Set SLA: SEV-C (Low - single user, non-urgent).
Posted internal note: *"Password expiry confirmed. Resetting now."*

**Step 2 - Reset the password in Active Directory**

On the Domain Controller:
1. Tools → Active Directory Users and Computers
2. Found `sarah.johnson` in `_CORP → HR`
3. Right-clicked the account → **Reset Password**
4. Set a temporary password: `Welcome@123!`
5. Checked: ✅ **"User must change password at next logon"**
6. Left unchecked: ❌ "Unlock account" (not needed-no lockout)
7. Clicked OK

The Reset Password dialog appeared. I set a temporary password
and made sure 'User must change password at next logon' was checked.
This is important , it means the user sets their own permanent password
and I never have access to it after this point.
**Step 3 - Communicate the resolution securely**

Posted reply to the user in osTicket:
> *"Your password has been reset. Please log in with the temporary password
> below and you will be prompted to create a new one immediately.*
>
> *Temporary password: [communicated via separate secure channel]*
>
> *Please do not share this password with anyone. Once you log in and
> change it, only you will know your new password."*

> **Note:** The temporary password should ideally be communicated by phone
> or in-person rather than written in the ticket for security reasons.

**Step 4 - Verify**

I logged on to the Windows 10 client as sarah.johnson using the
temporary password. Windows immediately prompted 'You must change
your password before logging on the first time.' I entered a new
password and the login completed successfully.

User confirmed login success → Ticket marked **Resolved → Closed**.

---

## Result

Password reset completed in under 8 minutes. User back to work
with a new password only they know. SLA met.

**Key lesson:** Always check "User must change password at next logon."
Never leave users on a temporary password long-term , it's a security risk.
Temporary passwords should be communicated via a separate secure channel,
not left as plain text in the ticket.

---

## Commands Used (Alternative via PowerShell)

```powershell
# Reset password and force change at next logon
Set-ADAccountPassword -Identity sarah.johnson `
    -Reset `
    -NewPassword (ConvertTo-SecureString "Welcome@123!" -AsPlainText -Force)

Set-ADUser -Identity sarah.johnson -ChangePasswordAtLogon $true

# Verify
Get-ADUser sarah.johnson -Properties PasswordExpired, PasswordLastSet
```
