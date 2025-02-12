---
title: Co-manage internet-based devices
titleSuffix: Configuration Manager
description: Learn how to prepare your Windows 10 internet-based devices for co-management.
author: aczechowski
ms.author: aaroncz
manager: dougeby
ms.date: 03/05/2019
ms.topic: conceptual
ms.prod: configuration-manager
ms.technology: configmgr-client
ms.assetid: 101de2ba-9b4d-4890-b087-5d518a4aa624
ms.collection: M365-identity-device-management
---

# How to prepare internet-based devices for co-management

This article focuses on the second path to co-management, for new internet-based devices. This scenario is when you have new Windows 10 devices that join Azure AD and automatically enroll to Intune. You install the Configuration Manager client to reach a co-management state.  



## Windows Autopilot

For new Windows 10 devices, you can use the Autopilot service to configure the out of box experience (OOBE). This process includes joining the device to Azure AD and enrolling the device in Intune.  

For more information, see [Overview of Windows Autopilot](https://docs.microsoft.com/windows/deployment/windows-autopilot/windows-autopilot).    

To configure your devices to be automatically enroll into Intune when they join Azure AD, see [Enroll Windows devices for Microsoft Intune](https://docs.microsoft.com/intune/windows-enroll).  


### Gather information from Configuration Manager

Starting in version 1802, use Configuration Manager to collect and report the device information required by the Microsoft Store for Business and Education. This information includes the device serial number, Windows product identifier, and a hardware identifier. It's used to register the device in the Microsoft Store to support Windows Autopilot. 

1. In the Configuration Manager console, go to the **Monitoring** workspace, expand the **Reporting** node, expand **Reports**, and select the **Hardware - General** node.  

2. Run the report, **Windows Autopilot Device Information**, and view the results.  

3. In the report viewer, select the **Export** icon, and choose the **CSV (comma-delimited)** option.  

4. After saving the file, upload the data to the Microsoft Store for Business and Education.  

For more information, see [Add devices in Microsoft Store for Business and Education](https://docs.microsoft.com/microsoft-store/add-profile-to-devices#add-devices-and-apply-autopilot-deployment-profile).


### Autopilot for existing devices
<!--1358333-->

[Windows Autopilot for existing devices](https://techcommunity.microsoft.com/t5/Windows-IT-Pro-Blog/New-Windows-Autopilot-capabilities-and-expanded-partner-support/ba-p/260430) is available in Windows 10, version 1809 or later. This feature allows you to reimage and provision a Windows 7 device for [Windows Autopilot user-driven mode](https://docs.microsoft.com/windows/deployment/windows-autopilot/user-driven) using a single, native Configuration Manager task sequence. 

For more information, see [Windows Autopilot for existing devices task sequence](/sccm/osd/deploy-use/windows-autopilot-for-existing-devices).



## Install the Configuration Manager client

For internet-based devices in the second path, you need to create an app in Intune. Deploy this app to Windows 10 devices that aren't already Configuration Manager clients. 

### Get the command line from Configuration Manager

1. In the Configuration Manager console, go to the **Administration** workspace, expand **Cloud Services**, and select the **Co-management** node.  

2. Select the co-management object, and then choose **Properties** in the ribbon.  

3. On the **Enablement** tab, copy the command line. Paste it into Notepad to save for the next process.  

The following command line is an example:
`CCMSETUPCMD="CCMHOSTNAME=contoso.cloudapp.net/CCM_Proxy_MutualAuth/72186325152220500 SMSSiteCode=ABC"`

<!--1358215-->
Starting in version 1806, fewer command-line properties are now required.  

- The following command-line properties are required in all scenarios:  
    - CCMHOSTNAME  
    - SMSSITECODE  

- The following properties are required when using Azure AD for client authentication instead of PKI-based client authentication certificates:  
    - AADCLIENTAPPID  
    - AADRESOURCEURI  

- If the client roams back to the intranet, the following property is required:  
    - SMSMP  

- If using your own PKI SSL certificate and your CRL isn't published to the internet, the following parameter is required:  
    - /noCRLCheck  
    
     For more information, see [Planning for CRLs](/sccm/core/plan-design/security/plan-for-security#BKMK_PlanningForCRLs)  

Starting in version 1810, the site publishes additional Azure AD information to the cloud management gateway (CMG). An Azure AD-joined client gets this information from the CMG during the ccmsetup process, using the same tenant to which it's joined. This behavior further simplifies enrolling devices to co-management in an environment with more than one Azure AD tenant. Now the only two required ccmsetup properties are **CCMHOSTNAME** and **SMSSiteCode**.<!--3607731-->

> [!Note]
> If you're already deploying the Configuration Manager client from Intune, update the Intune app with a new command line and new MSI. <!-- SCCMDocs-pr issue 3084 -->

The following example includes all of these properties:   
`ccmsetup.exe CCMHOSTNAME=CONTOSO.CLOUDAPP.NET/CCM_Proxy_MutualAuth/72186325152220500 SMSSiteCode=ABC AADCLIENTAPPID=7506ee10-f7ec-415a-b415-cd3d58790d97 AADRESOURCEURI=https://contososerver SMSMP=https://mp1.contoso.com`

For more information, see [Client installation properties](/sccm/core/clients/deploy/about-client-installation-properties).


### Create the app in Intune

1. Go to the [Azure portal](https://portal.azure.com), and then open the Intune page.  

2. Select **Client Apps** > **Apps** > **Add**.  

3. Under **Other**, select **Line-of-business app**.  

4. Upload the **ccmsetup.msi** app package file. Find this file in the following folder on the Configuration Manager site server: `<ConfigMgr installation directory>\bin\i386`.  

    > [!Tip]  
    > When you update the site, make sure you also update this app in Intune.  

5. After the app is updated, configure the app information with the command line that you copied from Configuration Manager.  

> [!IMPORTANT]    
> If you customize this command line, make sure it isn't more than 1024 characters long. When the command line length is greater than 1024 characters, the client installation fails.


