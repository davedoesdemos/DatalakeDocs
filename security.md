# Security

**produced by Dave Lusty**

## Introduction

This document aims to cover some security considerations for cloud data environments. This is not intended as a comprehensive guide but rather as an explainer to several concepts.

## Access Controls

### Who Has Access?

developer, system etc.

### Types of access

logon, run program, read data, write data

## System Security

<table>
<tr>
<td width="70%">One of the concepts which is very important to understand is system security. Your laptop or mobile phone is a good example here. You know that your device is secure enough because you know where it is, who has access, and that there is a local security password. Even if you lost your device, it's unlikely to be a problem because the person finding it wouldn't know your password to access it. The same is true of servers in your compute environment. You provision them with known images, you configure them with access controls, add firewall and network controls, and monitor their logs. This makes those servers a trusted place in your environment, and by controlling who can access them, you can control who has access to the contents. </td>
<td width="30%"><img src="images/systemSecurity.png" /></td>
</tr>
</table>

Back to your device now, you can log in to cloud services, banks, and other systems within your account. Remembering those logins means that you can use considerably more secure passwords or other access controls than if you had to remember them. Face ID, fingerprint ID or a password/PIN can then be used to verify you before using these passwords, but you only need your system password. Servers can use this same technique for security. The operations team installing the server can implement an ID on the server during initial install, as part of the system imaging, or at some later time. We trust this process because the operations team or script is a known action which itself is controlled. The process is giving the server an ID which it can use to interact with the outside world, but this ID is safe on the server.

## Code Security

Code can be problematic, because it's easy to copy and store. For this reason it is considered bad practice to store passwords within your code. Instead, you should use some kind of password safe such as Azure Key Vault to store the credentials on your behalf. This has multiple benefits. Firstly, the developer/data engineer/data scientist writing code might not personally have access to the thing being protected. They are just writing code which will have access, and it's important to understand the difference. When using Key Vault, the entity running the code will connect to the vault and retrieve credentials at run time. This also means that you can use different vaults for different environments (production vs. test) to access different resources. The entity running the code could be a server as explained in System Security above. This ensures that the credentials to important resources are not stored in code, and that the system gains access when necessary in a secure way. To revoke access we then just need to remove the access key for the system to the Key Vault, or to remove that system entirely.

<table>
<tr>
<td width="50%">This diagram shows the security boundary of the operations team, allowing them access to generate credentials for storage and place them in the Key Vault. Operations do not have access to the storage, however, so they cannot see the data. The developer has a security boundary allowing them access to the code and the code repository. They have no access to the Key Vault or to the storage, although they may in some instances be granted access to a system which does have acess to that storage. The developer would usually deploy using automation such as Azure DevOps, and therefore would not have access to the server (which may also be a cluster or other system, running the code). This is especially true of production environments.</td>
<td width="50%"><img src="images/codeSecurity.png" /></td>
</tr>
</table>