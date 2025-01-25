# Snowflake


## Part 1. Setup
```sql
create database analytics;
create warehouse transforming with warehouse_size = 'MEDIUM';

-- Role for dbt developers to access these objects.
-- Design is to grant appropriate permissions for a role and finally assign the role the users who need them.
create role transformer;

grant imported privileges on database snowflake_sample_data to role transformer;
grant usage on schema snowflake_sample_data.tpch_sf10 to role transformer; -- This throws an error.
grant select on all tables in schema snowflake_sample_data.tpch_sf10 to role transformer;

grant usage on database analytics to role transformer;
grant reference_usage on database analytics to role transformer; -- This throws an error.
grant modify on database analytics to role transformer;
grant monitor on database analytics to role transformer;
grant create schema on database analytics to role transformer;

grant operate on warehouse transforming to role transformer;
grant usage on warehouse transforming to role transformer;

-- Grant the roles to users.
grant role transformer to user bkowshik;


-- User to use with DBT.
create user dbt
email = '' -- todo
password = '' -- todo
default_role = transformer,
default_warehouse = transforming,
must_change_password = false;

grant role transformer to user dbt;
```


## Part 2. Test

```sql
-- Check permissions to query sample datasets.
use warehouse transforming;
select * from snowflake_sample_data.tpch_sf1.customer;

-- Check permissions to create schema and table inside the database.
use database analytics;
create or replace schema dbt_availlant;

-- Here we are copying the raw dataset into a dataset for modelling.
-- This is where all the transformations will come in.
create table analytics.dbt_availlant.customer as (
select * from snowflake_sample_data.tpch_sf1.customer);

-- Cleanup.
drop schema analytics.dbt_availlant cascade;
```
