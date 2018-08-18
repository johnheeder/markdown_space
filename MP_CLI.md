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

The server will print the ports it uses.

_Note: the server must be run as root._

The outpt from this will show the ports used by the web server.
>New log level: info>

>cfmi listening at http://[::]:3123 cfmi listening at https://[::]:3124>

The log level is only displayed if the -v option has been used.
## BC CLI Build
The source for the BC CLI utility is in C and requires compilation. The build system is designed to be easily rebuilt on multiple Linux distributions by use the autoconf suite of tools. The following are required to be installed on a Debian system to build the BC CLI.
### Debian build requirements
`sudo apt-get update`

`sudo apt-get install autoconf`
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
### CentOS build requirements
`sudo yum install autoconf sg3_utils-devel sg3_utils-libs`

Run the following lines to build the code.

`cd cfmi/sg autoreconf -i`

`./configure make`

To get additional debug output when the program is run, use these commands.
`cd cfmi/sg autoreconf -i`

`./configure`

`make CFLAGS="-DDEBUG"`

After building the cfmisg utility must be placed under the /usr/local/bin directory.
`sudo cp cfmi/sg/src/cfmisg /usr/local/bin`

##Basic command structure
BC CLI commands follow a common syntax:

>command object fields...

A command is an action to take on the object such as show, create or update. The object is an BC defined component such as storage, host or drive.  The fields describe and expand the command.  They provide metadata required to complete the command action.
##Basic Usage
The BC CLI utility is run from a terminal command line. It supports a variety of command line options which can be viewed using the -? option. First, change to where the executable lives.

`cd src`

Now you can see the help message.
`./cfmicli -?`

The current (at the time of this writing) output from this command looks like the following.

`cfmicli`

`Version: 0.1.0 - 20160907:1409MDT`

`cfmicli [ -h?apstT -c file -h host -P port -p pw -u userid | -l `

###Specifying the remote server
The BC CLI connects to a CFMI web server. The CFMI web server can be on the same host as the BC CLI utility or on a remote host.  The BC CLI utility has command line options to specify where to connect to the CFMI web server. The following examples show the BC CLI being used to connect to a CFMI web server on the local host at the default port and two different ways to connect to a remote server on the default port.

`./cfmicli -h localhost -P 3123`

`./cfmicli -h myhost.someplace.com -P 3123`

`./cfmicli -h 192.168.10.55 -P 3123`

###Authentication
Every interaction between the BC CLI and the CFMI web server requires authentication. The BC CLI accepts authentication information using command line options.

`./cfmicli -u admin -p admin`

###Cached credentials
Remote server, port, user id and password credentials are stored in $HOME/.cfmicli. This allows the user to specify these values once and then not have to respecify with each use of the single command or interactive modes (modes are described in the next section). The content of the cache file is human readable.

`$ cat ~/.cfmicli host:localhost port:3123 userid:admin pw:admin`

##Output Formats
The -f option to the cfmicli command provides for data translation of the JSON data returned from the upstream server. The default is to print the JSON data in an uncompressed, easy to read plain text format. Some JSON fields returned from the server may include escaped newlines. To convert those escaped newlines to real newlines use the -f pretty option.
##Operational modes
There are three modes for using the BC CLI:
*interactive*
*command file*
*single command*
###Single commands
Single commands are provided by the -a option.  The command is wrapped in double quotes, as in the following example.

`./cfmicli -a "show enclosure"`

Output is printed directly on the console.

###Command file
Single or multiple commands can be specified in a command file. Any line of the command file that starts with a hash tag (#) is considered a comment and ignored by the command interpreter. Blank lines are also ignored. Any non-blank line that does not begin with a hash tag is considered a command to perform. The first four lines are the credentials. No blank lines or comments are allowed in the first four lines but the order of the credentials is not fixed.
Each command in the file is run, regardless of the result of any previous command.  The command file is specified using the -c option.

`./cfmicli -c ../data/sample.cmd`

Output is printed directly on the console.
