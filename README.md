# iRMCtools
Redfish tools and examples for Fujitsu iRMC

# iRMC-Redfish-Tools  

Here are some scripts/examples to automate tasks regarding iRMC (integrated Remote Management Controller) of Fujitsu PRIMERGY servers.
 
The number of tools/scripts will increase over time ...

This toolset is provided W/O ANY WARRANTY and use at your own risk!  

## Requirements

This toolset is intended to be used in Linux environments. Alternatively it can be used in Windows environments with activated WSL (Windows Subsystem Linux) and installed Ubuntu or Debian for example from Microsoft Store. You can also use [Cygwin](https://cygwin.org) (In this case WSL is not needed).

Following commands are required:
- bash (including common tools like awk, sed, grep, ...)
- curl (for talking with iRMC)
- jq (for filtering of data from JSON output)

## Setup

To use this toolset run the following steps:

### 1. Clone the repository

```shell
$ git clone https://github.com/fujitsu/iRMCtools.git
$ cd iRMCtools
```
or extract this [ZIP file](https://github.com/fujitsu/iRMCtools/archive/refs/heads/master.zip).
```shell
$ wget https://github.com/fujitsu/iRMCtools/archive/refs/heads/master.zip
$ unzip master.zip
$ cd iRMCtools-main
```
## Toolset 1 (NIC infos, RAID infos):
Those tools start with "irmc_" in their script names.

### Config file:
`.irmc_env` contains defaults to make things more comfortable.
```shell
# $Id: .irmc_env 134 2022-11-19 08:11:29Z HMBJOrth $
# for documentation see https://github.com/fujitsu/iRMCtools
#
#
# Settings for iRMC tools based on bash and curl
#

# iRMC: IP Addr or name or FQDN of iRMC . Last entry wins ;-)
iRMC=${iRMC:-10.172.124.78}

# USER: iRMC User with appropriate rights (default: administrator)
USER=${iRMC_CRED%:*}
USER=${USER:-admin}                     # if iRMC_CRED not set use defaults

# PASSWD: Password for above USER (default: admin)
PASSWD=${iRMC_CRED#*:}
PASSWD=${PASSWD:-${DCDEFAULTPW:-admin}} # if iRMC_CRED not set use defaults

# CACERT: filename of CA file (doesn't matter if not existant)
CACERT=${0%/*}/DCMA.crt

# Debug files for troubleshooting
HEADERFILE=/tmp/header.log      # HTTPS header
REDFISHFILE=/tmp/redfish.log    # redfish commands
OUTPUTFILE=/tmp/output.log      # redfish output

# DEFAULTOPTIONS: add. options for curl. Can be changed according to network e.g. proxy settings
DEFAULTOPTIONS=" -D $HEADERFILE --silent --noproxy $iRMC"
```

### Used ENV vars:
* `iRMC`: IP-address, name or FQDN of iRMC
* `iRMC_CRED`: User credentials in format `user:password`  
  If this vars are not set like `export iRMC=10.11.12.13` then defaults from file `.irmc_env` are taken.
* `session_id` and `session_token`: These vars are used to handle iRMC sessions. They should be set/unset with commands like `eval $(irmc_login)` or `eval $(irmc_logout)`.
* `DEBUG`: If set (e.g. `export DEBUG=1`) the scripts will output all commands to `stderr`. Additional all native redfish commands are logged to `$REDFISHFILE` which is defined in `.irmc_env`, too.
* `WARNING`: If set a warning message appears when https data is not confirmed by certificate.

### Available commands:
* `irmc_showenv`: Display the current environment: 
```shell
$ irmc_showenv
iRMC:            10.172.124.78
iRMC_FQDN:       10.172.124.78
iRMC_IP:         10.172.124.78
USER:            admin
PASSWD:          admin
session_id:
session_token:
```

* `irmc_chcon`: This command check if you can run Redfish commands.
```shell
$ # Here OK
$ irmc_chcon
Connection to 10.172.124.78 (via user/password) OK

$ iRMC_CRED=admin:IdontKnow
$ # Here: Error
$ irmc_chcon
HTTP/1.1 401 Unauthorized
Connection to to 10.172.124.78 (via user/password) not possible (HTTP/1.1 401 Unauthorized)
```

* `irmc_login`: Used for initiating an iRMC session and setting of the required ENV vars `session_id` and `session_token`. Usage: `eval $(irmc_login)`. With an established session there is no need for authentication overhead when doing several requests in a row. The performance factor is up to two! 

* `irmc_logout`: Used for destroying an iRMC session and unsetting the session related ENV vars. Usage: `eval $(irmc_logout)`

* `irmc_cmd`: Basic command to perfom Redfish tasks: Usage: `irmc_cmd get|post|patch|delete redfish_cmd [other options ..]`. You can use redfish_cmd w/ or w/o leading "/". You can also use the full name like "/redfish/v1/Systems/0". But, of course, it's less typing using only "Systems/0". Example: 
```shell
* $ irmc_cmd get Systems/0
{
  "@odata.id":"\/redfish\/v1\/Systems\/0",
  "@odata.type":"#ComputerSystem.v1_4_10.ComputerSystem",
  "Oem":{
    "ts_fujitsu":{
      "@odata.type":"#FTSComputerSystem.v2_10_0.FTSComputerSystem",
      "FirmwareInventory":{
        "@odata.id":"\/redfish\/v1\/Systems\/0\/Oem\/ts_fujitsu\/FirmwareInventory"
      },
      "MainBoard":{
        "Manufacturer":"FUJITSU",
        "Model":"D3890",
        "SerialNumber":"SM2137PNB00I",
:
:
:
```
* `irmc_shownic`: Display NIC (Network Interface Controller) related information.
```shell
$ irmc_shownic
#########################################################
PRIMERGY RX2540 M6 rx2540m6-4-81.bupc-test.hmb.fsc.net 10.172.124.82
#########################################################

LAN MAC addresses:

MAC               Ctrl Port Link     Name
================= ==== ==== ======== =========================================
68:05:CA:CF:75:EC    0    0 LinkUp   PLAN CP I350-T4 4X 1000BASE-T OCPv3
68:05:CA:CF:75:ED    0    1 LinkUp   PLAN CP I350-T4 4X 1000BASE-T OCPv3
68:05:CA:CF:75:EE    0    2 LinkDown PLAN CP I350-T4 4X 1000BASE-T OCPv3
68:05:CA:CF:75:EF    0    3 LinkDown PLAN CP I350-T4 4X 1000BASE-T OCPv3
40:A6:B7:3F:59:44    1    0 null     PLAN EP X710-DA2 2x10Gb SFP
40:A6:B7:3F:59:45    1    1 null     PLAN EP X710-DA2 2x10Gb SFP
40:A6:B7:7C:CB:10    3    0 null     PLAN EP X710-DA2 2x10Gb SFP
40:A6:B7:7C:CB:11    3    1 null     PLAN EP X710-DA2 2x10Gb SFP


PCI-Cards slot mapping ...

Slots:
======
PLAN CP I350-T4 4X 1000BASE-T OCPv3       OCP : 1
PLAN EP X710-DA2 2x10Gb SFP               PCI Slot : 1
PFC EP LPe31002                           PCI Slot : 7
PLAN EP X710-DA2 2x10Gb SFP               PCI Slot : 5
```
* `irmc_showraid`: Display storage controllers and respective RAID configuration:
```shell
$ irmc_showraid
#########################################################
PRIMERGY RX2530 M6 rx2530m6-4-77.bupc-test.hmb.fsc.net
#########################################################

Storage-Controller (0):
    PRAID EP540i (0) ControllerNumber=534 Status=OK
        Disks:
            SEAGATE XS1600LE70084 (0) Size=1490 GiB (1600 GB) Status=OK
            SEAGATE XS1600LE70084 (1) Size=1490 GiB (1600 GB) Status=OK
            SEAGATE XS1600LE70084 (2) Size=1490 GiB (1600 GB) Status=OK
            SEAGATE XS1600LE70084 (3) Size=1490 GiB (1600 GB) Status=OK
        Volumes:

Storage-Controller (1):
    PDUAL CP100 (1) ControllerNumber=239632 Status=OK
        Disks:
            MICRON 5300 MTFDDAV240TDS (0) Size=224 GiB (240 GB) Status=OK
            MICRON 5300 MTFDDAV240TDS (1) Size=224 GiB (240 GB) Status=OK
        Volumes:
            ESXi7Boot (RAID1) Size=224 GiB (240 GB) Status=OK
                Disk number 0
                Disk number 1
```
It's also possible to configure new volumes and so on. But those actions must be done with care to prevent data loss. In such cases you can run a command like `irmc_cmd post Systems/0/Storage/1/Volumes -d "@NewVolumeCreateSettings.json" -i | head -1`. Please check the Redfish API Spec v3.39, Chapter "CreateVolumes on volume collection", pages 111 and following.

## Toolset 2
This tool is mentioned to mount an ISO image via NFS, CIFS, or HTTP as remote media in order to boot a server with with image. This can be used for OS installation as well as for applying Fujitsu Update DVD.

###  Edit the configfile

```shell
# CFG for isomount
# $Id: isomount.cfg 112 2022-07-19 14:33:05Z HMBJOrth $ #
#
# Example of URIs (Possible are HTTP, SMB/CIFS and NFS)
# Uri='smb://domain;user:password@server/share/folder/file.iso'
# Uri='nfs://user:group@server/export/test/file.iso'
# Uri='nfs://server:port/export/test/file.iso'
# Uri='nfs://server/export/test/file.iso'
# Uri='http://server/path/file.iso'

# CHANGE lines below
iRMC=${1:-10.172.126.245} User=${2:-admin} Pw=${3:-admin}
Uri='http://10.172.125.9/ISO/VMware-ESXi-6.7.0-14320388-Fujitsu-v480-1.iso'
```

Set the variables `iRMC`, `User`, `Pw` and `Uri` by editing isomount.cfg. 
Alternatively you can override the first three vars when calling `isomount`. There are examples of URIs for http/nfs/smb.

Set permissions:

```shell
$ chmod go-rwx isomount.cfg	  # for security
$ chmod +x isomount           # if necessary
```

## Commands
### 1. isomount 
  
#### Usage: `isomount  [ <IPaddress or dnsname of irmc> [<username> [<password>]]]`

Important: The server has to be shutdown before running this tool. This is checked by the tool.  

Hint: Directory of  `isomount` should be in `$PATH` of course!

#### Functionality:
- Check prerequisites (jq availabe / server off)
- Mount ISO
- Change boot device for next boot only to ISO
- Power on server (in order to update)
- Unmount ISO after user acknowledge

#### Debugging:

For evaluation purposes it is possible to run `isomount` in debug mode. For this enable debugging (before calling `isomount`) with:
```shell
$ export DEBUG=1
```
If debugging is enabled all Redfish calls and outputs are loggend in file `/tmp/isomount-<$iRMC>.log`. Disable debugging with:
```shell
$ unset DEBUG
```

#### Running `isomount` in parallel:

You can use this tool to update several servers in parallel:

So with an example input file for iRMC addresses, users, passwords like:
```shell
myirmc
172.25.47.11 smith
server13.mycompany.com admin veryscrecetpassword
```
you can use it like:
```shell
$ while read irmc user password
> do
>  echo "Processing $irmc ..."
>  nohup isomount $irmc $user $password &
> done < <inputfile>
```

Further links to documents / API specifications and so can you find [here](https://github.com/JuergenOrth/primergy).
