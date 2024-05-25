---
title: "BitLocker Reporting in Intune"
date: 2024-04-24 08:00:00 -0400
categories: [blue-team]
tags: [blue-team,intune]
image:
    path: /assets/img/posts/bitlockerentraid/thumbnail.png
--- 

<style>
  .body {
    display:block;
  }
</style>

## Introduction

Reporting BitLocker metrics is crucial for both compliance and security purposes in an organization’s Information Technolgy (IT) infrastructure. BitLocker, as a disk encryption feature provided by Microsoft, plays a vital role in safeguarding sensitive data stored on devices. By monitoring BitLocker metrics, organizations can ensure compliance with industry regulations and internal policies regarding data protection and privacy.

From a compliance perspective, reporting BitLocker metrics provides evidence of encryption implementation and compliance with regulatory requirements such as GDPR or HIPAA. These regulations mandate the encryption of sensitive data to prevent unauthorized access and mitigate the risk of data breaches. Regular reporting helps demonstrate due diligence in meeting these obligations, which is essential for avoiding penalties and maintaining trust with stakeholders.

On the security front, monitoring BitLocker metrics offers insights into the encryption status of devices across the organization’s network. It allows administrators to identify potential vulnerabilities or gaps in security measures, such as devices lacking encryption or those with outdated encryption configurations. Timely detection of such issues enables prompt remediation actions, such as enforcing encryption policies, updating encryption settings, or addressing compliance deviations.

Overall, reporting BitLocker metrics enhances the organization’s data protection posture, strengthens security measures, and ensures alignment with regulatory requirements, thereby minimizing the risk of data breaches and preserving the integrity and confidentiality of sensitive information.

## Guide

I have provided the detailed steps and scripts on how to get the reporting for these metrics in Intune. In order to perform these operations in the Intune console, you must have the role of Intune Administrator.

Step 1) Login to the Intune portal by navigating to https://intune.microsoft.com/

Step 2) On the left side, click on the Devices > Manage devices > Scripts and remediations

<p class="body"><img class="body" src="./assets/img/posts/bitlockerentraid/image01.png"></p>

Step 3) Under the Remediations tab, click Create.

<p><img class="body" src="./assets/img/posts/bitlockerentraid/image02.png"></p>

Step 4) Add a name and description to the remediation and click Next.

<p><img class="body" src="./assets/img/posts/bitlockerentraid/image03.png"></p>

Step 5) Upload the detection script file that I have provided in my GitHub below as a .ps1 file.

IntuneScripts/GetBitLockerVolumeStatus.ps1 at main · ultraakicks/IntuneScripts (github.com)

<p><img class="body" src="./assets/img/posts/bitlockerentraid/image04.png"></p>


Step 6) On the bottom settings you will want the first two to be set the No and the third to be set to Yes as shown then click Next.

<p><img class="body" src="./assets/img/posts/bitlockerentraid/image05.png"></p>

Step 7) Use the scope tags or assignments sections to scope to the devices you want to target, then review and create the detection script.

NOTE: The script is scheduled to run once every hour when the device checks in.

Step 8) Once the script is run on the device (s), you can navigate to the script > Monitor > Device Status.

<p><img class="body" src="./assets/img/posts/bitlockerentraid/image06.png"></p>

Step 9) Click on Columns and add the Column Pre-remediation detection output and click Apply.

<p><img class="body" src="./assets/img/posts/bitlockerentraid/image07.png"></p>

Step 10) Now on the page, you will see the output from the script for each device scoped to the detection script.

<p><img class="body" src="./assets/img/posts/bitlockerentraid/image08.png"></p>

By clicking Review you can see the full output

<p><img class="body" src="./assets/img/posts/bitlockerentraid/image09.png"></p>

You can also Export this to Excel for further analysis.

<p><img class="body" src="./assets/img/posts/bitlockerentraid/image10.png"></p>
## Conclusion

The guide I have provided comprehensively lays out each step required to enable efficient reporting on crucial metrics for BitLocker within Intune. By following the outlined procedures, Intune Administrators can effectively setup a detection script for BitLocker reporting for Security and Compliance.