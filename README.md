# Windows Server 2022 SMB File Sharing with Ubuntu

## Project Overview

In this homelab, I configured a centralized SMB (Server Message Block) file share using Windows Server 2022 and accessed it from an Ubuntu virtual machine.

The Windows Server also functions as an Active Directory Domain Controller for `changserver.com`.

The objective was to demonstrate how Windows and Linux systems can communicate across a virtual network and access shared resources using SMB.

I successfully created a shared folder on Windows Server, connected to it from Ubuntu, and verified file visibility across both operating systems.

## Technologies Used

- Oracle VirtualBox
- Windows Server 2022
- Ubuntu Linux
- Active Directory Domain Services (AD DS)
- SMB (Server Message Block)
- NTFS File Permissions
- Windows Server Manager
- GNOME Files
- TCP/IP Networking

## Network Environment

| Component | Configuration |
|---|---|
| Hypervisor | Oracle VirtualBox |
| Server OS | Windows Server 2022 |
| Client OS | Ubuntu Linux |
| Active Directory Domain | changserver.com |
| Windows Server IP | 10.0.2.5 |
| Domain Controller | changserver.changserver.com |
| Shared Folder Name | testing |
| Local Folder Path | C:\Shares\testing |
| SMB Network Path | \\Changserver\testing |
| SMB Protocol Port | TCP 445 |

## Prerequisites

Before starting this project, I completed the following:

1. Installed Windows Server 2022 in VirtualBox.
2. Installed and configured Active Directory Domain Services.
3. Created the `changserver.com` domain.
4. Created an Active Directory user named `testguy`.
5. Installed Ubuntu Linux in VirtualBox.
6. Joined Ubuntu to the Windows Server Active Directory domain.
7. Configured networking to allow communication between the virtual machines.
8. Verified connectivity and domain discovery.

**Related Project:** [Ubuntu Join Windows Server 2022 Active Directory](https://github.com/ChangITTech/UbtunuJoinWindowServer2022)

---

# Part 1: Creating the Shared Folder

## Step 1: Open Windows Server Manager

Logged into Windows Server 2022 using an administrative account.

Opened:

`Server Manager → File and Storage Services → Shares`

This section allows administrators to create, manage, and configure network file shares.

**Screenshot:**

![Windows Server Manager Shares](images/step1-server-manager.png)

## Step 2: Launch the New Share Wizard

In the Shares section:

1. Clicked **TASKS**.
2. Selected **New Share**.
3. Opened the New Share Wizard.

Selected:

`SMB Share - Quick`

This profile provides a straightforward method for configuring SMB file sharing.

**Screenshot:**

![SMB Share Quick](images/step2-smb-share-quick.png)

## Step 3: Select the Share Location

Selected the Windows Server named `Changserver`.

Chose the `C:` volume as the storage location.

Windows Server would create the shared directory under the Shares folder.

**Screenshot:**

![Share Location](images/step3-share-location.png)

## Step 4: Configure the Share Name

Entered the following configuration:

**Share Name:**

`testing`

**Local Folder Path:**

`C:\Shares\testing`

**Remote Network Path:**

`\\Changserver\testing`

The local path identifies where files are stored on Windows Server.

The network path allows other computers to access the shared folder through SMB.

**Screenshot:**

![Share Name](images/step4-share-name.png)

## Step 5: Configure Share Settings

Reviewed the available SMB settings:

- Access-based enumeration
- Share caching
- BranchCache
- SMB encryption

For this lab, I retained the default settings.

Share caching remained enabled.

Access-based enumeration and SMB encryption were not enabled in the wizard.

**Screenshot:**

![Share Settings](images/step5-share-settings.png)

---

# Part 2: Configuring File Permissions

## Step 6: Review NTFS Permissions

Opened the Advanced Security Settings window for:

`C:\Shares\testing`

Reviewed the existing NTFS permission entries:

| Security Principal | Permission |
|---|---|
| SYSTEM | Full Control |
| Administrators | Full Control |
| CREATOR OWNER | Full Control on applicable child objects |

NTFS permissions determine which users and groups can access, modify, or manage files stored on the Windows Server.

**Screenshot:**

![NTFS Permissions](images/step6-ntfs-permissions.png)

## Step 7: Create the SMB Share

Completed the New Share Wizard.

Windows Server displayed the following successful operations:

- Create SMB share — Completed
- Set SMB permissions — Completed

This confirmed that the `testing` share was successfully created.

**Screenshot:**

![Share Created](images/step7-share-created.png)

---

# Part 3: Active Directory User Configuration

## Step 8: Open Active Directory Users and Computers

Navigated to:

`Server Manager → Tools → Active Directory Users and Computers`

Located the domain:

`changserver.com`

Opened the properties for the existing domain account:

`testguy@changserver.com`

Selected the **Member Of** tab.

**Screenshot:**

![Active Directory User](images/step8-ad-user.png)

## Step 9: Configure Group Membership

For administrative group membership testing, added the `testguy` account to the Domain Admins group.

Steps:

1. Opened the **Member Of** tab.
2. Clicked **Add**.
3. Entered `Domain Admins`.
4. Clicked **Check Names**.
5. Confirmed the group.
6. Applied the changes.

The account then displayed membership in:

- Domain Users
- Domain Admins

**Security Note:** Domain Admins provides extensive administrative privileges within Active Directory. It is not required for ordinary SMB file access. In a production environment, standard users should receive only the folder permissions necessary for their work.

**Screenshot:**

![AD Group Membership](images/step9-group-membership.png)

## Step 10: Verify Domain Group Membership from Ubuntu

Opened the Ubuntu terminal and executed:

```bash
id testguy@changserver.com
```

The command returned the user's UID, primary group, and supplementary Active Directory groups.

The results included Domain Users and Domain Admins.

This confirmed that Ubuntu could retrieve the updated Active Directory group membership through SSSD.

**Screenshot:**

![Ubuntu AD Group Verification](images/step10-ubuntu-ad-groups.png)

---

# Part 4: Accessing the Windows Share from Ubuntu

## Step 11: Open Ubuntu Files

Logged into the Ubuntu virtual machine.

Opened the GNOME Files application.

Selected **Network** from the left navigation panel.

**Screenshot:**

![Ubuntu Files](images/step11-ubuntu-files.png)

## Step 12: Connect to the Windows SMB Share

Used the following SMB address:

```text
smb://10.0.2.5/testing
```

Clicked **Connect**.

This address specifies:

- `smb://` — SMB file-sharing protocol
- `10.0.2.5` — Windows Server IP address
- `testing` — Shared folder name

The Ubuntu file manager connected to the Windows Server shared folder.

When required, SMB connections can authenticate using authorized Active Directory credentials.

**Screenshot:**

![Connect to SMB Share](images/step12-smb-connect.png)

---

# Part 5: Testing File Sharing

## Step 13: Create a Test File on Windows Server

Returned to Windows Server 2022.

Opened the shared folder:

`\\Changserver\testing`

Created a new text document named:

`can you see me.txt`

This file was used to verify that Ubuntu could access content stored on the Windows Server.

**Screenshot:**

![Windows Test File](images/step13-windows-test-file.png)

## Step 14: Verify File Visibility on Ubuntu

Returned to Ubuntu.

Opened the connected SMB share.

Confirmed that the following file appeared:

`can you see me.txt`

**Result: SUCCESS**

The file created on Windows Server was visible from Ubuntu through the SMB network share.

This demonstrated successful cross-platform file access.

**Screenshot:**

![Ubuntu File Verification](images/step14-ubuntu-file-verification.png)

---

# Part 6: Troubleshooting and Verification

## Network Connectivity

To verify basic network communication, I used the following command from Ubuntu:

```bash
ping 10.0.2.5
```

A successful response confirms IP connectivity between the Ubuntu VM and Windows Server.

## Active Directory Connectivity

To verify domain connectivity through SSSD:

```bash
sudo sssctl domain-status changserver.com
```

An online domain status indicates that SSSD can communicate with the Active Directory domain.

## DNS Verification

To verify Active Directory service discovery:

```bash
nslookup -type=SRV _ldap._tcp.dc._msdcs.changserver.com
```

This checks the DNS SRV records used to locate Active Directory domain controllers.

## SMB Troubleshooting Considerations

If Ubuntu cannot access the shared folder, check:

1. Whether Windows Server is powered on.
2. Whether both virtual machines can communicate over the network.
3. Whether the SMB share exists.
4. Whether Windows Firewall allows SMB traffic on TCP port 445.
5. Whether the account has appropriate share and NTFS permissions.
6. Whether the supplied SMB credentials are correct.

**Important:** Successful ping responses alone do not prove that SMB access is working. SMB connectivity and permissions must be verified separately.

---

# Project Results

| Test | Result |
|---|---|
| Windows Server SMB share creation | Passed |
| Active Directory group membership update | Passed |
| Ubuntu AD group lookup | Passed |
| Ubuntu connection to SMB share | Passed |
| Windows-to-Ubuntu file visibility | Passed |
| Ubuntu-to-Windows write access | Not yet tested |
| Second Ubuntu VM share access | Not yet tested |
| Standard-user permission restrictions | Not yet tested |

## Skills Demonstrated

**Windows Server Administration**
- Configured an SMB file share using Windows Server Manager.
- Reviewed NTFS and SMB security settings.
- Managed Active Directory user group membership.

**Linux Administration**
- Used Ubuntu to access Windows network resources.
- Verified Active Directory user and group information using Linux commands.
- Worked with SMB shares through GNOME Files.

**Networking**
- Applied TCP/IP networking concepts.
- Verified communication between virtual machines.
- Used SMB to access shared network resources.
- Practiced DNS and Active Directory service discovery.

**IT Infrastructure**
- Configured centralized file storage.
- Integrated Windows and Linux systems.
- Practiced basic network resource troubleshooting.
- Documented infrastructure configuration and testing results.

## Key Takeaways

This project helped me understand how Windows Server can provide centralized file-sharing services to Linux clients.

I learned that:

1. SMB enables cross-platform network file access.
2. Active Directory can provide centralized user and group identities.
3. SMB share permissions and NTFS permissions jointly determine effective file access.
4. Network connectivity, authentication, and file permissions are separate components of troubleshooting.
5. Linux clients can access Windows Server resources without running Windows.

## Future Improvements

- Connect a second Ubuntu virtual machine to the same SMB share.
- Test creating and modifying files directly from Ubuntu.
- Configure a dedicated Active Directory security group for shared-folder access.
- Apply least-privilege permissions instead of relying on Domain Admins.
- Test access-denied scenarios using a standard domain account.
- Compare read-only and read/write access.

---

## Conclusion

Successfully configured an SMB shared folder on Windows Server 2022 and accessed it from Ubuntu Linux in a VirtualBox homelab.

Verified that a text file created on Windows Server was visible from Ubuntu through the shared network folder.

This project provided hands-on experience with Windows Server administration, Active Directory integration, SMB file sharing, Linux interoperability, and network troubleshooting.
