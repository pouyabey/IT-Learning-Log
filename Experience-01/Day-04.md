# Day 4 – Planned Tasks

Today, I will be working on the following tasks:

- Start a Windows Update and Patch Management project using Microsoft Intune.
- Troubleshoot and repair several laptops.
- Prepare new computers for incoming employees.
- Update the inventory of IT equipment in the storage area.
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
