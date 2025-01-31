---
title: Upgrade from NetQ 2.0/2.1 to NetQ 2.2.x
author: Cumulus Networks
weight: 125
aliases:
 - /display/NETQ21/Upgrade+from+NetQ+2.0_2.1+to+NetQ+2.2.x
 - /pages/viewpage.action?pageId=12321007
pageID: 12321007
product: Cumulus NetQ
version: '2.1'
imgData: cumulus-netq-21
siteSlug: cumulus-netq-21
---
This document describes the steps required to upgrade from all NetQ 2.0
and NetQ 2.1 releases to NetQ 2.2.x.

{{%notice info%}}

Cumulus Networks recommends only upgrading NetQ during a network
maintenance window.

Any data you have collected while using NetQ 2.1.0 is maintained during
this upgrade process.

{{%/notice%}}

{{%notice note%}}

Events generated during the upgrade process will not be available in the
database. Once the upgrade process is complete, the agents re-sync with
the current state of the Host or Cumulus Linux switch with the NetQ
Platform.

{{%/notice%}}

To upgrade from NetQ 1.x to NetQ 2.2.x, please follow the instructions
[here](/version/cumulus-netq-21/Cumulus-NetQ-Deployment-Guide/Upgrade-NetQ/Upgrade-from-NetQ-1.x-to-NetQ-2.2.x).
Instructions for installing NetQ 2.2.x for the first time can be found
[here](/version/cumulus-netq-21/Cumulus-NetQ-Deployment-Guide/Install-NetQ).

## Prerequisites</span>

Before you begin the upgrade process, please note the following:

  - The minimum supported Cumulus Linux version for NetQ 2.2.x is 3.3.2.

  - You must upgrade your NetQ Agents as well as the NetQ Platform.

  - You can upgrade to NetQ 2.2.x without upgrading Cumulus Linux.

  - The NetQ installer pod `netq-installer` should be up in either the
    *Containercreating* or *Running* state. The `netq-installer` pod
    state could also be *ContainerCreating*, in which case the host is
    initializing with the SSH keys.

## Upgrade the NetQ Platform</span>

To upgrade the NetQ Platform:

1.  Download the NetQ Platform VM upgrade image.
    
    1.  On the [Cumulus
        Downloads](https://cumulusnetworks.com/downloads/) page, select
        *NetQ* from the **Product** list box.
    
    2.  Click *2.2* from the **Version** list box, and then select
        *2.2.x* from the submenu.
    
    3.  Optionally, select the hypervisor you wish to use (*VMware,
        VMware (cloud), KVM,* or *KVM (cloud)*) from the
        **Hypervisor/Platform** list box.  
        **Note**: You can ignore the ONIE and Appliance options, as they
        are for the NetQ appliances.
        
        {{% imgOld 0 %}}
    
    4.  Scroll down to review the images that match your selection
        criteria.
        
        {{% imgOld 1 %}}
    
    5.  Click **Upgrade** for the relevant version, being careful to
        select the correct deployment version.

2.  From a terminal window, log in to the NetQ Platform using your login
    credentials. This example uses the default *cumulus/CumulusLinux\!*
    credentials.
    
        <computer>:~<username>$ ssh cumulus@netq-platform
        cumulus@netq-platform's password: 
        cumulus@netq-platform:~$ 

3.  Change to the root user.
    
        cumulus@netq-platform:~$ sudo -i
        [sudo] password for cumulus:
        root@netq-platform:~#

4.  Create an *installables* subdirectory in the mount directory.
    
        root@netq-platform:~# mkdir -p /mnt/installables/
        root@netq-platform:~#

5.  Copy the upgrade image file into your new directory. The on-site
    file is named `NetQ-2.2.0.tgz` and the cloud file is named
    `NetQ-2.2.0-opta.tgz`.
    
        root@netq-platform:~# cd /mnt/installables/
        root@netq-platform:/mnt/installables# cp /home/usr/dir/<NetQ-image>.tgz ./ 

6.  Export the installer script.
    
        root@netq-platform:/mnt/installables# tar -xvf <NetQ-image>.tgz ./netq-install.sh

7.  Verify the contents of the directory. You should have the image file
    and the `netq-install.sh` script.
    
        root@netq-platform:/mnt/installables# ls -l
        total 9607744
        -rw-r--r-- 1 cumulus cumulus 5911383922 Apr 23 11:13 <NetQ-image>.tgz
        -rwxr-xr-x 1 _lldpd _lldpd 4309 Apr 23 10:34 netq-install.sh
        root@netq-appliance:/mnt/installables#

8.  Configure SSH access.
    
    {{%notice info%}}
    
    If you perform the upgrade more than once, you can skip this step
    after performing it once.
    
    If you have an existing SSH key, skip to step 8c.
    
    {{%/notice%}}
    
    1.  Generate the SSH key to enable you to run the script.
        
        {{%notice info%}}
        
        Leave the passphrase blank to simplify running the script.
        
        {{%/notice%}}
        
            root@netq-platform:/mnt/installables# ssh-keygen -t rsa -b 4096
            Generating public/private rsa key pair.
            Enter file in which to save the key (/root/.ssh/id_rsa):
            Created directory '/root/.ssh'.
            Enter passphrase (empty for no passphrase): 
            Enter same passphrase again:
            Your identification has been saved in /root/.ssh/id_rsa.
            Your public key has been saved in /root/.ssh/id_rsa.pub.
    
    2.  Copy the key to the `authorized_keys` directory.
        
            root@netq-platform:/mnt/installables# cat ~/.ssh/id_rsa.pub >> ~/.ssh/authorized_keys
            root@netq-platform:/mnt/installables# chmod 0600 ~/.ssh/authorized_keys
            root@netq-platform:/mnt/installables#
    
    3.  Associate the key with the installer.
        
            root@netq-platform:/mnt/installables/# ./netq-install.sh --usekey ~/.ssh/id_rsa
            [Fri 21 Jun 2019 06:34:47 AM UTC] - This Script can only be invoked by user: root
            [Fri 21 Jun 2019 06:34:47 AM UTC] - The logged in user is root
            [Fri 21 Jun 2019 06:34:47 AM UTC] - Install directory /mnt/installables exists on system.
            [Fri 21 Jun 2019 06:34:47 AM UTC] - File /root/.ssh/id_rsa exists on system...
            [Fri 21 Jun 2019 06:34:47 AM UTC] - checking the presence of existing instaler-ssh-keys secret/instaler-ssh-keys created
            [Fri 21 Jun 2019 06:34:48 AM UTC] - Unable to find netq-installer up and running. Sleeping for 10 seconds ...
            [Fri 21 Jun 2019 06:34:58 AM UTC] - Unable to find netq-installer up and running. Sleeping for 10 seconds ...
            [Fri 21 Jun 2019 06:35:08 AM UTC] - Able to find the netq-installer up and running...

9.  Upgrade the NetQ software. This example shows an upgrade to version
    2.2.0, on-site deployment.
    
        root@netq-platform:/mnt/installables# ./netq-install.sh  --installbundle  /mnt/installables/NetQ-2.2.0.tgz --updateapps
        [Fri 21 Jun 2019 08:18:37 PM UTC] - File /mnt/installables/NetQ-2.2.0.tgz exists on system for updating netq-installer ...
        [Fri 21 Jun 2019 08:18:37 PM UTC] - Check the netq-installer is up and running to process requests ....
        [Fri 21 Jun 2019 08:18:37 PM UTC] - Checking the Status of netq-installer ....
        [Fri 21 Jun 2019 08:18:37 PM UTC] - The netq-installer is up and running ...
        [Fri 21 Jun 2019 08:18:37 PM UTC] - Updating the netq-installer ...
        [Fri 21 Jun 2019 08:18:37 PM UTC] - Able to execute the command for updating netq-installer ...
        [Fri 21 Jun 2019 08:18:37 PM UTC] - Checking initialization of netq-installer update ...
        [Fri 21 Jun 2019 08:18:37 PM UTC] - Update of netq-installer is in progress ...
        ******************************0
        [Fri 21 Jun 2019 08:28:39 PM UTC] - Successfully updated netq installer....
        0,/mnt/installables/NetQ-2.2.0.tgz
        [Fri 21 Jun 2019 08:28:39 PM UTC] - File /mnt/installables/NetQ-2.2.0.tgz exists on system for updating netq apps...
        [Fri 21 Jun 2019 08:28:39 PM UTC] - User selected to update netq-apps ...
        [Fri 21 Jun 2019 08:28:39 PM UTC] - Checking the Status of netq-installer ....
        [Fri 21 Jun 2019 08:28:41 PM UTC] - Unable to find netq-installer up and running. Sleeping for 10 seconds ...
        [Fri 21 Jun 2019 08:28:52 PM UTC] - Unable to find netq-installer up and running. Sleeping for 10 seconds ...
        [Fri 21 Jun 2019 08:29:03 PM UTC] - Unable to find netq-installer up and running. Sleeping for 10 seconds ...
        [Fri 21 Jun 2019 08:29:14 PM UTC] - Unable to find netq-installer up and running. Sleeping for 10 seconds ...
        [Fri 21 Jun 2019 08:29:24 PM UTC] - The netq-installer is up and running ...
        [Fri 21 Jun 2019 08:29:24 PM UTC] - Able to execute the command for netq apps updates ...
        [Fri 21 Jun 2019 08:29:24 PM UTC] - Checking initialization of apps update ...
        [Fri 21 Jun 2019 08:29:29 PM UTC] - netq apps update is in progress ...
        ************************************************************************************************************************************    ******************************************************************************************************************************************************************************
        0
        [Fri 21 Jun 2019 09:20:31 PM UTC] - Successfully updated netq apps ....
        root@netq-appliance:/mnt/installables#
    
    {{%notice info%}}
    
    Please allow about an hour for the upgrade to complete.
    
    {{%/notice%}}

{{%notice note%}}

If you have changed the IP Address of the NetQ Platform, you need to
re-register this address with the Kubernetes containers before you can
continue.

1.  Reset all Kubernetes administrative settings. Run the command twice
    to make sure all directories and files have been reset.
    
    ` cumulus@switch:~$ sudo kubeadm reset -f  `  
    `cumulus@switch:~$ sudo kubeadm reset -f`

2.  Remove the Kubernetes configuration.  
    `cumulus@switch:~$ sudo rm /home/cumulus/.kube/config`

3.  Reset the NetQ Platform install daemon.  
    `cumulus@switch:~$ sudo systemctl reset-failed`  
    `  `

4.  Reset the Kubernetes service.  
    ` cumulus@switch:~$ sudo systemctl restart cts-kubectl-config  `  
    ***Note**: Allow 15 minutes for the prompt to return.*

5.  Reboot the VM.  
    ***Note**: Allow 5-10 minutes for the VM to boot.*

{{%/notice%}}

### Verify the Installation</span>

1.  Verify you can access the NetQ CLI.
    
    1.  From a terminal window, log in to the NetQ Platform using the
        default credentials (*cumulus/CumulusLinux\!*).
        
            <computer>:~<username>$ ssh cumulus@<netq-platform-ipaddress>
            Warning: Permanently added '<netq-platform-hostname>,192.168.1.254' (ECDSA) to the list of known hosts.
            cumulus@<netq-platform-hostname>'s password: <enter CumulusLinux! here>
             
            Welcome to Cumulus (R) Linux (R)
             
            For support and online technical documentation, visit
            http://www.cumulusnetworks.com/support
             
            The registered trademark Linux (R) is used pursuant to a sublicense from LMI,
            the exclusive licensee of Linus Torvalds, owner of the mark on a world-wide
            basis.
             
            cumulus@<netq-platform-hostname>:~$ 
    
    2.  Run the following command to verify all applications are
        operating properly. ***Note**: Please allow 10-15 minutes for
        all applications to come up and report their status.*
        
            cumulus@<netq-platform-hostname>:~$ netq show opta-health
            Application                    Status    Health    Kafka Stream    Git Hash    Timestamp
            -----------------------------  --------  --------  --------------  ----------  ------------------------
            netq-app-macfdb                UP        true      up              14b42e6     Mon Jun  3 20:20:35 2019
            netq-app-interface             UP        true                      0fe11c6     Mon Jun  3 20:20:34 2019
            netq-app-vlan                  UP        true                      4daed85     Mon Jun  3 20:20:35 2019
            netq-app-sensors               UP        true      up              f37272c     Mon Jun  3 20:20:34 2019
            netq-app-topology              UP        true                      3f4a887     Mon Jun  3 20:20:34 2019
            kafka-broker                   UP                                              Mon Jun  3 20:20:35 2019
            netq-app-mstpinfo              UP        true      up              ef5565d     Mon Jun  3 20:20:35 2019
            netq-app-address               UP        true      up              7e0d03d     Mon Jun  3 20:20:35 2019
            netq-gui                       UP                                              Mon Jun  3 20:20:35 2019
            netq-app-kube                  UP        true      up              fbcaa9d     Mon Jun  3 20:20:34 2019
            netq-app-link                  UP        true      up              6c2b21a     Mon Jun  3 20:20:35 2019
            netq-app-ptm                   UP        true      up              7162771     Mon Jun  3 20:20:34 2019
            netq-opta                      UP        true                                  Mon Jun  3 20:20:34 2019
            netq-app-clagsession           UP        true      up              356dda9     Mon Jun  3 20:20:34 2019
            netq-endpoint-gateway          UP        true                      295e9ed     Mon Jun  3 20:20:34 2019
            netq-app-ospf                  UP        true      up              e0e2ab0     Mon Jun  3 20:20:34 2019
            netq-app-lldp                  UP        true      up              90582de     Mon Jun  3 20:20:35 2019
            netq-app-inventory             UP        true      up              bbf9938     Mon Jun  3 20:20:34 2019
            netq-app-tracecheck-scheduler  UP        true                      5484c68     Mon Jun  3 20:20:34 2019
            netq-app-infra                 UP        true      up              13f9e7c     Mon Jun  3 20:20:34 2019
            kafka-connect                  UP                                              Mon Jun  3 20:20:35 2019
            netq-app-search                UP        true      up              e47aaba     Mon Jun  3 20:20:34 2019
            netq-app-procdevstats          UP        true      up              b8e280e     Mon Jun  3 20:20:34 2019
            netq-app-vxlan                 UP        true      up              123c577     Mon Jun  3 20:20:34 2019
            zookeeper                      UP                                              Mon Jun  3 20:20:35 2019
            netq-app-resource-util         UP        true      up              41dfb07     Mon Jun  3 20:20:34 2019
            netq-app-evpn                  UP        true      up              05a4003     Mon Jun  3 20:20:34 2019
            netq-api-gateway               UP        true                      c40231a     Mon Jun  3 20:20:34 2019
            netq-app-port                  UP        true      up              4592b70     Mon Jun  3 20:20:35 2019
            netq-app-macs                  UP        true                      dd6cd96     Mon Jun  3 20:20:35 2019
            netq-app-notifier              UP        true      up              da57b69     Mon Jun  3 20:20:35 2019
            netq-app-events                UP        true      up              8f7b4d9     Mon Jun  3 20:20:34 2019
            netq-app-services              UP        true      up              5094f4a     Mon Jun  3 20:20:34 2019
            cassandra                      UP                                              Mon Jun  3 20:20:35 2019
            netq-app-configdiff            UP        true      up              3be2ef1     Mon Jun  3 20:20:34 2019
            netq-app-neighbor              UP        true      up              9ebe479     Mon Jun  3 20:20:35 2019
            netq-app-bgp                   UP        true      up              e68f7a8     Mon Jun  3 20:20:35 2019
            schema-registry                UP                                              Mon Jun  3 20:20:35 2019
            netq-app-lnv                   UP        true      up              a9ca80a     Mon Jun  3 20:20:34 2019
            netq-app-healthdashboard       UP        true                      eea044c     Mon Jun  3 20:20:34 2019
            netq-app-ntp                   UP        true      up              651c86f     Mon Jun  3 20:20:35 2019
            netq-app-customermgmt          UP        true                      7250354     Mon Jun  3 20:20:34 2019
            netq-app-node                  UP        true      up              f676c9a     Mon Jun  3 20:20:34 2019
            netq-app-route                 UP        true      up              6e31f98     Mon Jun  3 20:20:35 2019
             
            cumulus@<netq-platform-hostname>:~$
        
        {{%notice info%}}
        
        If any of the applications or services display Status as DOWN
        after 30 minutes, open a [support
        ticket](https://cumulusnetworks.com/support/file-a-ticket/) and
        attach the output of the `opta-support` command.
        
        {{%/notice%}}

2.  Verify that NTP is configured and running. NTP operation is critical
    to proper operation of NetQ. Refer to [Setting Date and
    Time](/display/NETQ21/Setting+Date+and+Time) in the *Cumulus Linux
    User Guide* for details and instructions.

3.  Continue the NetQ installation by loading the NetQ Agent on each
    switch or host you want to monitor. Refer to the next section for
    instructions.

## Upgrade the NetQ Agents</span>

Whether using the NetQ Appliance or your own hardware, the NetQ Agent
should be upgraded on each of the existing nodes you want to monitor.
The node can be a:

  - Switch running Cumulus Linux version 3.3.2 or later

  - Server running Red Hat RHEL 7.1, Ubuntu 16.04 or CentOS 7

  - Linux virtual machine running any of the above Linux operating
    systems

To upgrade the NetQ Agent you need to install the OS-specific meta
package, `cumulus-netq`, on each switch. Optionally, you can install it
on hosts. The meta package contains the NetQ Agent, the NetQ command
line interface (CLI), and the NetQ library. The library contains modules
used by both the NetQ Agent and the CLI.

  - [Upgrade NetQ Agent on a Cumulus Linux
    Switch](#src-12321007_safe-id-VXBncmFkZWZyb21OZXRRMi4wLzIuMXRvTmV0UTIuMi54LUFnZW50Q0w)

  - [Upgrade NetQ Agent on an Ubuntu
    Server](#src-12321007_safe-id-VXBncmFkZWZyb21OZXRRMi4wLzIuMXRvTmV0UTIuMi54LUFnZW50VWJ1bnR1)

  - [Upgrade NetQ Agent on a Red Hat or CentOS
    Server](#src-12321007_safe-id-VXBncmFkZWZyb21OZXRRMi4wLzIuMXRvTmV0UTIuMi54LUFnZW50UkhD)

{{%notice info%}}

If your network uses a proxy server for external connections, you should
first <span style="color: #339966;"> <span style="color: #339966;">
[configure a global proxy](/display/NETQ21/Configuring+a+Global+Proxy)
</span> </span> , so apt-get can access the meta package on the Cumulus
Networks repository.

{{%/notice%}}

### <span id="src-12321007_safe-id-VXBncmFkZWZyb21OZXRRMi4wLzIuMXRvTmV0UTIuMi54LUFnZW50Q0w" class="confluence-anchor-link"></span>Upgrade NetQ Agent on a Cumulus Linux Switch</span>

A simple process installs the NetQ Agent on a Cumulus switch.

1.  Edit the `/etc/apt/sources.list` file to add the repository for
    Cumulus NetQ. ***Note** that NetQ has a separate repository from
    Cumulus Linux.*
    
        cumulus@switch:~$ sudo nano /etc/apt/sources.list
        ...
        deb http://apps3.cumulusnetworks.com/repos/deb CumulusLinux-3 netq-2.2
        ...
    
    {{%notice tip%}}
    
    The repository `deb http://apps3.cumulusnetworks.com/repos/deb
    CumulusLinux-3 netq-latest` can be used if you want to always
    retrieve the latest posted version of NetQ.
    
    {{%/notice%}}

2.  Update the local `apt` repository, then install the NetQ meta
    package on the switch.
    
        cumulus@switch:~$ sudo apt-get update
        cumulus@switch:~$ sudo apt-get install cumulus-netq 

3.  Verify the upgrade.
    
        cumulus@switch:~$ dpkg -l | grep netq
        ii  cumulus-netq                      2.2.0-cl3u17~1557345432.a60ec9a    all          This meta-package provides installation of Cumulus NetQ packages.
        ii  netq-agent                        2.2.0-cl3u17~1559681411.2bba220    amd64        Cumulus NetQ Telemetry Agent for Cumulus Linux
        ii  netq-apps                         2.2.0-cl3u17~1559681411.2bba220    amd64        Cumulus NetQ Fabric Validation Application for Cumulus Linux

4.  Restart `rsyslog` so log files are sent to the correct destination.
    
        cumulus@switch:~$ sudo systemctl restart rsyslog.service

5.  Configure the NetQ Agent to send telemetry data to the NetQ Platform
    and, optionally, configure the switch or host to run the NetQ CLI.
    In this example, the IP address for the agent and cli servers is
    *192.168.1.254*.  
    **Note:** If you intend to use VRF, skip to [Configure the Agent to
    Use
    VRF](#src-12321007_safe-id-VXBncmFkZWZyb21OZXRRMi4wLzIuMXRvTmV0UTIuMi54LUFnZW50VlJG).
    If you intend to specify a port for communication, skip to
    [Configure the Agent to Communicate over a Specific
    Port](#src-12321007_safe-id-VXBncmFkZWZyb21OZXRRMi4wLzIuMXRvTmV0UTIuMi54LXBvcnQ).
    
        cumulus@switch:~$ netq config add agent server 192.168.1.254
        cumulus@switch:~$ netq config add cli server 192.168.1.254
    
    This command updates the configuration in the `/etc/netq/netq.yml`
    file and enables the NetQ CLI.

6.  Restart NetQ Agent and CLI.
    
        cumulus@switch:~$ netq config restart agent
        cumulus@switch:~$ netq config restart cli
    
    Repeat these steps for each Cumulus switch, or use an automation
    tool to install NetQ Agent on multiple Cumulus Linux switches.

### <span id="src-12321007_safe-id-VXBncmFkZWZyb21OZXRRMi4wLzIuMXRvTmV0UTIuMi54LUFnZW50VWJ1bnR1" class="confluence-anchor-link"></span>Upgrade NetQ Agent on an Ubuntu Server (Optional)</span>

To install the NetQ Agent on an Ubuntu server:

1.  Reference and update the local `apt` repository.
    
        root@ubuntu:~# wget -O- https://apps3.cumulusnetworks.com/setup/cumulus-apps-deb.pubkey | apt-key add -

2.  In `/etc/apt/sources.list.d/cumulus-host-ubuntu-xenial.list`, verify
    the following repository is included:
    
        root@ubuntu:~# vi /etc/apt/sources.list.d/cumulus-apps-deb-xenial.list
        ...
        deb [arch=amd64] https://apps3.cumulusnetworks.com/repos/deb xenial netq-latest
        ...
    
    {{%notice note%}}
    
    The use of `netq-latest` in this example means that a `get` to the
    repository always retrieves the latest version of NetQ, even in the
    case where a major version update has been made. If you want to keep
    the repository on a specific version — such as `netq-2.1` — use that
    instead.
    
    {{%/notice%}}

3.  Install the meta package on the server.
    
        root@ubuntu:~# apt-get update
        root@ubuntu:~# apt-get install cumulus-netq

4.  Verify the upgrade.
    
        root@ubuntu:~$ dpkg -l | grep netq
        ii  cumulus-netq                      2.2.0-cl3u17~1557345432.a60ec9a    all          This meta-package provides installation of Cumulus NetQ packages.
        ii  netq-agent                        2.2.0-cl3u17~1559681411.2bba220    amd64        Cumulus NetQ Telemetry Agent for Cumulus Linux
        ii  netq-apps                         2.2.0-cl3u17~1559681411.2bba220    amd64        Cumulus NetQ Fabric Validation Application for Cumulus Linux

5.  Configure the NetQ Agent to send telemetry data to the NetQ
    Platform.
    
        user@ubuntu:~# netq config add agent server <netq-platform-ip-address>
        Updated agent server 192.168.1.254 vrf default. Please restart netq-agent (netq config restart agent).

6.  Restart the NetQ Agent
    
        user@ubuntu:~# netq config restart agent

7.  Optionally, configure the Ubuntu server to run the NetQ CLI.
    
        user@ubuntu:~# netq config add cli server <netq-platform-ip-address>
        Updated cli server 192.168.1.254 vrf default. Please restart netqd (netq config restart cli).

8.  Restart the CLI.
    
        user@ubuntu:~# netq config restart cli

9.  Repeat these steps for each switch/host running Ubuntu, or use an
    automation tool to install NetQ Agent on multiple switches/hosts.

### <span id="src-12321007_safe-id-VXBncmFkZWZyb21OZXRRMi4wLzIuMXRvTmV0UTIuMi54LUFnZW50UkhD" class="confluence-anchor-link"></span>Upgrade NetQ Agent on a Red Hat or CentOS Server (Optional)</span>

To install the NetQ Agent on a Red Hat or CentOS server:

1.  Reference and update the local `yum` repository.
    
        root@rhel7:~# rpm --import https://apps3.cumulusnetworks.com/setup/cumulus-apps-rpm.pubkey
        root@rhel7:~# wget -O- https://apps3.cumulusnetworks.com/setup/cumulus-apps-rpm-el7.repo > /etc/yum.repos.d/cumulus-host-el.repo

2.  Edit `/etc/yum.repos.d/cumulus-host-el.repo` to set the `enabled=1`
    flag for the two NetQ repositories.
    
        [cumulus@firewall-2 ~]$ cat /etc/yum.repos.d/cumulus-host-el.repo 
        [cumulus-arch-netq-2.2]
        name=Cumulus netq packages
        baseurl=http://rohbuild03.mvlab.cumulusnetworks.com/dev/rpm/el/7/netq-latest/$basearch
        gpgcheck=1
        enabled=1
         
        [cumulus-noarch-netq-2.2]
        name=Cumulus netq architecture-independent packages
        baseurl=http://rohbuild03.mvlab.cumulusnetworks.com/dev/rpm/el/7/netq-latest/noarch
        gpgcheck=1
        enabled=1
         
        [cumulus-src-netq-2.2]
        name=Cumulus netq source packages
        baseurl=http://rohbuild03.mvlab.cumulusnetworks.com/dev/rpm/el/7/netq-latest/src
        gpgcheck=1
        enabled=1

3.  Update the NetQ meta packages on the server.
    
        root@rhel7:~# yum update cumulus-netq.x86_64

4.  Verify the upgrade.
    
        root@ubuntu:~$ yum list installed | grep netq
        ii  cumulus-netq                      2.2.0-cl3u17~1557345432.a60ec9a    all          This meta-package provides installation of Cumulus NetQ packages.
        ii  netq-agent                        2.2.0-cl3u17~1559681411.2bba220    amd64        Cumulus NetQ Telemetry Agent for Cumulus Linux
        ii  netq-apps                         2.2.0-cl3u17~1559681411.2bba220    amd64        Cumulus NetQ Fabric Validation Application for Cumulus Linux

5.  Configure the NetQ Agent to send telemetry data to the NetQ
    Platform.
    
        root@rhel7:~# netq config add agent server <netq-platform-ip-address>
        Updated agent server 192.168.1.254 vrf default. Please restart netq-agent (netq config restart agent).

6.  Restart the NetQ Agent.
    
        root@rhel7:~# netq config restart agent

7.  Optionally, configure the RHEL/CentOS server to run the NetQ CLI.
    
        root@rhel7:~# netq config add cli server <netq-platform-ip-address>
        Updated cli server 192.168.1.254 vrf default. Please restart netqd (netq config restart cli).

8.  Restart the CLI.
    
        root@rhel7:~# netq config restart cli

9.  Repeat these steps for each switch/host running Ubuntu, or use an
    automation tool to install NetQ Agent on multiple switches/hosts.

## Configure Optional NetQ Agent Settings</span>

Once the NetQ Agents have been installed on the network nodes you want
to monitor, the NetQ Agents must be configured to obtain useful and
relevant data. The code examples shown in this section illustrate how to
configure the NetQ Agent on a Cumulus switch, but it is exactly the same
for the other type of nodes. If you have already configured these
settings, you do not need to do so again.

  - [Configuring the Agent to Use a
    VRF](https://docs.cumulusnetworks.com/display/NETQ/Copy+of+Upgrade+from+NetQ+2.0.x+to+NetQ+2.1.0#CopyofUpgradefromNetQ2.0.xtoNetQ2.1.0-AgentVRF)

  - [Configuring the Agent to Communicate over a Specific
    Port](https://docs.cumulusnetworks.com/display/NETQ/Copy+of+Upgrade+from+NetQ+2.0.x+to+NetQ+2.1.0#CopyofUpgradefromNetQ2.0.xtoNetQ2.1.0-port)

### <span id="src-12321007_safe-id-VXBncmFkZWZyb21OZXRRMi4wLzIuMXRvTmV0UTIuMi54LUFnZW50VlJG" class="confluence-anchor-link"></span>Configure the Agent to Use a VRF Interface</span>

While optional, Cumulus strongly recommends that you configure NetQ
Agents to communicate with the NetQ Platform only via a
[VRF](/display/NETQ21/Virtual+Routing+and+Forwarding+-+VRF), including a
[management VRF](/display/NETQ21/Management+VRF). To do so, you need to
specify the VRF name when configuring the NetQ Agent. For example, if
the management VRF is configured and you want the agent to communicate
with the NetQ Platform over it, configure the agent like this:
<span style="color: #222222;"> </span>

    cumulus@leaf01:~$ netq config add agent server 192.168.1.254 vrf mgmt
    cumulus@leaf01:~$ netq config add cli server 192.168.254 vrf mgmt

You then restart the agent: <span style="color: #222222;"> </span>

    cumulus@leaf01:~$ netq config restart agent
    cumulus@leaf01:~$ netq config restart cli

### <span id="src-12321007_safe-id-VXBncmFkZWZyb21OZXRRMi4wLzIuMXRvTmV0UTIuMi54LXBvcnQ" class="confluence-anchor-link"></span>Configure the Agent to Communicate over a Specific Port</span>

By default, NetQ uses port 8981 for communication between the NetQ
Platform and NetQ Agents. If you want the NetQ Agent to communicate with
the NetQ Platform via a different port, you need to specify the port
number when configuring the NetQ Agent like this:

    cumulus@switch:~$ netq config add agent server 192.168.1.254 port 7379

You then restart the agent: <span style="color: #222222;"> </span>

    cumulus@leaf01:~$ netq config restart agent

<article id="html-search-results" class="ht-content" style="display: none;">

</article>

<footer id="ht-footer">

</footer>
