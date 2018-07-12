# Nagios plugin for Alfresco Search Services

This plugin allows to monitor some advanced metrics of Alfresco's Solr server.
It is known to work with Alfresco 5.x using Solr4 and Solr6 (Alfresco search Services)

#### This is work in progress and still suffers from many unhandled errors and has room for improvement.

## Installation

### Prerequisites:

The system must have python 2.7 or higher (should work with python3).
This plugin uses a python library for Nagios plugins and urllib3 which both need to be installed.

On Debian-like systems the following should work:

```
$ sudo apt install python-nagiosplugin python-urllib3
```

> Or python3-urllib3 & python3-nagiosplugin for systems sing by default python3.

Otherwise simply install it with pip

```
$ sudo pip install nagiosplugin
$ sudo pip install urllib3
```

### Plugin deployment

Simply copy the file check_alfresco_solr.py to the nagios plugin directory usually something like `/usr/lib/nagios/plugins`.

## Configuration

Once deployed, add the following to your Nagios configuration.

* a check command:

> $NAGIOSPLUGINS$/check_alfresco_solr.py --host $HOSTADDRESS$ --port $_HOSTSOLRPORT$ --scheme $_HOSTSOLRSCHEME$ --admin $_HOSTSOLRADMINURL$ --monitor $_SERVICESOLRMONITOR$ --item "$_SERVICESOLRMONITORITEM$" -w $ARG1$ -c $ARG2$ $ARG3$  
with:  
ARG1 : WARNING threshold  
ARG2 : CRITICAL threshold  
ARG3 : Solr Core name  

> MACRO (SERVICE) SOLRMONITOR : type of item to monitor (see check_alfresco_solr.py --help)  
MACRO (SERVICE) SOLRMONITORITEM : item to monitor (see check_alfresco_solr.py --help)  
MACRO (HOST) SOLRPORT : TCP port where Solr is listening (should be 8983 for alfresco 5 and newer and 8443 for alfresco 4)  
MACRO (HOST) SOLRADMINURL : URL for the core admin  
MACRO (HOST) SOLRSCHEME : HTTP scheme used by Solr server (http or https)

* appropriate template services:

> For caches,  
SOLRMONITOR: caches  
SOLRMONITORITEM: one of /alfrescoPathCache, /alfrescoAuthorityCache, /queryResultCache, /filterCache  

> For handlers,  
SOLRMONITOR: handlers  
SOLRMONITORITEM: one of /alfresco, /afts, /cmis  

> For index,  
SOLRMONITOR: index  
SOLRMONITORITEM: one of Alfresco Acls in Index, Alfresco Nodes in Index, Alfresco Transactions in Index, Alfresco Acl Transactions in Index, Alfresco States in Index, Alfresco Unindexed Nodes, Alfresco Error Nodes in Index, numDocs, segmentCount, size  

> For fts,  
SOLRMONITOR: fts  
SOLRMONITORITEM: one of New, Clean, Dirty

* an host template linked to all the service templates created above

* and finally a host with the macros bellow:

> SOLRPORT  
SOLRSCHEME  
SOLRADMINURL

## Usage

### Command line

```
usage: check_alfresco_solr.py [-h] [--host HOST] [--port PORT]
                              [--scheme SCHEME] [--sslcert SSLCERT]
                              [--sslkey SSLKEY] [--admin ADMIN] [--fix]
                              --monitor {fts,index,handlers,caches}
                              [-w WARNING] [-c CRITICAL] [--item ITEM]
                              core

Monitor different parts of Solr. The following can be monitored:
        - index:    A group of item regarding index status. warning (-w) and critical (-c) command line parameters are
                    applied to the item specified with --item
                    Available items: Alfresco Acls in Index,
                    Alfresco Nodes in Index, Alfresco Transactions in Index,
                    Alfresco Acl Transactions in Index,
                    Alfresco States in Index, Alfresco Unindexed Nodes,
                    Alfresco Error Nodes in Index, numDocs, segmentCount,size

        - fts:      warning (-w) and critical (-c) command line parameters are
                    applied to the item specified with --item
                    Available items: New, Clean, Dirty

        - handlers: Will return CRITICAL if the handler report any error,
                    WARNING in case of timeouts. warning (-w) and critical (-c)
                    command line parameters are applied to the average request
                    time for that handler.
                    Available items: /alfresco, /afts, /cmis

        - caches:   warning (-w) and critical (-c) command line parameters
                    apply to the hitratio of the cache.
                    Available items: /alfrescoPathCache, /alfrescoAuthorityCache,
                    /queryResultCache, /filterCache


positional arguments:
  core                  Specify the name of the Solr core

optional arguments:
  -h, --help            show this help message and exit
  --host HOST           hostname or IP address of the Solr service
  --port PORT           port number of the Solr service
  --scheme SCHEME       protocol scheme of the Solr service
  --sslcert SSLCERT     file containing the SSL client certificate if any
  --sslkey SSLKEY       file containing the client private key if any
  --admin ADMIN         Cores admin URL of the Solr service
  --fix                 Attempt a FIX action if error nodes are detected
  --monitor {fts,index,handlers,caches}
                        Specify what Solr part (group of items) we want to monitor
  -w WARNING            Warning threshold for Nagios style monitoring
  -c CRITICAL           Critical threshold for Nagios style monitoring
  --item ITEM           Name of item to monitor (see above for details)
```

### SSL enabled Solr

By default Solr4 on Alfresco 5 is configured with SSL enabled and mutual authentitcation. If that's your case you will needs to use a client certificate file and its associated private key.
The default one, **which you should not be using in real life environements**, are provided [here](ssl/) for convenience and can be used with the command lineorameters bellow:

```
  --sslkey ssl/key_solr_client --sslcert ssl/cert_solr_client
```

As Alfresco recommands setting up your own certificate and discourages the use of the default one those ceertificates provided here won't probably work.
Bellow is an example of how to generate proper key and certificate files from your own p12 (generated from your own pki for instance):

```
$ openssl -in solr_client.p12 -nocerts -nodes -out ssl/key_solr_client
$ openssl -in solr_client.p12 -nokeys -out ssl/cert_solr_client
```

> Here we use unencrypted key file for automation purposes (`-nodes`). Make sure you take appropriate actions to make it as secure as possible (not world readable, appropriate owner only, ...)
