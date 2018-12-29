---
layout: post
title: Apache 2.4 Connector for Crowd and RHEL7
---

## TL;DR
If you need to build the [Apache/Subversion Crowd Connector](https://confluence.atlassian.com/crowd031/integrating-crowd-with-apache-949753124.html) for Apache 2.4 on Red Hat Enterprise Linux 7 (RHEL7 for short), you can do it with [cwdapache-rhel7](https://github.com/ferstl/cwdapache-rhel7), which is a fork of the original connector.
It probably does also work for other Linux distributions shipping with Apache httpd 2.4.

## Slightly More Detailed Version
We are using the [Apache/Subversion Crowd Connector](https://confluence.atlassian.com/crowd031/integrating-crowd-with-apache-949753124.html) to authenticate users for internal web sites and some legacy Subversion repositories with [Atlassian Crowd](https://www.atlassian.com/software/crowd).

The connector was originally provided by Atlassian for Apache httpd 2.2. However, by the end of 2014, Atlassian decided to discontinue development and maintenance of the connector and open sourced it on [Atlassian Labs](https://bitbucket.org/atlassianlabs/cwdapache.git).

As we had to migrate all our Linux servers from SLES11 (comes with httpd 2.2) to RHEL7 (comes with httpd 2.4), we had the problem that the connector was not working anymore due to API changes in httpd.

 The original Git repository of the connector contains a [Pull Request](https://bitbucket.org/atlassianlabs/cwdapache/pull-requests/18/added-apache-24-compatibility-and-fixed/diff) that adds support for Apache httpd 2.4. However, since Atlassian has stopped supporting the connector, the PR was never merged. In the meantime, the repository containing the code changes was deleted and the changes were lost.
 
 Fortunately, @fgimian has another fork of the original repository on [GitHub](https://github.com/fgimian/cwdapache.git) which contains the code changes of the PR above and some additional changes. Unfortunately, with the additional changes it was not possible to build the connector on RHEL7.
 
 This is the reason why the [cwdapache-rhel7 Fork](https://github.com/ferstl/cwdapache-rhel7) of the connector came to live. It is based on the the original repository on Atlassian Labs and contains exactly the changes for httpd 2.4 and two slight modifications in the build setup to build on RHEL7. Additionally, it contains two Docker files to build the connector without having a running RHEL7 instance.

A detailed description of how to build the connector on RHEL7 and more details can be found in the [README.md](https://github.com/ferstl/cwdapache-rhel7/blob/master/README.md) of the forked repository.

## Links

- [cwdapache-rhel7 Fork](https://github.com/ferstl/cwdapache-rhel7)
- [Original Atlassian Labs Repository](https://bitbucket.org/atlassianlabs/cwdapache.git)
- [@fgimian's GitHub Repository](https://github.com/fgimian/cwdapache.git)
- [Original PR for httpd 2.4](https://bitbucket.org/atlassianlabs/cwdapache/pull-requests/18/added-apache-24-compatibility-and-fixed/diff)
- [Integrating Crowd with Apache](https://confluence.atlassian.com/crowd031/integrating-crowd-with-apache-949753124.html)
