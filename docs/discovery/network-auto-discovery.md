---
title: "SNMP - Network Autodiscovery"
sidebar_position: 27
---

The following steps explain configuring and running SNMP discovery against your network. Please see our [list of vendors supported by Device42 for SNMP autodiscovery](https://docs.device42.com/auto-discovery/vendors-supported-in-snmp-auto-discovery/) for a list of supported hardware vendors. Please let us know if you had a device that needs additional support!

## Intro to SNMP-based discovery

SNMP, or Simple Network Management Protocol, is a protocol and a standard that is supported by just about any managed network-connected hardware. There are three widely deployed versions: SNMP v1, v2c _(most commonly used)_, and v3. SNMP is typically utilized read-only, but supports read/write, and by default utilized port 161. SNMP exposes management data in the form of 'variables', which are organized in what is known as a MIB, or "Management Information Base". A MIB essentially describes the variables available on a given system, each of which can be remotely queried via SNMP.

**Note**: SNMP autodiscovery now supports IPv6 addresses for device discovery.

### What can be autodiscovered using SNMP?

Network devices can be discovered by Device42 using SNMP v1, v2c, or v3. If you're looking to do [Storage discovery](storage_arrays_autodiscovery/snmp-san-server-auto-discovery) via SNMP, you may want to visit the dedicated [SNMP SAN/Server Auto-Discovery](storage_arrays_autodiscovery/snmp-san-server-auto-discovery) page. SNMP discovery will pull in CDP/LLDP neighbors as long as SNMP credentials are the same across all neighbors. Should the credentials \*not\* be the same, you may instead add devices using different credentials separately, as their own discovery job.

### Specific categories of data SNMP can discover

Depending on device type and compatibility matrix linked above, the following data is discovered:

- **Switch inventory:** Switch name, serial #, model and manufacturer.
- **Stacked switches:** For stacked switches, it will add the stack as cluster device and all physical devices as part of cluster.
- **Access Points**: Access points will be added as device type other with Controller device as the device host.
- **VLANs** : L2 vlans.
- **Subnets:** L3 subnets.
- **Switch IP and MAC address** : IP address and MAC address belonging to the switch.
- **IP to MAC address association:** Basically the ARP table, if available. So all IPs that are available with MAC association.
- **MAC address to switch port association:** Switch ports and mac addresses found on that port. (MAC table)

MAC to switch port association brings only switch ports that have MAC addresses associated. Using "Get all switch ports" option you can get:

- **Port name**
- **Port description**
- **Port up/down status**
- **Port administratively up/down status**
- **Remote port connectivity, if any**

* * *

## SNMP discovery jobs

### Create or edit an SNMP discovery job

![](/assets/images/SNMP-menuadd-job-700x395.png)

**Note**: when creating an SNMP job for Cisco Nexus, you must setup an SNMP server context for the management VRF (this actually needs to be done for any Cisco VRF contexts that you want to query over SNMP): snmp\-server context mymgmt vrf management

Go to _Discovery > SNMP_ to add a new network autodiscovery job.

- **IP Address**: Enter the IP address of a switch or an IP range.
- **Port**: Leave at 161 if you are unsure
- **SNMP Version**: Choose SNMP v1, v2c, or v3
- **Community String**: Save your community string(s) as passwords, and select them for v1 or v2c. See below for v3.
- **Run Autodiscover on CDP/LLDP Neighbors**: Find all CDP/LLDP neighbors that are reachable.
- **Strip Domain Name**: Strip domain name from discovered switch name.
- **Get all Switch Ports**: Retrieve all switch ports.
- **Delete Switch Ports Not Found**: Delete any switch ports in Device42 that were not found in this discovery.
- **Use Alias/Name for port description**: Choose if you prefer the Alias/Name for the port description.
- **Delete older mac association after**: Delete any mac addresses not found for the specified number of days.
- **ICMP/TCP Port Check**: If you experience any issues with mulitcast IP's, then you will need to uncheck the ICMP/TCP Port Check option.

![](/assets/images/Screen-Shot-2022-05-01-at-2.28.26-PM.png)

### Device-specific SNMP v3 info

Note on Cisco Nexus 7K switches:
- The user for SNMP v3 autodiscovery may need to be in the network-operator / vdc-operator group.

Note on Huawei Switches:
- By default, some Huawei devices ship with LLDP (link layer discovery protocol) via SNMP off.
- You must switch it on by creating a new 'mib-view' and attaching the 'ISO tree' containing the Huawei LLDP MIB to the community.
- Consult Huawei's documentation for complete setup & management details.

Note on Cisco Switches:
- Changing from SNMP v1 or v2c to v3 on many Cisco switches can cause SNMP polling of Netdisco to stop functioning, preventing collection of the per-VLAN MAC tables.
- You will likely see an authorization error in the macsuck log if this is happening.
- To fix this authentication error on Cisco hardware, an additional snmp-server configuration is required on these switches that enables access to the per-VLAN/per-context MAC address table:

Switches running newer versions of Cisco IOS:
- Simply run: `snmp-server group v3group v3 auth context vlan- match prefix once.`

Switches with older IOS releases (versions that don't support "match prefix wildcard"):
- Issue the above command for newer IOS releases on EACH VLAN configured for the switch.
- Use `show snmp context` to list configured VLANs.


* * *

### Preferred Credentials

You can enter preferred community string credentials when you create an SNMP discovery job. When the job runs, it will use the credentials in the order in which you enter them, stopping at the first successful authentication. Subsequent job runs use the last successful credential and then the remaining credentials in the ordered list. 

![](/assets/images/SNMP-multi-creds-1-700x338.png)

- Click _Add another community string_ and use the magnifying glass icon to select the secret for the community string.
- Click _Move Up_ or _Move Down_ to order the credentials.

![](/assets/images/v3_creds.png)

## Network Device Options

Expanding the "Network Device Options" section will reveal the following settings, specific to the discovery of network connected hardware and devices: ![](/assets/images/SNMP-add-job-net-device-options-700x323.png)

### Get all switch ports

![](/assets/images/SNMP-add-job-net-device-options-2-700x399.png) 

If you select _Get all switch ports_, you will see 6 extra form items:

1. **Port name prefix to ignore macs**: Ignore mac addresses from port that start with this prefix.
2. **VLANs to ignore** : Do not discover mac addresses on these VLANs.
3. **Give precedence to hostname**: Check this option to give precedence to the discovered hostname in the network device discovery.
4. **Delete older mac association after**: To keep your mac addresses and switch port connectivity up-to-date leave this at 0. This will delete all stale mac addresses not discovered on switch port anymore. Otherwise, you can choose the # of days after you want to delete the stale mac association with a switch port.
5. **Discovered port types to ignore**: You might not want to see certain ports types in your switch port list. Here you can choose what port types to ignore. For first time:
    - You will have to let it find the port types first time,
    - and if you want to ignore some, you will have to delete the switch ports manually(you can filter by discovered type on IPAM > Switch Ports),
    - add the ports to ignore list on the discovery page
6. **Discovered port types not to count:** Similar to above. This will still bring the ports in, but selected port types will not be included in the count.

## Run Now or Schedule

![](/assets/images/image-700x115.png)

Select **Run Now** from the list page to run the job right away.

![](/assets/images/AD_Blade-Discovery-Run-Schedule.png)

Select **Add another Autodiscovery Schedule** from the when editing the job to create a run schedule for the job.

A note on autodiscovery scheduling behavior: newly created jobs will not run on the first day they are created, to prevent an unintended large amount of jobs from running initially. If you would like to run a job after its initial creation, simply select the "Run Now" button next to the job after creation.

## Status

You can view the status and/or results of a discovery job during or after the job has run by visting the job edit screen: ![](/assets/images/SNMP-job-status-tab-700x233.png) You can also see a real-time report of all running jobs and their statuses by visiting _Reports > Job Status:_ ![](/assets/images/SNMP-job-status-dashboard-700x223.png) Status information can be viewed in full after each execution of the resptive job.

## Run network discovery

![Run network discovery](/assets/images/run_network_discovery-2018v15.png) 

Once you have saved the network switch for autodiscovery, you will need to kick off the autodiscovery process. If not scheduled, you can click on "Run now" button on list, view or edit page.
