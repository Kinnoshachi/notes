# DB Instance

### Q: What should I do if my queries seem to be running slowly?
    provides access to over 50 CPU, memory, file system, and disk I/O metrics. You can enable these features on a per-instance basis, granularity (all the way down to 1 second). High levels of CPU utilization can reduce query performance and in this case you may want to consider scaling your DB instance class. 

    MySQL or MariaDB, you can access the slow query logs. You could set the "slow_query_log" DB Parameter and query the mysql.slow_log table to review the slow-running SQL queries. 
    Oracle, you can use the Oracle trace file data to identify slow queries.
    SQL Server, you can use the client side SQL Server traces to identify slow queries


# Database Engine Versions

### Q: Does Amazon RDS provide guidelines for support of new DB engine versions?
    aim to support new engine versions within 5 months of their general availability.

### Q: How do I specify which supported DB engine version I would like my DB instance to run?
    via the Launch DB Instance operation in the AWS Management Console or the CreateDBInstance API.

### engine version of my DB instance is upgraded?
    manually upgrade a database instance to a supported engine version, use the Modify DB Instance command on the AWS Management Console or the ModifyDBInstance API and set the DB Engine Version parameter, applied immediately or during maintenence period.
    downtime is required to upgrade a DB engine version, even for Multi-AZ instances
    can turn off automatic minor version upgrades, you can do so by setting the Auto Minor Version Upgrade setting to “No”.
    
    Oracle and RDS for SQL Server, if the upgrade to the next minor version requires a change to a different edition, then we may not schedule automatic upgrades even if you have enabled the Auto Minor Version Upgrade setting.

    Backups will simply be taken from your standby to avoid I/O suspension on the DB instance primary.

# Billing
### Q: How will I be charged and billed for my use of Amazon RDS?
    * DB instance hours
    * Storage (per GB per month)
    * I/O requests per month
    * RDS Magnetic Storage and Amazon Aurora only
    * Provisioned IOPS per month, Provisioned IOPS (SSD) Storage only
    * Backup Storage
    * data transfer in and out
    * not charged for the data transfer incurred in replicating data between your primary and standby



#### Q: What is a DB Subnet Group and why do I need one?

