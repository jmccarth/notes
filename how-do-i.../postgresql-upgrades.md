# PostgreSQL Upgrades

After a lot of work I managed to upgrade a PostgreSQL 9.6 server to 10.10 with a corresponding PostGIS 2.3.7 to 2.4.4 upgrade. Here's how I did it on a Windows server

## Preparation

1. Take a snapshot of the server before starting
2. Take a full cluster backup of the server berfore starting

## Install New Server

1. Install PostgreSQL 10.10 from installer
2. Install PostGIS 2.4.4 from installer \(stack builder does not work on the server I used\)
3. To avoid errors related to not finding rtpostgis-x-x-x.dll I needed to copy 2 dlls \(libeay32.dll and ssleay32.dll\) from %datadir%/bin/postgisgui to %datadir/bin. See [https://gis.stackexchange.com/questions/331653/error-could-not-load-library-c-program-files-postgresql-11-lib-rtpostgis-2-5](https://gis.stackexchange.com/questions/331653/error-could-not-load-library-c-program-files-postgresql-11-lib-rtpostgis-2-5)
4. Configure the new server to use port 5433

## Update Old Databases

1. Install PostGIS 2.4.4 onto old \(9.6\) database
2. Upgrade any databases with PostGIS extension to 2.4.4

   ```sql
   ALTER EXTENSION postgis UPDATE;
   SELECT postgis_full_version(); -- ensure version is now 2.4.4
   ```

##  Migrate Data from Old to New

I basically followed the docs on this, but the process was

```text
.\pg_dumpall.exe -c --if-exists -p 5432 -U postgres -f C:\temp\96dumpall.dump
.\psql.exe -v ON_ERROR_STOP=1 -d postgres -U postgres -p 5433 -f C:\temp\96dumpall.dump
```

Then I just needed to configure postgresql.conf and pg\_hba.conf to match the 9.6 configuration, change the port to 5432, turn off the old server from starting automatically, and start the new server.

