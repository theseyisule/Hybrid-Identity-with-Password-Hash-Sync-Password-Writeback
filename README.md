# 🔑 Hybrid Identity with Password Hash Sync + Password Writeback

 ### [Repro Lab Demo](https://youtu.be/7eJexJVCqJo)

## 📘 Description
This project demonstrates a hybrid identity configuration between **Active Directory Domain Services (AD DS)** and **Microsoft Entra ID** using **Entra Connect (Azure AD Connect)**. It implements **Password Hash Synchronization** for user sign-in and **Password Writeback** to synchronize password changes from Microsoft Entra ID back to on-premises AD DS, ensuring a unified credential experience.
<br />

## 🎯 Lab Objective
To configure a Conditional Access policy that enforces **passwordless authentication (via Microsoft Authenticator or Temporary Access Pass)** when a user attempts to **activate a privileged role through PIM**, ensuring sensitive operations require stronger authentication.

## ⚙️ Key Configuration Steps
1. **Create an Authentication Strength**  
   - Include **Microsoft Authenticator phone sign-in** and **Temporary Access Pass (TAP)** as valid passwordless methods.

2. **Create an Authentication Context**  
   - Define a tag (e.g., `pim-auth-context`) to identify when stronger authentication should be applied.

3. **Configure a Conditional Access Policy**  
   - Target the Authentication Context created above.  
   - Apply the **Authentication Strength** requiring passwordless methods.  
   - Enforce this policy when users access or activate PIM-protected roles.

4. **Set Up Privileged Identity Management (PIM)**  
   - Assign a user as eligible for a privileged role (e.g., *Security Administrator*).  
   - Require activation with justification and apply the step-up Conditional Access policy.  
   - During activation, the policy triggers the passwordless requirement (TAP or Authenticator).


<h2>Platforms & Tools Used</h2>

| Category | Portal / Tool | Purpose |
|-----------|----------------|----------|
| **Identity & Access** | Microsoft Entra ID Portal| Configure Conditional Access, PIM, Authentication Methods, Authentication Contexts and Authentication Strengths |
| **Administration** | Microsoft 365 Admin Center| Manage users, licenses, and global tenant configuration |
| **ID Governance** | Entra ID > ID Governance > Privileged Identity Management| Assign and activate eligible roles for privileged access testing |
| **Device Access** | Mobile Device with Microsoft Authenticator App | Enable passwordless sign-in and approve multi-factor authentication requests |
| **Temporary Access Pass (TAP)** | Entra ID > Authentication Methods | Generate and use TAP for passwordless testing |


<h2>Labs walk-through:</h2>
### 1️⃣ Configure Authentication Methods
1. Go to **Entra ID → Protection → Authentication methods**  
2. Enable:  
   - **Microsoft Authenticator (Passwordless sign-in)**  
   - **Temporary Access Pass (TAP)**  
3. Target to all users or a test group

<p align="center">
Authentication Method: <br/>
<img src="https://imgur.com/bY42WaD.png" height="80%" width="80%" alt=/>
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

## 📈 Result & Behavior
> When users activate a privileged role, Conditional Access evaluates the **Authentication Context (`PIM-Auth`)**.  
>  
> The CA policy enforces the **Passwordless_Strength**, requiring either passwordless sign-in or TAP.  
>  
> If initial sign-in used a password, Entra ID steps up the session to meet the passwordless requirement before role activation.  
>  
> The result is a secure, Zero Trust aligned design with **adaptive authentication** and **role-based protection**.

---

## 💡 Key Learnings

| Area | Learning Outcome |
|------|------------------|
| **Conditional Access** | Designed and applied a CA policy linked to authentication context and custom strength |
| **Authentication Strengths** | Defined passwordless and TAP methods to enforce strong MFA |
| **PIM Integration** | Implemented step-up authentication for privileged role activation |
| **Zero Trust Design** | Achieved context-aware access and strong identity assurance |

--!>

