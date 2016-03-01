---
layout: post
date: 2014-07-23
author: Marc van Eijk
title: Windows Azure Pack configuration with a named instance and a non-default SQL port
tags: Marc van Eijk, Mixed mode, Named Instance, SQL, SQL Authenitcation, SQL Port, Windows Azure Pack
---
Installing Windows Azure Pack in a lab environment is relatively easy. You control all the environment variables. Changes to operating systems, required permissions or other settings are within your own hands.

How different is this when you implement Windows Azure Pack in a Service Provider or Enterprise Organization environment. All kind of security requirements are in place. Each change in the environment is preceded by a request for change procedure. Planning, prerequisites and design documents are essential from the start of the project. If you have not invested in these upfront you will find yourself confronted with a new change each time that, in its turn, results in another RFC with accompanying handling time. Within a couple of days your teeth marks will be visible in the steering wheel of your car.

(Previously) a prerequisite for Windows Azure Pack:
- Windows Azure Pack requires a SQL server running in mixed authentication mode and the SQL instance must be running on the default SQL port 1433

But the security policy at the Enterprise Organization or Service Provider dictates:
- The SQL Server must be in Windows Authenticated mode only using a named instance and non-default SQL port

Most deployments start with an installation in a development environment that reflect the production environment. In the development environment the SQL configuration that is required for Windows Azure Pack is tolerated but flagged. Once we move to production the RFC can possibly block the implementation.

I contacted the folks from the Windows Azure Pack program group a couple of months ago. They provided the means to configure Windows Azure Pack with a named instance and against a non-default SQL port. It was OK to use this configuration for the development environments, but they needed some additional testing to validate that this configuration would not break in hotfix or upgrade scenarios.

## Named instance and non-default SQL port

It is now supported to configure Windows Azure Pack with a named instance and a non-default SQL port. Configure the database connection in the configuration wizard with the following format.

```
<SQL Server>\<Instance>,<Port number>
```

In this example

```
SQL01\hypervu,10001
```

<img src="/images/2014-07-23/07-Commit.png" width="700">

## SQL Authentication

Windows Azure Pack does still require a SQL Server in mixed authenticated mode. During the installation SQL accounts are created that are used in the encrypted part of the web.config file of each Windows Azure Pack website. But if this SQL authentication discussion comes up in a project consider this:

A long time ago, Microsoft [recommended](http://technet.microsoft.com/en-us/library/aa905171(v=SQL.80).aspx) “When possible, use Windows Authentication” for SQL databases. That recommendation was not based on security issues with SQL authentication. It was a best practice for applications which would work better with pass through user authentication rather than using a service principle. That statement was interpreted by most organizations with “you should never use SQL authentication”.

Is that statement (still) true or is SQL authentication a secure choice?

The best example of using SQL authentication for databases is Microsoft itself. Microsoft Azure SQL Database (database-as-a-service) [only supports SQL Server Authentication](http://msdn.microsoft.com/en-us/library/ff394108.aspx#authentication). Windows Authentication (integrated security) is not supported.

If Microsoft wasn’t comfortable using SQL authentication, they wouldn’t run a few hundred thousand SQL servers. That are on the internet!!

Image taken from the new Microsoft Azure Portal that is in preview.

<img src="/images/2014-07-23/07-Commit.png" width="700">

## More Information

- [Windows Azure Pack, SQL Always On, Listener and Port story](http://www.vnext.be/2014/06/13/windows-azure-pack-sql-always-on-listener-and-port-story/)
- [Microsoft SQL Server versions supported in a Windows Azure Pack deployment](http://technet.microsoft.com/en-us/library/dn469343.aspx)
