---
title: "Device Control with Microsoft Defender for Endpoint"
date: 2023-02-21 08:00:00 -0400
categories: [blue-team]
tags: [blue-team,mde]
image:
    path: /assets/img/posts/mdedevicecontrol/thumbnail.png
--- 

<style>
  .body {
    display:block;
  }
</style>

## Introduction

Whether you are a security engineer, architect, or even an analyst, it will be an organizational requirement that USB ports on endpoints be locked down. This will allow for better control over covered and/or confidential data as well as prevent malicious files from getting on endpoints. There are many solutions out there that can do this, however you may be stuck with finding a solution with your already purchased Microsoft licensing. In this article, I will explain how to deploy device control enterprise-wide using Microsoft Intune.

## Licensing Requirements

You will need to be licensed for Microsoft Intune.

## Pre-Requirements

You will need to have a discussion with your Security team and your executive leadership on policies and procedures for what drives are approved and the actions to take (read-only or full block).

## Policy

The policy in this post will be configured to only allow read access for unapproved drives, however approved drives will have standard read, write, and execute access. The policy can be configured to fully block unapproved drives as well, however there will likely be workflow issues during initial deployment enterprise wide.

## Deployment Process

We will be using Microsoft OMA-URIs to target a Policy CSP via Intune in order deploy device control since the granularity is not built into the Microsoft endpoint Manger (MEM) yet natively. This will be done by creating a configuration profile with 4 OMA-URI settings. Below are the following steps to setup the configuration profile for the policy:

<b>Step 1)</b> Login to Microsoft Endpoint Manager (MEM) at https://intune.microsoft.com

<b>Step 2)</b> On the left bar, go to Devices > Configuration Profiles

<p class="body"><img class="body" src="./assets/img/posts/mdedevicecontrol/image01.png" alt=""></p>

<b>Step 3)</b> Click on + Create Profile

<p class="body"><img class="body" src="./assets/img/posts/mdedevicecontrol/image02.png" alt=""></p>

<b>Step 4)</b> Select Windows 10 and later as the platform, Templates as the profile type, and under the template name select Custom then click Create

<p class="body"><img class="body" src="./assets/img/posts/mdedevicecontrol/image03.png" alt=""></p>

<b>Step 5)</b> Give the profile a name and a description

<p class="body"><img class="body" src="./assets/img/posts/mdedevicecontrol/image04.png" alt=""></p>

<b>Step 6)</b> Click Add to create our first OMA-URI and here are the following, once filled out click Save. This OMA-URI will enable the ability to have removable storage access control.

<b>Name:</b> Enable RSAC<br>
<b>Description:</b> Enable Removable Storage Access Control<br>
<b>OMA-URI:</b> ./Vendor/MSFT/Defender/Configuration/DeviceControlEnabled<br>
<b>Data type:</b> Integer<br>
<b>Value:</b> 1<br>

<p class="body"><img class="body" src="./assets/img/posts/mdedevicecontrol/image05.png" alt=""></p>

<b>Step 7)</b> Click Add to create our second OMA-URI and here are the following. You will need to copy and paste the XML, save the file locally and import the XML into the portal. Once completed click Save. This will match all USB related to removeable media, CD-ROMs, and WPD devices.

<b>Name:</b> Any Removeable Storage Group <br>
<b>Description:</b> Includes all removeable media, CD-ROMs, and WPD devices<br>
<b>OMA-URI:</b> ./Vendor/MSFT/Defender/Configuration/DeviceControl/PolicyGroups/%7b9b28fae8-72f7-4267-a1a5-685f747a7146%7d/GroupData<br>
<b>Data type:</b> String (XML File)<br>
<b>Custom XML:</b>

```xml
<Group Id="{9b28fae8-72f7-4267-a1a5-685f747a7146}">
    <MatchType>MatchAny</MatchType>
    <DescriptorIdList>
        <PrimaryId>RemovableMediaDevices</PrimaryId>
        <PrimaryId>CdRomDevices</PrimaryId>
        <PrimaryId>WpdDevices</PrimaryId>
    </DescriptorIdList>
</Group>
```

<p class="body"><img class="body" src="./assets/img/posts/mdedevicecontrol/image06.png" alt=""></p>

<b>Step 8)</b> Click Add to create our third OMA-URI and here are the following. You will need to copy and paste the XML, save the file locally and import the XML into the portal. Once completed click Save.

<b>Note:</b> Under the DescriptorIDList, you will add all approved USB devices that can bypass this policy. Please reference this Microsoft documentation on what is accepted here. In my case, we are approving all Apricorn products based on the Vendor ID (VID) since they are Self Encrypting Drives (SEDs). Below is the Microsoft documentation on how parameters for various other device identifiers.

<a href="https://learn.microsoft.com/en-us/microsoft-365/security/defender-endpoint/device-control-removable-storage-access-control?view=o365-worldwide">Microsoft Defender for Endpoint Knowledge Base</a>

<b>Name:</b> Approved USBs
<b>Description:</b> Contains a list of all approved USB drives
<b>OMA-URI:</b> ./Vendor/MSFT/Defender/Configuration/DeviceControl/PolicyGroups/%7b8a1f1742-ce62-41a1-826b-64738580566a%7d/GroupData
<b>Data type:</b> String (XML File)
<b>Custom XML:</b>

```xml
<Group Id="{8a1f1742-ce62-41a1-826b-64738580566a}">
   <!– Approved USBs Group –>
<MatchType>MatchAny</MatchType>
    <DescriptorIdList>
        <!– Apricorn –>
        <VID_PID>0984_</VID_PID>
    </DescriptorIdList>
</Group>
```

<p class="body"><img class="body" src="./assets/img/posts/mdedevicecontrol/image07.png" alt=""></p>

<b>Step 9)</b> Click Add to create our fourth OMA-URI and here are the following. You will need to copy and paste the XML, save the file locally and import the XML into the portal. Once completed click Save. You can modify the Access Policy rule by following the guidance in this Microsoft documentation:

<a href="https://learn.microsoft.com/en-us/microsoft-365/security/defender-endpoint/device-control-removable-storage-access-control?view=o365-worldwide">Microsoft Defender for Endpoint Knowledge Base</a>

<b>Name:</b> Read Only Policy Rule
<b>Description:</b> Policy to allow only read from unapproved USBs
<b>OMA-URI:</b> ./Vendor/MSFT/Defender/Configuration/DeviceControl/PolicyRules/%7b4e390fc0-7db7-428d-8129-5ee401b84a2d%7d/RuleData
<b>Data type:</b> String (XML File)
<b>Custom XML:</b>

```xml
<PolicyRule Id="{4e390fc0-7db7-428d-8129-5ee401b84a2d}">
    <Name>Block Write and Execute Access to Unapproved USBs</Name>
    <IncludedIdList>
        <GroupId>{9b28fae8-72f7-4267-a1a5-685f747a7146}</GroupId>
    </IncludedIdList>
    <ExcludedIdList>
        <GroupId>{8a1f1742-ce62-41a1-826b-64738580566a}</GroupId>
    </ExcludedIdList>
    <Entry Id=”{f8ddbbc5-8855-4776-a9f4-ee58c3a21414}”>
        <Type>Deny</Type>
        <Options>0</Options>
        <AccessMask>6</AccessMask>
    </Entry>
    <Entry Id=”{07e22eac-8b01-4778-a567-a8fa6ce18a0c}”>
        <Type>AuditDenied</Type>
        <Options>3</Options>
        <AccessMask>6</AccessMask>
    </Entry>
</PolicyRule>
```

<p class="body"><img class="body" src="./assets/img/posts/mdedevicecontrol/image08.png" alt=""></p>

<b>Step 10)</b> Add an Azure AD test group of machines that you want to deploy this policy to first then click Next (Note: mine is set to all devices in a test Azure tenant)

<p class="body"><img class="body" src="./assets/img/posts/mdedevicecontrol/image09.png" alt=""></p>

<b>Step 11)</b> Skip the applicability rules by clicking Next

<b>Step 12)</b> Review the configuration profile and then click Create

## Additional Comments

If you decide to change the GUIDs with the policy and rule sets, you can use the PowerShell command **[guid]::NewGuid()** to do so. The GUIDs in the OMA-URI's **Any Removeable Storage Group** and **Approved USBs** must be mapped to the policy **Read Only Policy Rule** correctly for the IncludedIDList and ExcludedIDList.

## Additional Microsoft Documentation References:

* <a href="https://learn.microsoft.com/en-us/microsoft-365/security/defender-endpoint/mde-device-control-device-installation?view=o365-worldwide">Microsoft Defender for Endpoint Device Control Installation</a>

* <a href="https://learn.microsoft.com/en-us/microsoft-365/security/defender-endpoint/device-control-removable-storage-access-control?view=o365-worldwide">Microsoft Defender for Endpoint Device Control</a>

* <a href="https://learn.microsoft.com/en-us/microsoft-365/security/defender-endpoint/deploy-manage-removable-storage-intune?view=o365-worldwide#deploy-removable-storage-access-control-by-using-intune-oma-uri">Microsoft Intune OMA URI</a>