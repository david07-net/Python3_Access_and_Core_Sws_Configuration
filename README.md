# Python3_Access_and_Distribution_and_Core_Sws_Configuration

This is a tutorial on how to connect and configure 3 Access switches and Core/Distribution switches. For this tutorial we are using the library knows at [Netmiko](https://github.com/ktbyers/netmiko). Please feel free to go in the link for more information. 

## Requirements

1. Have Python3 and Netmiko install. 
2. We well need to configure manually the following parameters: SSH connection, username, password, mangement IP address. 

### Configuring the Cisco Devices

For this tutorial we will be using the out of band management port to configure the devices. I this is more easy in case we go with in-band manament connection and we end up affecting out ssh connection.

Also, at this point for this tutoring we are just configuring the Access switches and Distribution Switches. Enventually, I will create one tutorials with core switches, cisco routers, and cisco ASAs.

We start configuring the basic settings on each switch so we can establish an ssh connection to the switches.

#### For Access Switch 1

```
!
hostname ASw1
!
!
enable secret cisco
!
username david priv 15 pass cisco
!
ip domain-name pythonccie.com
!
ip domain-lookup
!
crypto key generate rsa modulus 2048
!
ip ssh v 2
!
int g0/0
no switchport
ipp add 192.168.10.11 255.255.255.0
no shut
description OUT_MGMT_PORT
!
!
line vty 0 4
transport input ssh
login local
!
```

#### For Access Switch 2

```
!
hostname ASw2
!
!
enable secret cisco
!
username david priv 15 pass cisco
!
ip domain-name pythonccie.com
!
ip domain-lookup
!
crypto key generate rsa modulus 2048
!
ip ssh v 2
!
int g0/0
no switchport
ipp add 192.168.10.12 255.255.255.0
no shut
description OUT_MGMT_PORT
!
!
line vty 0 4
transport input ssh
login local
!
```

#### For Access Switch 3

```
!
hostname ASw1
!
!
enable secret cisco
!
username david priv 15 pass cisco
!
ip domain-name pythonccie.com
!
ip domain-lookup
!
crypto key generate rsa modulus 2048
!
ip ssh v 2
!
int g0/0
no switchport
ipp add 192.168.10.13 255.255.255.0
no shut
description OUT_MGMT_PORT
!
!
line vty 0 4
transport input ssh
login local
!
```

#### For Distribution Switch 1

```
!
hostname DSw1
!
!
enable secret cisco
!
username david priv 15 pass cisco
!
ip domain-name pythonccie.com
!
ip domain-lookup
!
crypto key generate rsa modulus 2048
!
ip ssh v 2
!
int g0/0
no switchport
ipp add 192.168.10.21 255.255.255.0
no shut
description OUT_MGMT_PORT
!
!
line vty 0 4
transport input ssh
login local
!
```

#### For Distribution Switch 2

```
!
hostname DSw2
!
!
enable secret cisco
!
username david priv 15 pass cisco
!
ip domain-name pythonccie.com
!
ip domain-lookup
!
crypto key generate rsa modulus 2048
!
ip ssh v 2
!
int g0/0
no switchport
ipp add 192.168.10.22 255.255.255.0
no shut
description OUT_MGMT_PORT
!
!
line vty 0 4
transport input ssh
login local
!
```

I used this commands because I think are the basic ones that you configure in any access or distribution switch for out-of-band management. But We can always add as many commands as we need assuming that are applicable to all access switches. I saved the file in the same directory as the code so it the path to the file is not long you need to be careful with that.

### Commands on the Access Switch File

```
!
!# This is the commands for access switches
!
vlan 10
name Trunk_VLAN
!
Vlan 11
name DATA_VLAN
!
vlan 12
name PHONE_VLAN
!
!
!
! Configuring the Data ports
!
interface range g0/1-3, g1/0-3
desc DATA_Network
switchport host
switchport access vlan 11
no shutdown
!
! Configuring the phone ports
!
interface range g2/0-3
desc PHONE_Network
switchport host
switchport access vlan 12
no shutdown
!
! Configuring trunking ports
!
interface range g3/0-3
desc TRUNK_PORTS
switchport trunk encapsulation dot1q
switchport mode trunk
switchport trunk native vlan 10
switchport trunk allowed vlan 10,11,12
no shutdown
!
```

### Commands for the Distribution Switches

```
!
!# These are the commands for Distribution Switches
!
!# Enable IPv4 Routing
!
ip routing
!
!
!# Creating VLANs
!
vlan 10
name Trunk_VLAN
!
Vlan 11
name DATA_VLAN
!
vlan 12
name PHONE_VLAN
!
!
!
!
!
! Configuring trunking ports
!
interface range g0/1-3,g1/0-3,g2/0-3
switchport trunk encapsulation dot1q
switchport mode trunk
switchport trunk native vlan 10
switchport trunk allowed vlan 10,11,12
no shutdown
!
!
interface range g2/0-3
desc PORTCHANNEL_BTW_SW
channel-group 1 mode active
!
interface range g3/1-2
desc PORTCHANNEL_UPLINK
channel-group 2 mode active
!
!
```

### Python Scripts

```Python
#
# We import getpass modules for password protection
# We import Netmiko for SSH
#
from getpass import getpass
from netmiko import ConnectHandler

#
# We open the file where the commands are for
# the  Access Switches.
#
with open('cisco_Access_SW') as f:
    commands_Access_switch = f.read().splitlines()

#
# We open the file where the commands are for
# the Distribution switches.
#
with open('cisco_Distribution_SW') as f:
    commands_Distribution_switch = f.read().splitlines()
#
# We open the file  where the hostname of the
# devices are stored.
#
with open('hostname_file') as f:
    hostname_list = f.read().splitlines()
#
# We open the file where the IP address of the
# devices are stored.
#
with open('device_file') as f:
    devices_list = f.read().splitlines()
#
# Asking for Username and password
# instead of having it on the file
#
username = input('Enter your SSH username: ')
password = getpass()
#
#
# Creating a for loop for each  device in the list
#
for devices in devices_list:
    print ('Connecting to device" ' + devices)
    ip_address_of_device = devices
    # Creating a device dictionary
    ios_device = {
        'device_type': 'cisco_ios',
        'ip': ip_address_of_device,
        'username': username,
        'password': password
    }


    # Connecting to the device

    net_connect = ConnectHandler(**ios_device)

    # Creating a  for loop for each device connected
    # to see which command they should get

    for device_name in hostname_list:
        print("Checking for " + device_name)
        output_name = net_connect.send_command('show run | include hostname')
        int_name = 0 # Reset value
        int_name = output_name.find(device_name) # Check name of the device
        if int_name > 0:
            print ('Device Name found: ' + device_name)
            break
        else:
            print ('Did not find ' + device_name)

    if int_name == 'DSw1' or 'DSw2' :
        print ('Running ' + device_name + ' commands')
        output = net_connect.send_config_set(commands_Distribution_switch)
        print (output)
    elif int_name == 'ASw1' or 'ASw2' or 'ASw3':
        print ('Running ' + device_name + ' commands')
        output = net_connect.send_config_set(commands_Access_switch)
        print (output)
    else:
        print(" For the Device  " + int_name + "\n We dont have commands \n")
```
