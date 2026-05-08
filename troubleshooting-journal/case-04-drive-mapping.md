# Case 04 - Shared Drive Not Appearing on Client After GPO

**Category:** Group Policy / File Sharing  
**Difficulty:** Medium  
**Time to Resolve:** ~20 minutes  
**osTicket Ticket:** #1004



## Situation

After configuring a GPO to automatically map a shared department folder
as drive `Z:` for all users in the Sales OU, the drive was not appearing
on the Windows 10 client (`CLIENT1`) after login.

The share existed on the server and was accessible if mapped manually.
The GPO appeared to be configured correctly in Group Policy Management.
No error messages were shown to the user — the drive simply wasn't there.



## Task

Diagnose why the GPO drive mapping was not applying to the client
and ensure `Z:` drive auto-maps correctly for all Sales users on login.

---

## Action

**Step 1 - Verify the share exists and is accessible**

On the server, confirmed the shared folder existed:
- Folder: `C:\Shares\Sales`
- Share name: `Sales`
- Share path: `\\172.16.0.1\Sales`

Tested manually on the client:
1. Opened File Explorer → This PC → Map Network Drive
2. Entered `\\172.16.0.1\Sales`
3. Connected successfully ✅

The share itself was working. The problem was the GPO not applying.

**Step 2 - Force a GPO refresh on the client**

Ran on the client:
```
gpupdate /force
```

Logged off and back on , drive still not appearing. ❌

**Step 3 - Check which GPOs are actually applying**

Ran on the client:
```
gpresult /r
```

Reviewed the output. The drive mapping GPO was **not listed** under
"Applied Group Policy Objects."

gpresult showed the Default Domain Policy was applying but my
custom drive mapping GPO was not listed at all — it showed under
'Denied GPOs' with reason: 'Inaccessible'.

**Step 4 — Identify the root cause**

I went back to Group Policy Management and found I had linked
the GPO to the _CORP OU root but not to the Sales OU specifically.
Since the users were in _CORP → Sales, and I didn't enable
inheritance, the policy wasn't reaching them.
**Step 5 - Apply the fix**

I moved the GPO link from the _CORP OU to the Sales OU directly.
Then ran gpupdate /force on the client and logged off and back on.

**Step 6 - Verify the fix**

Ran `gpupdate /force` again → logged off → logged back in.

Opened File Explorer → `Z:` drive appeared automatically. ✅
`gpresult /r` now showed the drive mapping GPO under applied policies. ✅

---

## Result

Drive mapping GPO applying correctly for all users in the Sales OU.
Root cause was [YOUR SPECIFIC CAUSE HERE].

**Key lesson:** Always run `gpresult /r` before assuming a GPO is broken.
It tells you exactly which policies are applying, which are being denied,
and why — saving significant troubleshooting time compared to guessing.

---

## GPO Configuration Used

```
Group Policy Management
└── Forest: mydomain.com
    └── Domains
        └── mydomain.com
            └── _CORP
                └── Sales              ← GPO linked HERE
                    └── [Drive Map GPO]

GPO Settings:
User Configuration
└── Preferences
    └── Windows Settings
        └── Drive Maps
            └── New → Mapped Drive
                Action: Create
                Location: \\172.16.0.1\Sales
                Drive Letter: Z
                Label: Sales Share
                ✅ Reconnect
```

---

## Commands Used

```powershell
# Force GPO refresh on client
gpupdate /force

# Check which GPOs are applying (run on client)
gpresult /r

# More detailed GPO report (generates HTML report)
gpresult /h C:\gpresult.html
```
