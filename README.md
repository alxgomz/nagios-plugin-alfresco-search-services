# Nagios plugin for Alfresco Search Services

This plugin allows to monitor some advanced metrics of Alfresco's Solr server.
It is known to work with Alfresco 5.x using Solr4 and Solr6 (Alfresco search Services)

#### This is work in progress and still suffers from many unhandled errors and has room for improvement.

## Installation

### Prerequisites:

The system must have python 2.7 or higher (should work with python3). This plugin uses a python library for Nagios plugins which needs to be installed.

On Debian-like systems the following should work:

```
$ sudo apt install python-nagiosplugin
```

> Or python3-nagiosplugin for python3 systems

Otherwise simply install it with pip

```
$ pip install nagiosplugin
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

