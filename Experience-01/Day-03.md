# Day 3 – Intune Device Inventory and Compliance Audit

Today is my third day, and I have started working on a new project focused on Microsoft Intune device management and compliance.

## Project Overview

This project focuses on reviewing and analyzing the organization's devices managed through Microsoft Intune. The goal is to understand the current device environment and identify potential management, compliance, or security issues.

The audit will review device types, operating systems, enrollment status, ownership, compliance status, last check-in time, OS versions, encryption status, and inactive or outdated devices.

## Goal

The main goal is to create a clear overview of the organization's Intune-managed devices and identify devices that may require further investigation or remediation.

## Expected Outcome

By the end of the project, I will have a better understanding of how devices are managed and monitored in Intune and will produce a summary of the current device inventory, compliance status, and identified issues.

--- 


## Step 1 – Review the Current Intune Device Inventory

The first step of the project was to review the current device inventory in Microsoft Intune and understand the existing managed device environment.

I navigated to the All Devices section in Intune and reviewed the available device information, including:

- Device name
- Management platform
- Ownership
- Compliance status
- Operating system
- OS version
- Primary user
- Last check-in time

The current environment includes 137 managed devices:

- 132 Windows devices
- 5 macOS devices

All devices are currently managed by Microsoft Intune. Most devices are classified as Corporate-owned, while a smaller number are marked as Personal.

During the initial review, I identified several noncompliant devices. Some had checked in recently, while others had not checked in for an extended period. This showed that noncompliance and device inactivity need to be investigated separately.

I then reviewed several recently active noncompliant devices to identify the specific causes of noncompliance. The most common issues found were:

- Firewall noncompliance under the Basic Compliance policy
- TPM noncompliance on multiple devices
- Devices marked as noncompliant because they were not considered active
- One device that did not appear to have the Basic Compliance policy assigned

The initial review showed that Firewall and TPM requirements are recurring compliance issues in the environment. It also identified inactive devices and a possible compliance policy assignment gap that require further investigation.

This completes the initial inventory and compliance review. The next step will focus on reviewing compliance policy assignments, investigating recurring failures, and determining which devices should be prioritized for remediation.

---

## Step 2 – Review Compliance Policy Assignment and Scope

The second step focused on reviewing how the Basic Compliance policy is assigned and investigating why one managed device was not receiving the policy.

I opened the Basic Compliance policy and reviewed its assignment settings.

### Policy Assignment

The policy is currently assigned to:

- All users

There are no excluded groups.

### Policy Monitoring

The current policy status shows:

- Compliant devices: 110
- Noncompliant devices: 2
- Other status: 4
- Total devices evaluated: 116

I searched for one of the devices identified in Step 1 that only showed the Default Device Compliance Policy. The device did not appear in the Basic Compliance policy results.

To investigate further, I reviewed the device and its associated user.

### Device and User Review

The device had:

- A valid Primary User
- Microsoft Entra joined status
- Microsoft Intune as its MDM provider
- A registered owner
- Only the Default Device Compliance Policy visible under its compliance policies

The associated user account was:

- Enabled
- User type: Member

The Default Device Compliance Policy showed that the device had a compliance policy assigned, but the Basic Compliance policy was still missing.

### License Investigation

I then reviewed the user's assigned licenses and found:

- Office 365 E1
- Microsoft Power Automate Free

The user did not have Microsoft 365 E3 assigned.

I also reviewed the organization's available licenses and confirmed that Microsoft 365 E3 licenses are available in the tenant. Users whose devices are receiving the Basic Compliance policy were found to have Microsoft 365 E3 assigned.

This created a strong indication that licensing may be related to the issue. Microsoft 365 E3 includes Intune licensing, while Office 365 E1 alone does not.

### Current Finding

The investigation identified a likely licensing-related cause for the missing Basic Compliance policy. The affected user does not have an Intune-inclusive license assigned, while users whose devices successfully receive the policy have Microsoft 365 E3 licenses.

No configuration or licensing changes were made during this investigation. Further validation should be completed before making any changes.




