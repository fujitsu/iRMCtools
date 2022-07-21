# iRMCtools
Redfish tools for Fujitsu iRMC

# iRMC-Redfish-Tools  

Here is a script to automatically mount some Update-DVD-ISO-Image using the Redfish API of Fujitsu iRMC in order to update Fujitsu PRIMERGY servers.
 
The number of tools/scripts will increase over time ...

This toolset is provided W/O ANY WARRANTY and use at your own risk!  

## Requirements

This toolset is intended to be used in Linux environments. Alternatively it can be used in Windows environments with activated WSL (Windows Subsystem Linux) and installed Ubuntu or Debian for example from Microsoft Store. You can also use [Cygwin](https://cygwin.org) (In this case WSL is not needed).

Following commands are required:
- bash
- curl (for talking with iRMC)
- jq (for filtering of data from JSON output)

## Setup

To use this toolset run the following steps:

### 1. Clone the repository

```shell
$ git clone https://github.com/JuergenOrth/iRMCtools.git
$ cd iRMCtools
```
or extract this [ZIP file](https://github.com/JuergenOrth/iRMCtools/archive/refs/heads/master.zip).
```shell
$ wget https://github.com/JuergenOrth/iRMCtools/archive/refs/heads/master.zip
$ unzip master.zip
$ cd iRMCtools-master
```

### 2. Edit the configfile

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

