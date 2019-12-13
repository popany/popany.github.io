---
layout: post
title: Install Oracle Instant Client
---

## [Instant Client Installation for Linux x86-64 (64-bit)](https://www.oracle.com/database/technologies/instant-client/linux-x86-64-downloads.html)

ODBC users should follow the [ODBC Installation Instructions](https://www.oracle.com/database/technologies/releasenote-odbc-ic.html).

### Install from zip files

1. Download the desired Instant Client ZIP files. All installations require a **Basic** or **Basic Light** package.

2. Unzip the packages into a single directory such as `/opt/oracle/instantclient_19_3` that is accessible to your application. For example:

        # cd /opt/oracle      
        # unzip instantclient-basic-linux.x64-19.3.0.0.0dbru.zip

    The various packages install into subdirectories of `/usr/lib/oracle`, `/usr/include/oracle`, and `/usr/share/oracle`.

3. Prior to version **18.3**, create the appropriate links for the version of Instant Client. For example:

        # cd /opt/oracle/instantclient_12_2
        # ln -s libclntsh.so.12.1 libclntsh.so
        # ln -s libocci.so.12.1 libocci.so

4. Install the operating system `libaio` package. This is called `libaio1` on some Linux distributions.
For example, on Oracle Linux, run:

        sudo yum install libaio

5. Update the runtime link path, for example:

        # sh -c "echo /opt/oracle/instantclient_19_3 > /etc/ld.so.conf.d/oracle-instantclient.conf"
        # ldconfig

6. If you intend to co-locate optional Oracle configuration files such as `tnsnames.ora`, `sqlnet.ora`, `ldap.ora`, or `oraaccess.xml` with Instant Client, put them in the `network/admin` subdirectory. This needs to be created for `12.2` and earlier, for example:

        mkdir -p /opt/oracle/instantclient_12_2/network/admin

    This is the default Oracle configuration directory for applications linked with this Instant Client.

    Alternatively, Oracle configuration files can be put in another, accessible directory. Then set the environment variable `TNS_ADMIN` to that directory name.

7. To use binaries such as sqlplus from the SQL*Plus package, unzip the package to the same directory as the Basic package and then update your `PATH` environment variable, for example:

        export PATH=/opt/oracle/instantclient_19_3:$PATH

8. Start your application.

## [Installing Oracle Instant Client ODBC](https://www.oracle.com/database/technologies/releasenote-odbc-ic.html)

### On Linux and UNIX

1. Download Install the Instant Client **Basic** or **Basic Light** package as described above.

2. Download the **Instant Client ODBC package**. Unzip it in the same directory as your Basic or Basic Light package.

3. Install the **unixODBC driver manager** if it is not already available.

4. Execute `odbc_update_ini.sh` from the Instant Client directory.

5. Set any Oracle Globalization variables required for your locale. See the Oracle Database Globalization Support Guide for more information. For example on Linux you could set export `NLS_LANG=JAPANESE_JAPAN.JA16EUC` to work in the `JA16EUC` character in Japanese.

### Uninstalling Oracle ODBC Instant Client On Linux and UNIX

The procedure to uninstall Instant Client ODBC on Linux/UNIX is:

1. Remove the Oracle ODBC driver entry from the `odbcinst.ini` file of the `unixODBC driver manager`. The default name of this entry is like `[Oracle 19c ODBC driver]`.

2. Remove the `DSN` entry of the Oracle ODBC driver from `odbc.ini`. The default name of the DSN entry is like `[OracleODBC-19c]`.

3. Delete all files and directories in the Instant Client ODBC directory.
