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
3. Configure **Password Hash Synchronization** for hybrid identity sync.  
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

### 1️⃣ Install and Configure Entra Connect (Custom Settings)

1. **Run the Entra Connect Installer** and select **Custom ** during setup.  
2. Once installation is complete, choose **Password Hash Synchronization ** as the preferred authentication method.  
3. **Connect to Entra ID (Azure AD)** by signing in with **Global Administrator credentials**, then select **Next**.  
4. Under **Connect your directories**, select **Add Directory** → choose **Create new AD DS Connector account** → provide the **username and password** of the new account → click **Next**.  
5. In **Domain and OU Filtering**, choose whether to synchronize **all directories** or **specific Organizational Units (OUs)**, depending on your sync scope → click **Next**.  
6. Under **Optional Features**, ensure **Password Hash Synchronization** is **enabled** → click **Next**.  
7. Review the configuration summary and click **Install** to begin synchronization.  
8. Once installation completes, monitor and verify that the initial synchronization has successfully finished via the **Synchronization Service Manager** or **Event Viewer logs** on the server.  

<p align="center">
Install Entra Connect: <br/>
<img src="https://imgur.com/9RcTRyq.png" height="80%" width="80%" alt=/>
<br />
<br />

OU Filtering and Optional Features:  <br/>
<img src="https://imgur.com/Fmh3Xmz.png" height="80%" width="80%" alt=/>
<br />
<br />

Configuration complete:  <br/>
<img src="https://imgur.com/sA5FrvY.png" height="80%" width="80%" alt=/>
<br />
<br />

Hybrid Setup confirmed:  <br/>
<img src="https://imgur.com/ees9uer.png" height="80%" width="80%" alt=/>
<br />
<br />

Synchronization Service:  <br/>
<img src="https://imgur.com/PuzX7l0.png" height="80%" width="80%" alt=/>
<br />
<br />

### 2️⃣ Prepare AD for Password Writeback https://learn.microsoft.com/en-us/entra/identity/authentication/tutorial-enable-sspr-writeback

1. **Enable Advanced Features in Active Directory Users and Computers:**  
   - Open **AD** → click **View** → select **Advanced Features**.  
   - Right-click the domain root → choose **Find** → locate the **Microsoft Entra Connector account** used by Entra Connectby typing Msol and click Find Now.  

2. **Grant Required Permissions to the Connector Account:**  
   - Right-click the User → select **Properties** → go to the **Security** tab → click **Advanced**.  
   - Select **Add** → click **Select a principal** → add the **MSOL** service account, Check Names.  
   - Under Type: Allow | Under **Applies to**, choose **Descendant User Objects**.  
   - In **Permissions**, Enable Reset password and Change password | In **Properties**, Enable **lockoutTime** and **pwdLastSet** → click **Apply** → **OK** to save changes.  

3. **Modify Group Policy to Allow Immediate Password Changes:**  
   - Open **Group Policy Management Console (GPMC)** → expand **Forest > Domains > Group Policy Objects**.  
   - Right-click **Default Domain Policy** → select **Edit**.  
   - Navigate to:  
     **Computer Configuration → Policies → Windows Settings → Security Settings → Account Policies → Password Policy**.  
   - Set **Minimum password age** to **0 days** to allow immediate password resets.  

### 3️⃣ Prepare Entra Connect for Password Writeback

1. Open **Entra Connect** on the synchronization server.  
2. Sign in with **Global Administrator credentials** → click **Next** to connect to Entra ID.  
3. Continue to the **Optional Features** page → select **Password Writeback** → click **Next**.  
4. Review configuration → click **Configure** to enable Password Writeback functionality → Configure

### 4️⃣ Verify On-Premises Integration Settings in Entra ID and Enable SSPR and Validate Password Writeback

1. In the **Microsoft Entra admin center**, navigate to:  
   **Users → Password Reset → On-premises integration**.  
2. Confirm that the **Integration with on-premises directories** option is **Enabled**.  
3. Under **Manage settings**, ensure that:  
   - **Enable password writeback for synced users** is turned **On**.  
   - **Allow users to unlock accounts without resetting their password** is turned **On**.  
4. In the **Microsoft Entra admin center**, navigate to: **Users → Password Reset → Properties** → enable **Self-Service Password Reset (SSPR)** for selected users or groups.
5. Configure authentication methods (e.g., mobile app, email, security questions).
6. Perform a **password change** and a **password reset** from the **Microsoft 365 portal** using a cloud-synced user account.
7. Validate that the reset event is reflected in **on-prem AD DS**, confirming that **Password Writeback** successfully synchronized the change back to the domain.  

## 💡 Key Learnings

- Password Hash Sync provides a simple and secure hybrid identity approach for most organizations.
- Password Hash Sync is reliable and easy to maintain for hybrid identity models.  
- Password Writeback keeps user credentials aligned across environments.  
- Combining both simplifies identity management while enhancing user experience.
- Password Writeback bridges modern cloud self-service features with on-premises infrastructure.  
- Ensuring proper permissions and verified UPN suffixes is crucial for successful sync.  
- Azure AD Connect Health helps monitor and maintain synchronization reliability.

--!>

