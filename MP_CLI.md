# Bryce Canyon CLI Users Guide
The BC (Bryce Canyon) Command Line Interface (CLI) utility is a command line program used to make requests via a REST API through the Customer Facing Management Interface (aka CFMI web server) to the Bryce Canyon firmware and associated software. The actual program name is cfmicli to distinguish its use as both a command line utility and its interaction with the CFMI web server.
## Overview
The cfmicli program can be used in multiple modes to make requests in a custom syntax to the Bryce Canyon firmware. The requests generate responses that are provided in JSON format and displayed on the terminal screen.
## Intent
The BC CLI utility is intended to comply with the Bryce Canyon CLI Command Set proposal with variances for implementation dependencies. This implies that while the command set defined by the proposal is the end goal, the actual structure of the commands may vary to support requirements in the actual implementation of the REST API, device drivers and firmware.
## Prerequisites
The BC CLI utility talks to the CFMI web server. Both are maintained in the cfmi GIT repository. The repository must be cloned in order to run the web server and compile and run the BC CLI utility.

`git clone ssh://git@cos-stash-metal:7999/mp/cfmi.git`

### vbm_cfmi
The vbm_cfmi program from the BC repository must be installed in /usr/local/bin, /usr/bin or anywhere in your path. This program should have been built with the BC firmware build.
## CFMI Web Server
The CFMI web server must run on a host with the VBS and Saratoga drivers loaded. The web server is a NodeJS application so the nodejs pac kage must be installed on the host. This can be done with the following commands on Debian systems.

`sudo apt-get update`
`sudo apt-get install -y curl nmap nodejs npm libcurl4-gnutls-dev`

To run the server, use the following commands:

`cd cfmi/server`
`sudo nodejs ./cfmi.js`

or

`cd cfmi/server`
`sudo nodejs ./cfmi.js -v info`

to get additional debug output on the server. The server will print the ports it uses.

_Note: the server must be run as root._
The outpt from this will show the ports used by the web server.
>New log level: info>
>cfmi listening at http://[::]:3123 cfmi listening at https://[::]:3124>

The log level is only displayed if the -v option has been used.
## BC CLI Build
The source for the BC CLI utility is in C and requires compilation. The build system is designed to be easily rebuilt on multiple Linux distributions by use the autoconf suite of tools. The following are required to be installed on a Debian system to build the BC CLI.
### Debian build requirements
`sudo apt-get update
sudo apt-get install autoconf`
Or on CentOS.
### CentOS build requirements
`sudo yum install autoconf libcurl-devel libuuid-devel`

Run the following commands to build the code.

`cd cfmi/cli autoreconf -i`
`./configure make`

Or to get additional debug output when the program is run, use these commands.

`cd cfmi/cli autoreconf -i`

`./configure`

`make CFLAGS="-DDEBUG"`
## CFMI SG Build
The cfmisg utility is the SES counterpart to the vbm_cfmi utility found in the BC repository. However, cfmisg has no dependencies on the BC repository so it is not kept there. This utility makes calls to scsi devices to retrieve SEP enclosure information. Like the BC CLI, the build system for this utility is designed to be easily rebuilt on multiple Linux distributions by use the autoconf suite of tools. The following are required to be installed on a Debian system to build the cfmisg utility.
### Debian build requirements
`sudo apt-get update`

`sudo apt-get install autoconf libsgutils2-dev`

Or on CentOS.

### CentOS build requirements
`sudo yum install autoconf sg3_utils-devel sg3_utils-libs`

Run the following lines to build the code.

`cd cfmi/sg autoreconf -i`
`./configure make`
