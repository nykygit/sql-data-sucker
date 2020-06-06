# sql-data-sucker

## Overview
If we have a large number of production databases, or even just one, we may choose to Extract the data, Load the Data into a Reporting Database, and then Transform the Data (ELT).  Allowing Report Developers to connect directly to production databases, while at first may be convenient, can cause a myriad of issues - performance, security, stability, etc... The goal of this project is to configure a SQL Server with a centralized Reporting Database for Report Developers to consume data from a central endpoint, with centralized permissions.


## Background
There are a number of ELT methods and third party products on the market.  Some are more or less invasive than others.  The goal of this is to access production with Read Only access, with absolutely no change to the Production Schema or System.  Some might ask, why not configure replication?  Try doing that on 100 servers, or where it is not possible to make changes in the production environment.  We just need to peer in via a looking glass with Read Only Access and selectively take only what is needed for Reporting purposes. This is a minimalistic on demand approach.  With large databases we may not be able to backup restore the whole database on an hourly basis, nor do we necessarily need all the data in the database for reporting.  Also, some database objects we may only need to be refreshed hourly versus daily, versus weekly.  This will save you on bandwidth and unecessary disk writes and reads, potentially extending hardware longevity.  You may only want to pull the newest rows, greater than a certain date, or primary key greater than the last one pulled.  We can even go as far as preventing performance issues by preventing syncing of tables with more than X rows on a certain schedule, or based on the query TTL.  By introducing a permissions layer on a centralized Reporting Database - you have the opportunity to reset your approach to data access control.  Production database permissions was never designed to overlap with reporting permissions - and you could actually introduce major security holes unknowingly!  Lastly you are probably asking why build this?  Well, because it's easy, you have more control over the features, and I just so happen to enjoy it!

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

