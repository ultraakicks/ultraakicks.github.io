---
title: "Employee Hire Date in Entra ID"
date: 2024-02-19 08:00:00 -0400
categories: [blue-team]
tags: [blue-team,intune]
image:
    path: /assets/img/posts/entraidemployeehiredate/thumbnail.png
--- 

<style>
  .body {
    display:block;
  }
</style>

## Introduction

In today’s world, automation is the key to ensuring success as human error does exist. In this blog post, I will break down the steps to incorporate employee hire dates into Azures Entra ID to enhance workflow efficiency for Identity and Access Management (IAM). I will incorporate a basic real-world use-case which will enforce MFA for newly onboarded users after 2 weeks of hire.

## Guide

Step 1) Add the employee hire date to the users Active Directory (AD) profile with an empty attribute. In this example, I have added 20240101120000.0Z to extensionAttribute2 which would make this users hire date January 1, 2024, at 8:00AM EST. The format is yyyyMMddHHMMSS.tZ (UTC Format).

<p class="body"><img class="body" src="./assets/img/posts/entraidemployeehiredate/image01.png" alt=""></p>

Step 2) Remote to your Entra ID Connect server.


Step 3) Open PowerShell as Admin and run “Set-ADSyncScheduler -SyncCycleEnabled $false” to disable the Sync while we make the inbound/outbound rule changes.


Step 4) Open the “Synchronization Rules Editor”

<p class="body"><img class="body" src="./assets/img/posts/entraidemployeehiredate/image02.png" alt=""></p>

Step 5) We are going to add two rules: inbound and outbound. You can optionally add scoping and join rules for each.

a) Inbound Rule:

<p class="body"><img class="body" src="./assets/img/posts/entraidemployeehiredate/image03.png" alt=""></p>

b) Outbound Rule:

<p class="body"><img class="body" src="./assets/img/posts/entraidemployeehiredate/image04.png" alt=""></p>


Step 6) Open PowerShell as Admin and run “Set-ADSyncScheduler -SyncCycleEnabled $true” to re-enable the Sync and run “Start-ADSyncSyncCycle -PolicyType Delta” to run a delta sync.


Step 7) Next, we will setup a dynamic group in Entra ID to only add accounts where the employee hire date is more than 2 weeks. In Entra ID, create a dynamic group like the following and click "Add dynamic query"

<p class="body"><img class="body" src="./assets/img/posts/entraidemployeehiredate/image05.png" alt=""></p>

Step 8) In the dynamic query add the following:

<p class="body"><img class="body" src="./assets/img/posts/entraidemployeehiredate/image06.png" alt=""></p>

Step 9) Link the dynamic group to a Conditional Access (CA) policy that requires MFA and now all newly onboarded users will be required to satisfy MFA after 2 weeks of being onboarded.

<p class="body"><img class="body" src="./assets/img/posts/entraidemployeehiredate/image07.png" alt=""></p>
<p class="body"><img class="body" src="./assets/img/posts/entraidemployeehiredate/image08.png" alt=""></p>