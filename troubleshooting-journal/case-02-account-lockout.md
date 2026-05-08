# Case 02 - Account Lockout: Diagnosis and Unlock

**Category:** Active Directory  
**Difficulty:** Easy  
**Time to Resolve:** ~5 minutes  
**osTicket Ticket:** #1002

---

## Situation

A domain user (`john.smith` in the Sales OU) submitted a ticket through
osTicket reporting they could not log in to their workstation.
The error message displayed was:

> *"Your account has been locked. Please contact your system administrator."*

The user confirmed they had been trying to remember their password
and had attempted to log in several times before giving up.



## Task

Verify the account lockout, identify the cause, unlock the account,
and communicate the resolution to the user (all documented within the ticket).



## Action

**Step 1 - Acknowledge the ticket**

Opened the osTicket and posted an internal note:
*"Account lockout suspected. Investigating in Active Directory."*
Changed ticket status to: **In Progress**

**Step 2 - Locate the user in Active Directory**

On the Domain Controller:
1. Tools → Active Directory Users and Computers
2. Searched for `john.smith` (Action → Find)
3. Located the account in `_CORP → Sales`

When I opened the account properties I could see the account was
grayed out with a small lock icon. On the Account tab, the checkbox
'Unlock account' was checked and grayed out, confirming the lockout.

**Step 3 - Confirm lockout details**

Right-clicked the user → Properties → **Account tab**

Observed: ✅ "Unlock account" checkbox was visible and checked
(this appears automatically when an account is locked)

**Step 4 - Unlock the account**

Unchecked the "Unlock account" checkbox → clicked **Apply → OK**

Account unlocked. No password reset required in this case since
the user remembered their password — they just had too many failed attempts.

**Step 5 - Verify and close the ticket**

Posted reply to the user in osTicket:
> *"Your account has been unlocked. You should now be able to log in normally.
> Your password has not been changed. If you continue to have issues,
> please reply to this ticket."*

User confirmed they could log in → Ticket marked **Resolved → Closed**.

---

## Result

Account unlocked within 5 minutes of ticket submission.
SLA target (SEV-C: 8 hours) met comfortably.

**Key lesson:** The domain's lockout policy (3 failed attempts → 30 min lockout)
is working as intended , this is a security feature, not a bug.
In a real environment, repeated lockouts for the same user might indicate
a forgotten password, a compromised account, or a saved credential
somewhere that's sending the wrong password automatically.

---

## Commands Used (Alternative Method via PowerShell)

```powershell
# Check if account is locked
Get-ADUser john.smith -Properties LockedOut | Select LockedOut

# Unlock the account
Unlock-ADAccount -Identity john.smith

# Verify it's unlocked
Get-ADUser john.smith -Properties LockedOut | Select LockedOut
```

---

## Prevention Note

Reminded the user in the ticket that the domain policy locks accounts
after 3 failed attempts. Suggested they use the "I forgot my PIN" option
on the Windows login screen to avoid repeated failures in the future.
