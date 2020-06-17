# SQL Data Sucker

## Overview

If we have a large number of production databases, or even just one, we may choose to Extract the data, Load the Data into a Reporting Database, and then Transform the Data (ELT).  In the cloud we have an array of tools to choose from.  On Premise environments might not have this option.  Introducing SQLDataSucker.  Selectively suck data from one or more SQL Servers into a central SQL database for reporting.  Build up a Data Catalog for your Report Developers.  Configure database objects to Sync and customize the scheduling.  Choose a custom update method.  Automatically assign permissions based on rules and more!

## Why use it?

Let's start from the beginning.  Allowing Report Developers to connect directly to Production Databases may be convenient but eventually you may find yourself in what some call Database Hell.  This is especially true if you manage multiple servers + potentially 100s of databases.  The impacts of allowing developers to connect directly to Production Databases will eventually show up as reports of degraded application performance, "degreaded" (..degraded reads) report performance, data sources in more holes than a chipmonk can dig, and the one thing no DBA wants to hear about, loss of trust when it comes to data security.  The solution is simple.  Copy your Production Data into a Reporting Database.  It's unavoidable.

So now that we have a basic premise for syncing data, we can ask:

1. How much data?
2. How often?
3. What technology/method to use?

Database replication?  If you have 100 servers there is a high chance you might not even be allowed to make changes on some of them.  Replication requires touching stuff that could break stuff.  You also don't have time to be making changes on 100 servers, and keeping track of things that other people have changed in your absence.  Central Configuration should be maximized and Remote Configuration should be minimal.  Either it works or it doesn't, and if it doesn't it shouldn't take more than 10 minutes to understand why and fix it.

Full Database Backup Restore?  Doesn't work if you have a +10GB database.  Even if you ran transaction log backups every minute, you're asking for trouble.  Not to mention, if the backup is done on a higher version of SQL Server than you central database server, you won't be able to restore it.  And you still have 100 separate databases at the end of the day...perhaps for the sake of 100 tables.  And you're killing your hardware by unecessary I/O.

Third party tool?  Haven't seen one I like.  Maybe I haven't looked hard enough but I'm not asking for much so I figured I'd just build one.

## Features Required

<in the Project Board>

## Terminology

### Industry Terms
- Report Viewer - The people who view reports.
- Report Developer - The people who develop reports.
- Data Scientist - The Report Developers who enjoy math in particular.
- Data Engineer - The guy who made this.
- Data Catalog - All database names, object names, and meta data.  No sample data.  Used by Report Developers to develop a Data Dictionary.
- Data Dictionary - Business definitions applied to the Data Catalog.  Identification of data.
- Sample Data - A subset of data from an object in Data Catalog to better identify what data exists.  Useful if you don't necessarily need to pull all the data in a large table.
- SQL Linked Servers - SQL Server technology/feature for connecting to remote servers.

### SQL Data Sucker Terms
- Sync Method
- Sync Schedule - There are set of Sync Schedules, for example, hourly, working days, weekends, nights, every 15 minutes.  Each object has a matching Schedule ID so the Sync Proc knows when to sync it.
- Sync Proc - Runs anytime a SQL Agent Schedule calls it.  Runs against all objects with a matching Schedule ID.
- Linked Servers - Used to connect to all Production Database Systems.
- AutoPermissions - Grant access to report database views based on key phrases - aka dynamic rules.
-Sync Objects - All the objects we sync/copy to the reporting datbase.

## Moving Parts / Underlying Assumptions

1. At any time, a production database, its schema, or data can be deleted or changed.  This can screw up reports.  This solution doesn't attempt to fix that.  Remapping Source Data to a Business Layer is just a part of life as a Report Developer / Data Scientist.  If source data is renamed or deleted, we simply leave the last copy intact on the Reporting Database and mark the Sync Objects as deprecated.

2. You cannot remotely call functions and stored procedures that exist on the source production database.  It goes against the principal of guaranteeing Remote Read Only access.  You have 2 options, recreate those functions in your own Dev Database and apply them to the Tables and Views sucked, or create a Dev View in the Production Database that uses those functions or stored procedures.

