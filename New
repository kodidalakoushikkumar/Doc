ysadminctl -adminUser itadmin -adminPassword - -secureTokenOn nan.nani -password - sudo defaults write /Library/Preferences/com.apple.loginwindow SHOWFULLNAME -bool YES sudo reboot now its working but i don't understand the above commands like whats mobile account and all

😂 Welcome to the world of macOS Enterprise. You fixed it, but the commands probably looked like black magic. Let me explain **what actually happened**.

---

# First understand how macOS thinks

Unlike Windows, macOS separates **authentication** from the **local user account**.

Imagine three layers:

```
          Active Directory
        (impressico.local)
                │
        verifies password
                │
                ▼
        Open Directory (macOS)
                │
        finds user information
                │
                ▼
      Local Mobile Account
        (/Users/nan.nani)
                │
                ▼
       Login Window + Desktop
```

Windows simply authenticates with AD and loads your profile.

macOS says

> "Okay, AD says you are real...  
> but where is your local profile?  
> Where is your home folder?  
> Do you have Secure Token?  
> Are you allowed to unlock the Mac?"

That's why it's more complicated.

---

# What is a Mobile Account?

Suppose your AD user is

```
nan.nani
```

Before login, that user **doesn't actually exist** on the Mac.

Run

```
dscl . list /Users
```

You'll only see

```
root
daemon
itadmin
...
```

No

```
nan.nani
```

because it's only in AD.

When an AD user logs in **for the first time**, macOS creates

```
/Users/nan.nani
```

and copies

```
Desktop
Documents
Downloads
Pictures
Movies
Music
```

Now the user has a local profile.

That is called a **Mobile Account**.

Think of it like

> "An offline cached version of an Active Directory user."

Exactly like Windows caching a domain profile under

```
C:\Users\username
```

---

# Command 1

```
sudo /System/Library/CoreServices/ManagedClient.app/Contents/Resources/createmobileaccount -n nan.nani
```

means

```
Create
a local profile
for AD user
nan.nani
```

instead of waiting until first login.

It creates

```
/Users/nan.nani
```

and registers the user locally.

---

# Why did it ask for Secure Token Admin?

Because on modern macOS,

creating users is a protected operation.

Apple says

> "Only an administrator that already owns a Secure Token may create another login user."

So it asked

```
SecureToken admin username:
```

because

```
itadmin
```

owns the Secure Token.

---

# What is Secure Token?

This is the confusing part.

Think of Secure Token as

```
Special Apple permission
to unlock encrypted login data
```

NOT

```
Active Directory permission
```

NOT

```
Administrator permission
```

A Secure Token is simply a cryptographic key Apple gives to login users.

Without it,

you cannot

- unlock FileVault
- sometimes create users
- sometimes log in after password changes
- manage APFS encrypted volumes

---

Imagine

```
Administrator
        │
        ▼
Can install software

Secure Token
        │
        ▼
Can unlock encrypted user data
```

They are **different things**.

---

# Command 2

```
sudo sysadminctl \
-adminUser itadmin \
-adminPassword - \
-secureTokenOn nan.nani \
-password -
```

means

```
Use
itadmin's Secure Token

to give

nan.nani

a Secure Token
```

It also synchronizes the Secure Token credentials with the current password.

That's why your company's note says

> Run this after every AD password change.

---

# Why password changes break

Suppose yesterday

```
AD password

abc123
```

Mac stores

```
Secure Token
encrypted using abc123
```

Today IT changes AD password

```
xyz789
```

AD now says

```
correct password

xyz789
```

But the Secure Token is still encrypted using

```
abc123
```

GUI login fails because it cannot unlock the local credentials.

`sysadminctl` updates that relationship.

---

# Command 3

```
sudo defaults write /Library/Preferences/com.apple.loginwindow SHOWFULLNAME -bool YES
```

This simply changes

From

```
👤 itadmin
```

to

```
Username:
Password:
```

Nothing security-related.

It edits

```
/Library/Preferences/com.apple.loginwindow
```

adding

```
SHOWFULLNAME = YES
```

---

# Why `su` worked but GUI didn't

This confused you the most.

```
su nan.nani
```

does

```
Ask Active Directory

Is password correct?

↓

AD says YES

↓

Open shell
```

Done.

No desktop.

No Finder.

No Secure Token.

No home folder creation.

No FileVault.

No LoginWindow.

---

GUI login does

```
LoginWindow

↓

Authenticate with AD

↓

Create mobile account

↓

Check Secure Token

↓

Create home folder

↓

Load LaunchServices

↓

Start Dock

↓

Start Finder

↓

Desktop appears
```

There are **many more steps**, so more things can fail.

---

# What actually happened in your case

```
Mac joined AD
        ✅

↓

AD lookup worked
(id nan.nani)
        ✅

↓

Password worked
(su nan.nani)
        ✅

↓

GUI login
        ❌

↓

No mobile account
        ❌

↓

Secure Token not synced
        ❌

↓

You created mobile account
        ✅

↓

You granted/synchronized Secure Token
        ✅

↓

LoginWindow accepted login
        ✅

↓

Desktop loaded
        ✅
```

---

## Why this felt so different from Windows

On Windows, domain login, profile creation, and desktop startup are tightly integrated. On macOS, Apple treats them as separate pieces: **directory authentication**, **local mobile account**, **Secure Token**, and **the graphical login session**. That's why you can have a situation where `su` succeeds but the graphical login still fails.

Now that you've been through this, you've actually debugged one of the more common enterprise macOS issues. It's a good example of how macOS integrates with Active Directory while still following Apple's own account and security model.
