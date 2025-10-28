# 🔑 Hybrid Identity with Password Hash Sync + Password Writeback

 ### [Repro Lab Demo](https://youtu.be/7eJexJVCqJo)

## 📘 Description
This project demonstrates a hybrid identity configuration between **Active Directory Domain Services (AD DS)** and **Microsoft Entra ID** using **Entra Connect (Azure AD Connect)**. It implements **Password Hash Synchronization** for user sign-in and **Password Writeback** to synchronize password changes from Microsoft Entra ID back to on-premises AD DS, ensuring a unified credential experience.
<br />

## 🎯 Lab Objective
To establish a hybrid identity environment where on-premises users can authenticate to Microsoft 365 services using synchronized credentials, and where password resets initiated in the cloud reflect seamlessly in the on-premises directory.

## ⚙️ Key Configuration Steps
1. Create on-premises user accounts within the AD DS environment.  
2. Install **Azure AD Connect** on the domain controller.  
3. Configure **Password Hash Synchronization (PHS)** for hybrid identity sync.  
4. Enable **Password Writeback** within Azure AD Connect.  
5. Configure **Self-Service Password Reset (SSPR)** in Microsoft Entra ID for scoped users.  
6. Test password reset from the Microsoft 365 portal and confirm on-premises synchronization.

<h2>Platforms & Tools Used</h2>

- **Windows Server 2016** – hosting **Active Directory Domain Services (AD DS)** and **Entra Connect**
- **Microsoft Entra ID (Azure AD)**
- **Microsoft Entra Connect (Azure AD Connect)** – hybrid identity bridge.
- **Microsoft 365 Admin Center** – used to verify synchronized users and password reset operations.
- **Event Viewer (Windows Server 2016)** – monitored **Password Writeback** logs confirming synchronization from cloud to on-prem AD.
- **Microsoft Entra ID Audit Logs** – tracked **Self-Service Password Reset (SSPR)** activity, including registration, reset initiation, and success events.

<h2>Labs walk-through:</h2>

### 1️⃣ Install Entra Connect with **Custom Settings 
1. Install, once installation is complete > Choose **Password Hash Sync as sync method
2. Connect Entra ID by login In with Admin account → Next**
3. In connect your directories > Add Directory > Create new AD account > Provide Username and password of new account > Next
4. Domain and OU filtering > Choose to sync all directories or selected Directories:  

<p align="center">
Install Entra Connect: <br/>
<img src="https://imgur.com/9RcTRyq.png" height="80%" width="80%" alt=/>
<br />
<br />

 ### 2️⃣ Create an Authentication Strength
1. Navigate to **Authentication Strengths**  
2. Create New Authentication Strength: `AuthStrenght_PSWLess_PIM`  
3. Include:  
   - ✅ Microsoft Authenticator (Phone sign-in)  
   - ✅ Temporary Access Pass (One-time Use)  
4. Save and confirm 

Create New Authentication Strenght:  <br/>
<img src="https://imgur.com/buqnj7R.png" height="80%" width="80%" alt=/>
<br />
<br />

### 3️⃣ Create an Authentication Context
1. Go to **Protection → Conditional Access → Authentication Contexts**  
2. Create a new context named `PIM-Auth`  
3. Add description and save  
   *(Acts as a logical tag for the CA policy)*

New Authentication Context: <br/>
<img src="https://imgur.com/z3x6zvK.png" height="80%" width="80%" alt=/>
<br />
<br />

### 4️⃣ Create a Conditional Access Policy
1. Go to **Conditional Access → Policies → New Policy**  
2. Give it a Name
3. Assignments:  
   - **Users**: Target your test user or group  
   - **Target Resources**: Select *Authentication context → Choose the custom auth context created PIM-Auth*  
4. Grant Controls:  
   - Require **Authentication Strength → Choose the custom Strength Created**  
5. Enable the policy

Traget Resources:  <br/>
<img src="https://imgur.com/Il7nGzW.png" height="80%" width="80%" alt=/>
<br />
<br />

Access Control:  <br/>
<img src="https://imgur.com/ggzKtoQ.png" height="80%" width="80%" alt=/>
<br />
<br />

### 5️⃣ Configure Privileged Identity Management (PIM)
1. Navigate to **Entra ID > Privileged Identity Management > Roles > Choose the Administrator role > Add Assignments**  
2. Select **Security Administrator** (or preferred role)
3. Add a user as member or add a group.
4. Setting > Assignment Type as **Eligible**
5. Assign your test user as *eligible* for the role
6. Configure role settings for Privileged Role, On Activation, Require Microsoft Entra conditional Access Authentication context > Select the Authentication context created.
7. Select the method of justification as **Require justification on activation** 

PIM configuration:  <br/>
<img src="https://imgur.com/erhJyHR.png" height="80%" width="80%" alt=/>
<br />
<br />

PIM configuration:  <br/>
<img src="https://imgur.com/JbmY8D5.png" height="80%" width="80%" alt=/>
<br />
<br />

### 6️⃣ Test the Scenario on User's side. 
1. Sign in as the test user  
2. Go to **Entra ID → Privileged Identity Management → My Roles**  
3. Attempt to **activate** the `Security Administrator` role  
4. Observe:  
   - If signed in with password → prompt for Temporary Access Password(stronger auth)  
   - If signed in passwordlessly → activation succeeds  
5. After TAP setup, activation completes successfully
User tries to activate the Role  <br/>
<img src="https://imgur.com/MQsdjUJ.png" height="80%" width="80%" alt=/>
<br />
<br />

The user initially signed in with password + Authenticator (MFA).
1. That satisfies “multi-factor” but not “passwordless” since a password was used.
2. When the user tried to activate the PIM role, PIM invoked the authentication context.
3. The conditional access policy tied to that context checked the authentication strength requirement and found: The current session used password + MFA, not passwordless.”
4. Because of that, it prompted for one of the methods that qualify under the Authentication strength configured: Microsoft Authenticator phone sign-in or Temporary Access Pass

PIM Requirement Request  <br/>
<img src="https://imgur.com/CrJhPL6.png" height="80%" width="80%" alt=/>
<br />
<br />

Since the user didn’t yet have a TAP configured and wasn’t set up for passwordless phone sign-in, Entra ID correctly said: “You need your admin to create a Temporary Access Pass.”

PIM Requirement Request  <br/>
<img src="https://imgur.com/LGxLMVC.png" height="80%" width="80%" alt=/>
<br />
<br />

Admin creates TAP <br/>
<img src="https://imgur.com/JmID93N.png" height="80%" width="80%" alt=/>
<br />
<br />

USer Revalidates and Porvided TAP <br/>
<img src="https://imgur.com/iCIRQRA.png" height="80%" width="80%" alt=/>
<br />
<br />

Once admin created the TAP and the user retried, Entra ID accepted that TAP authentication as valid for passwordless authentication strength, allowing the PIM role activation.
Validationg Is granted <br/>
<img src="https://imgur.com/EdzD95p.png" height="80%" width="80%" alt=/>
<br />
<br />

## 💡 Key Learnings

- Password Hash Sync provides a simple and secure hybrid identity approach for most organizations.
- Password Hash Sync is reliable and easy to maintain for hybrid identity models.  
- Password Writeback keeps user credentials aligned across environments.  
- Combining both simplifies identity management while enhancing user experience.
- Password Writeback bridges modern cloud self-service features with on-premises infrastructure.  
- Ensuring proper permissions and verified UPN suffixes is crucial for successful sync.  
- Azure AD Connect Health helps monitor and maintain synchronization reliability.

--!>

