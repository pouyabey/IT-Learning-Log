# Day 4 – Planned Tasks

Today, I will be working on the following tasks:

- Start a Windows Update and Patch Management project using Microsoft Intune. | Completed ✅
- BitLocker Encryption and Recovery Key Audit | Completed ✅
- Troubleshoot and repair several laptops. 
- Prepare new computers for incoming employees. | Completed ✅
- Explore ways to better organize and manage IT devices and equipment in storage.

---


## Step 1 – Review Existing Windows Update Policies

The first step of the project was to review the existing Windows Update policies configured in Microsoft Intune.

I identified two Update Ring policies:

### Normal Patching

Assignments:

- Included: All Devices and All Users
- Excluded: ITStaffSystems and Intune Test Devices

Key settings:

- Microsoft product updates: Allow
- Windows drivers: Allow
- Quality update deferral: 3 days
- Feature update deferral: 2 days
- Windows 10 to Windows 11 upgrade: Enabled
- Automatic update behavior: Notify download
- Quality update deadline: 14 days
- Feature update deadline: 14 days
- Grace period: 7 days
- Auto reboot before deadline: Enabled

### Patch Tuesday Policy – Pilot

Assignments:

- Included: Intune Test Devices
- Excluded: None

Key settings:

- Microsoft product updates: Allow
- Windows drivers: Allow
- Quality update deferral: 2 days
- Feature update deferral: 2 days
- Windows 10 to Windows 11 upgrade: Enabled
- Automatic update behavior: Auto install during maintenance time
- Active hours: 8:00 AM to 9:00 PM
- Quality update deadline: 3 days
- Feature update deadline: 3 days
- Grace period: 3 days
- Auto reboot before deadline: Enabled

The review showed that the organization is using a pilot deployment approach, where selected test devices receive updates under a more aggressive update schedule before the broader production environment.

---

## Step 2 – Review Update Ring Deployment Status

I reviewed the device assignment status for both Update Ring policies.

### Patch Tuesday Policy – Pilot

The Intune Test Devices group contains three devices.

The policy deployment report showed:

- Succeeded: 0
- Conflict: 1
- Error: 0
- Not applicable: 0

### Normal Patching

The deployment report showed:

- Succeeded: 92
- Conflict: 1
- Error: 5
- Not applicable: 19

This review identified several deployment issues that require further investigation, including one policy conflict in the pilot deployment and several errors in the normal production deployment.

---

## Step 3 – Investigate Update Ring Policy Conflicts

### Step 3.1 – Identify the Conflicting Device and Settings

I investigated the device reporting a conflict under the Patch Tuesday Policy – Pilot.

The following settings were reported as conflicting:

- Automatic update behavior
- Deadline for feature updates
- Deadline for quality updates
- Grace period
- Quality update deferral

The Windows 11 upgrade setting was reported as not applicable.

I then reviewed the device configuration policies assigned to the affected device and found that it was receiving both:

- Normal Patching
- Patch Tuesday Policy – Pilot

The pilot devices are members of the Intune Test Devices group, which is excluded from the Normal Patching policy. However, the Normal Patching policy is assigned to both All Devices and All Users.

This indicates that the device may be excluded from the device-based assignment through the Intune Test Devices group while still receiving the same policy through its user-based assignment.

As a result, the device is receiving two Update Ring policies with different configuration values, creating conflicts for several Windows Update settings.

This identified a potential issue in the current policy assignment design involving mixed user-based and device-based targeting.

### Step 3.2 – Confirm the Cause of the Conflict

I reviewed the assignments of the Normal Patching policy again.

The policy is assigned to:

- All Devices
- All Users

The following groups are excluded:

- Intune Test Devices
- ITStaffSystems

I also reviewed the affected pilot device and confirmed that both the Normal Patching policy and the Patch Tuesday Policy – Pilot were visible under its device configuration policies.

The same logged-in user was associated with both policy entries.

Although the Intune Test Devices group is excluded from the Normal Patching policy, the policy is also assigned to All Users. This suggests that the affected device may still be receiving the Normal Patching policy through the user-based assignment.

This explains why the device is receiving both Update Ring policies at the same time.

---

### Step 3.3 – Review the Assignment Design

The current assignment structure mixes device-based and user-based targeting.

The intended deployment structure appears to be device-based:

- Intune Test Devices receive the Patch Tuesday Policy – Pilot
- Other devices receive the Normal Patching policy
- ITStaffSystems are excluded from the Normal Patching policy

However, because the Normal Patching policy is also assigned to All Users, devices that are excluded through a device group may still receive the policy through their associated user.

This can result in a device receiving both the pilot and production Update Ring policies.

---

### Step 3.4 – Identify the Recommended Assignment Change

Based on the investigation, the recommended configuration is to use a consistent device-based assignment model for the Update Ring policies.

The proposed Normal Patching assignment structure is:

Included:

- All Devices

Excluded:

- Intune Test Devices
- ITStaffSystems

The All Users assignment should be removed from the Normal Patching policy.

This would allow devices in the Intune Test Devices group to receive only the Patch Tuesday Policy – Pilot while the remaining devices continue to receive the Normal Patching policy.

This change should remove the overlapping policy assignment and resolve the Update Ring setting conflicts identified on the pilot device.

Before making the change in the production environment, the proposed assignment modification should be reviewed and approved by the system administrator.


## Step 4 – Investigate Update Ring Errors

After reviewing the policy conflict, I investigated the five devices reporting errors under the Normal Patching Update Ring.

All five devices showed the same error pattern.

For documentation purposes, the device names were anonymized as Device-A through Device-E.

The common findings were:

- Logged-in user: SYSTEM account
- Last check-in: Today
- AllowWindows11Upgrade: Not applicable
- Deadline for feature updates: Error
- Deadline for quality updates: Error
- Error code: 2016281111

All five devices were running Windows 11.

The OS versions were:

- Two devices: 10.0.26200.8655
- Three devices: 10.0.26100.8655

Because the same errors appeared across devices running two different Windows builds, the issue did not appear to be limited to a single Windows version.

---

### Step 4.1 – Compare User and Device Context

I reviewed the device configuration entries for the affected devices and found that the Normal Patching policy appeared twice on each device:

- One entry under the SYSTEM account
- One entry under the user's normal company account

The deadline errors appeared only under the SYSTEM account entry.

The same policy applied under the normal user account did not report the deadline errors.

This showed that the error was occurring specifically in the device context rather than the user context.

Since the Normal Patching policy is assigned to both All Devices and All Users, the same policy is being evaluated through both device-based and user-based assignments.

---

### Step 4.2 – Review Other Intune Policies

I reviewed the other device configuration policies assigned to the affected devices to determine whether another Intune policy was also controlling Windows Update deadline settings.

No additional Intune policies related to Windows Update, update deadlines, restart behavior, or patching were identified.

This reduced the likelihood that the error was caused by another Intune configuration policy.

---

### Step 4.3 – Review Group Policy Possibilities

Because the environment uses both on-premises Active Directory and Microsoft Intune, I also investigated whether Group Policy could be contributing to the issue.

I generated a Group Policy Results report using:

gpresult /h "%USERPROFILE%\Desktop\gpresult.html"

I reviewed the applied Group Policy Objects and searched for policies related to:

- Windows Update
- Automatic Updates
- Update deadlines
- Windows Update for Business
- Patching

No clearly related Windows Update Group Policy was identified in the available report.

However, the report was generated from my assigned workstation rather than directly from one of the five affected devices.

Therefore, this could not fully confirm whether an affected device was receiving additional device-level Group Policy settings.

---

### Current Status and Escalation

The investigation narrowed the issue to the device-context application of the Normal Patching policy.

The current findings are:

- All five affected devices show the same deadline errors.
- The issue occurs only under the SYSTEM account / device context.
- The same policy under the user context does not show the error.
- The issue affects multiple Windows 11 builds.
- No additional conflicting Intune update policy was identified.
- No clearly related Windows Update GPO was identified from the available Group Policy report.

Further investigation would require additional access to one of the affected devices or access to more detailed device-level Group Policy, registry, event log, or Windows Update diagnostic information.

Because I did not have the required level of access to perform those checks directly, I documented the findings and escalated the issue to the system administrator for further investigation.


## Project Summary

- Reviewed existing Windows Update Rings to understand the current patching structure and policy assignments.
- Compared the Normal Patching and Pilot policies to identify differences in update timing, deadlines, and restart behavior.
- Reviewed deployment status to identify Conflict, Error, and Not Applicable devices.
- Investigated the Pilot conflict and identified overlapping Normal Patching and Pilot policies on the same device.
- Reviewed policy assignments and identified mixed user-based and device-based targeting as the likely cause of the overlap.
- Recommended using a consistent device-based assignment model for the Update Rings.
- Investigated five devices reporting the same Normal Patching errors.
- Compared Windows builds to check whether the errors were limited to a specific OS version; the same errors appeared on multiple builds.
- Compared SYSTEM and user-context policy entries to identify where the errors occurred; the errors appeared only in the device/SYSTEM context.
- Reviewed other Intune policies to check for additional Windows Update configurations; none were found.
- Generated a Group Policy Results report to check whether an Active Directory GPO could be controlling Windows Update settings:

gpresult /h "%USERPROFILE%\Desktop\gpresult.html"

- Reviewed the applied GPOs and found no clearly related Windows Update policy on the available workstation.
- Escalated the remaining issue because further troubleshooting required direct access to an affected device or additional device-level diagnostic access.


---


##  BitLocker Encryption and Recovery Key Audit

### Step 1 – Review Current Encryption Status

I started by reviewing the Windows device encryption report in Microsoft Intune.

The report included:

- Device name
- Operating system
- OS version
- TPM version
- Encryption readiness
- Encryption status
- User principal name

The audit showed that the number of encrypted and unencrypted devices was approximately equal.

Most of the unencrypted devices were marked as Ready for encryption, indicating that many devices support BitLocker but are currently not encrypted.

### Step 2 – Review Existing BitLocker Policies

I reviewed the following areas in Microsoft Intune:

- Endpoint Security → Disk Encryption
- Devices → Configuration Policies

No existing BitLocker or disk encryption policies were found.

### Step 3 – Create a BitLocker Pilot Policy

I created a new BitLocker policy under Endpoint Security → Disk Encryption.

The policy was configured to:

- Require device encryption
- Configure recovery password rotation
- Use XTS-AES 256-bit encryption for operating system and fixed data drives
- Use AES-CBC 256-bit encryption for removable drives
- Store BitLocker recovery information in Active Directory Domain Services
- Prevent BitLocker activation until recovery information is backed up to AD DS
- Use TPM-based BitLocker protection without requiring an additional startup PIN

The policy was assigned only to the Intune Test Devices group for controlled testing before wider deployment.
