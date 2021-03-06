# SQL Data Sucker

## Overview

If we have a large number of production databases, or even just one, we may choose to Extract the data, Load the Data into a Reporting Database, and then Transform the Data (ELT).  In the cloud we have an array of tools to choose from.  On Premise environments might not have this option.  Introducing SQLDataSucker.  Selectively suck data from one or more SQL Servers into a central SQL database for reporting.  Build up a Data Catalog for your Report Developers.  Configure database objects to Sync and customize the scheduling.  Choose a custom update method.  Automatically assign permissions based on rules and more!

## Why use it?

Let's start from the beginning.  Allowing Report Developers to connect directly to Production Databases may be convenient but eventually you may find yourself in what some call Database Hell.  This is especially true if you manage multiple servers + potentially 100s or 1000s of databases.  The impacts of allowing developers to connect directly to Production Databases will eventually show up as reports of degraded application performance, "degreaded" (..degraded reads) report performance, data sources in more holes than a chipmonk can dig, and the one thing no DBA wants to hear about, loss of trust when it comes to data security and no fingers to point.  The solution is simple.  Copy your Production Data into a Reporting Database, secure it, and set the baseline for a lovely day as a DBA.

So now that we have a basic premise for duplicating, aka syncing data, we can ask:

1. How much data?
2. How often?
3. What technology/method to use?
4. How do we secure it?
5. How do we procure access control?

How much data?  Well, if you have done much work with Report Development, you will probably agree that 10% of your tables contain 90% of the data that you actually care about as a business.  So let's just assume that it would be overkill to copy the entire database.  This puts Full Database Backup Restore out of the question.  Let's elaborate a bit.  Full backup restore is not an option with large databases - The backup restore time can easily exceed the data refresh interval requirement.  Even so, if the backup was done on a higher version of SQL Server than your Report Database SQL Server, you won't be able to restore it to a lower version.  Further more you don't want to end up with 100 separate reporting databases, especially for the sake of 100 tables.  Lastly, you're killing your hardware by unecessary I/O.  So it's definitely not an option.

Configure Replication?  If you have 100+ servers there is a high chance you might not even be allowed to make changes on some of them.  Replication requires touching stuff that could break stuff.  You also don't have time to be making changes on 100s servers, much less  keeping track of things on those servers that others may have changed in your absence.  Replication features are different for different versions of SQL Server and "different" is not "consistent" enough IMO.  Central Configuration should be maximized and Remote Configuration should be minimal.  Either it works or it doesn't, and if it doesn't it shouldn't take 10 minutes to understand why and fix it.  This means we have to use a method that is unquestionably supported on all versions of SQL Server, and with no remote chance of the feature being deprecated.  Hint hint: SELECT.

So with that said...why not just build a data sync solution.

## Terminology

- Report Viewer - The people who view reports.
- Report Developer - The people who develop reports.
- Data Scientist - The Report Developers who enjoy math in particular.
- Data Engineer - The guy who made this.
- Data Catalog - All database names, object names, and meta data.  No sample data.  Used by Report Developers to develop a Data Dictionary.
- Data Dictionary - Business definitions applied to the Data Catalog.  Identification of data.
- Sample Data - A subset of data from an object in Data Catalog to better identify what data exists.  Useful if you don't necessarily need to pull all the data in a large table.
- SQL Linked Servers - SQL Server technology/feature for connecting to remote SQL servers.  Critical dependency for this to work.

## Underlying Assumptions

1. At any time, a production database, its schema, or data can be deleted or changed.  This can screw up reports.  SQLDataSucker doesn't attempt to address that.  It is simply a pipeline for centralizing data.  Remapping Source Data to a Business Layer is a Data Science problem.  If source data is renamed or deleted, we simply leave the last copy intact on the Reporting Database and mark the Objects as deprecated so the Report Developer is aware.

2. You cannot, and should not, remotely call functions and stored procedures that exist on the production database.  It goes against the principal of Remote Read Only access, and will create a potentially massive security risk.  You have 2 options, copy those functions into your own Dev Database and create custom Tables and Views, or create a Dev View in the Production Database that uses those functions - assuming you have access to do so.



