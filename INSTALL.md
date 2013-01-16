Getting Started with DCM4CHEE Archive 4.0.0.Alpha1
==================================================

Requirements
------------
-   Java SE 6 or later - tested with [OpenJDK](http://openjdk.java.net/)
    and [Oracle JDK](http://java.com/en/download)

-   [JBoss Application Server 7.1.1.Final](http://www.jboss.org/jbossas/downloads)

-   Supported SQL Database:
    - [MySQL 5.5](http://dev.mysql.com/downloads/mysql)
    - [PostgreSQL 9.2.1](http://www.postgresql.org/download/)
    - [Firebird 2.5.1](http://www.firebirdsql.org/en/firebird-2-5-1/)
    - [DB2 10.1](http://www-01.ibm.com/software/data/db2/express/download.html)
    - [Oracle 11g](http://www.oracle.com/technetwork/products/express-edition/downloads/)
    - [Microsoft SQL Server](http://www.microsoft.com/en-us/download/details.aspx?id=29062)
      (not yet tested!)

-   LDAP Server - tested with
    - [OpenDJ 2.4.6](http://www.forgerock.org/opendj.html),
    - [OpenLDAP 2.4.33](http://www.openldap.org/software/download/) and
    - [Apache DS 2.0.0-M8](http://directory.apache.org/apacheds/2.0/downloads.html).

    *Note*: DCM4CHEE Archive 4.x also supports using Java Preferences as configuration
    backend. But because DCM4CHEE Archive 4.0.0.Alpha1 does not yet contain a
    configuration front-end, you would have to edit configuration entries in the
    Java Preferences back-end manually, which is quit cumbersome and therefore not
    further described here.

-   LDAP Browser - [Apache Directory Studio 1.5.3](http://directory.apache.org/studio/)

    *Note*: Because DCM4CHEE Archive 4.0.0.Alpha1 does not yet contain a specific
    configuration front-end, the LDAP Browser is needed to modify the archive
    configuration.


Download and extract binary distribution package
------------------------------------------------
DCM4CHEE Archive 4.x binary distributions for different databases can be obtained
from [Sourceforge](https://sourceforge.net/projects/dcm4che/files/dcm4chee-arc4/).
Extract (unzip) your chosen download to the directory of your choice.


Initialize Database
-------------------
*Note*: DCM4CHEE Archive 4.0.0.Alpha1 does not provide SQL scripts and utilities to
migrate DCM4CHEE Archive 2.x data base schema to DCM4CHEE Archive 4.x. There will be
provided by DCM4CHEE Archive 4.0.0 final release.

### MySQL

1. Enable remote access by commenting out `skip-networking` in configuration file `my.conf`.

2. Create database and grant access to user

        > mysql -u root -p<root-password>
        mysql> CREATE DATABASE <database-name>;
        mysql> GRANT ALL ON <database-name>.* TO '<user-name>' IDENTIFIED BY '<user-password>';
        mysql> quit

3. Create tables and indexes
       
        > mysql -u <user-name> -p<user-password> < $DCM4CHEE_ARC/sql/create-table-mysql.ddl
        > mysql -u <user-name> -p<user-password> < $DCM4CHEE_ARC/sql/create-index.ddl


### PostgreSQL

1. Create user with permission to create databases 

        > createuser -U postgres -P -d <user-name>
        Enter password for new role: <user-password> 
        Enter it again: <user-password> 

2. Create database

        > createdb -U <user-name> <database-name>

3. Create tables and indexes
       
        > psql -U <user-name> < $DCM4CHEE_ARC/sql/create-table-psql.ddl
        > psql -U <user-name> < $DCM4CHEE_ARC/sql/create-index.ddl


### Firebird

1. Define database name in configuration file `aliases.conf`:

        <database-name> = <database-file-path>

2. Create user

        > gsec -user sysdba -password masterkey \
          -add <user-name> -pw <user-password>

3. Create database, tables and indexes

        > isql 
        Use CONNECT or CREATE DATABASE to specify a database
        SQL> CREATE DATABASE 'localhost:<database-name>'
        CON> user '<user-name>' password '<user-password>';
        SQL> IN $DCM4CHEE_ARC/sql/create-table-firebird.ddl;
        SQL> IN $DCM4CHEE_ARC/sql/create-index.ddl;
        SQL> EXIT;

        
### DB2

1. Create database and grant authority to create tables to user
   (must match existing OS user)

        > sudo su db2inst1
        > db2
        db2 => CREATE DATABASE <database-name> PAGESIZE 16 K
        db2 => connect to <database-name>
        db2 => GRANT CREATETAB ON DATABASE TO USER <user-name>
        db2 => terminate
 
2. Create tables and indexes

        > su <user-name>
        Password: <user-password>
        > db2 connect to <database-name>
        > db2 -t < $DCM4CHEE_ARC/sql/create-table-db2.ddl
        > db2 -t < $DCM4CHEE_ARC/sql/create-index.ddl
        > db2 terminate
        

### Oracle 11g 

TODO


Setup LDAP Server
-----------------

### OpenDJ

1.  Copy LDAP schema files for OpenDJ from DCM4CHEE Archive distribution to
    OpenDJ schema configuration directory:

        > cp $DCM4CHEE_ARC/ldap/opendj/* $OPENDJ_HOME/config/schema/ [UNIX]
        > copy %DCM4CHEE_ARC%\ldap\opendj\* %OPENDJ_HOME%\config\schema\ [Windows]

2.  Run OpenDJ GUI based setup utility

        > $OPENDJ_HOME/setup
    
    Log the values choosen for
    -  LDAP Listener port (1389)
    -  Root User DN (cn=Directory Manager)
    -  Root User Password (secret)
    -  Directory Base DN (dc=example,dc=com)

    needed for the LDAP connection configuration of DCM4CHEE Archive.

4. After initial setup, you may start and stop OpenDJ by

        > $OPENDJ_HOME/bin/start-ds
        > $OPENDJ_HOME/bin/stopt-ds


### OpenLDAP

OpenLDAP binary distributions are available for most Linux distributions and
for [Windows](http://www.userbooster.de/en/download/openldap-for-windows.aspx).

OpenLDAP can be alternatively configured by

- [slapd.conf configuration file](http://www.openldap.org/doc/admin24/slapdconfig.html)
- [dynamic runtime configuration](http://www.openldap.org/doc/admin24/slapdconf2.html)

See also [Converting old style slapd.conf file to cn=config format][1]

[1]: http://www.openldap.org/doc/admin24/slapdconf2.html#Converting%20old%20style%20{{slapd.conf}}%285%29%20file%20to%20{{cn=config}}%20format

#### OpenLDAP with slapd.conf configuration file

1.  Copy LDAP schema files for OpenLDAP from DCM4CHEE Archive distribution to
    OpenLDAP schema configuration directory:

        > cp $DCM4CHEE_ARC/ldap/schema/* /etc/openldap/schema/ [UNIX]
        > copy %DCM4CHEE_ARC%\ldap\schema\* \Program Files\OpenLDAP\schema\ [Windows]

2.  Add references to schema files in `slapd.conf`, e.g.:

        include         /etc/openldap/schema/core.schema
        include         /etc/openldap/schema/dicom.schema
        include         /etc/openldap/schema/dcm4che.schema
        include         /etc/openldap/schema/dcm4che-hl7.schema
        include         /etc/openldap/schema/dcm4chee-archive.schema

3.  You may also change the default values for 

        suffix          "dc=my-domain,dc=com"
        rootdn          "cn=Manager,dc=my-domain,dc=com"
        rootpw          secret
   
    in `slapd.conf`.


#### OpenLDAP with dynamic runtime configuration

1.  Import LDAP schema files for OpenLDAP runtime configuration, binding as
    root user of the config backend, using OpenLDAP CL utility ldapadd, e.g.:

        > ldapadd -xW -Dcn=config -f $DCM4CHEE_ARC/ldap/slapd/dicom.ldif
        > ldapadd -xW -Dcn=config -f $DCM4CHEE_ARC/ldap/slapd/dcm4che.ldif
        > ldapadd -xW -Dcn=config -f $DCM4CHEE_ARC/ldap/slapd/dcm4che-hl7.ldif
        > ldapadd -xW -Dcn=config -f $DCM4CHEE_ARC/ldap/slapd/dcm4chee-archive.ldif

    If you don't know the root user and its password of the config backend, you may
    look into `/etc/openldap/slap.d/cn=config/olcDatabase={0}config.ldif`:

        olcRootDN: cn=config
        olcRootPW:: VmVyeVNlY3JldA==

    and decode the base64 decoded password, e.g:

        > echo -n VmVyeVNlY3JldA== | base64 -d
        VerySecret

    or specify a new password in plan text, e.g:

        olcRootPW: VerySecret

2.  Directory Base DN and Root User DN can be modified by changing the values of
    attributes

        olcSuffix: dc=my-domain,dc=com
        olcRootDN: cn=Manager,dc=my-domain,dc=com

    of object `olcDatabase={1}bdb,cn=config` by specifing the new values in a 
    LDIF file (e.g. `modify-baseDN.ldif`)

        dn: olcDatabase={1}bdb,cn=config
        changetype: modify
        replace: olcSuffix
        olcSuffix: dc=example,dc=com
        -
        replace: olcRootDN
        olcRootDN: cn=Manager,dc=example,dc=com
        -

    and applying it using OpenLDAP CL utility ldapmodify, e.g.:

        > ldapmodify -xW -Dcn=config -f modify-baseDN.ldif


### Apache DS 2.0

1.  Install [Apache DS 2.0.0-M8](http://directory.apache.org/apacheds/2.0/downloads.html)
    on your system and start Apache DS.

2.  Install [Apache Directory Studio 1.5.3](http://directory.apache.org/studio/) and
    create a new LDAP Connection with:

        Network Parameter:
            Hostname: localhost
            Port:     10389
        Authentication Parameter:
            Bind DN or user: uid=admin,ou=system
            Bind password:   secret

3.  Import LDAP schema files for Apache DS:

        $DCM4CHEE_ARC/ldap/apacheds/dicom.ldif
        $DCM4CHEE_ARC/ldap/apacheds/dcm4che.ldif
        $DCM4CHEE_ARC/ldap/apacheds/dcm4che-hl7.ldif
        $DCM4CHEE_ARC/ldap/apacheds/dcm4chee-archive.ldif

    using the LDIF import function of Apache Directory Studio LDAP Browser.

4.  You may modify the default Directory Base DN `dc=example,dc=com` by changing
    the value of attribute 

        ads-partitionsuffix: dc=example,dc=com`

    of object

        ou=config
        + ads-directoryServiceId=default
          + ou=partitions
              ads-partitionId=example
    
    using Apache Directory Studio LDAP Browser.


Import sample configuration into LDAP Server
--------------------------------------------  

1.  If not alread done, install
    [Apache Directory Studio 1.5.3](http://directory.apache.org/studio/) and create
    a new LDAP Connection corresponding to your LDAP Server configuration, e.g:

        Network Parameter:
            Hostname: localhost
            Port:     10389
        Authentication Parameter:
            Bind DN or user: cn=uid=admin,ou=system
            Bind password:   secret
        Browser Options:
            Base DN: dc=example,dc=com

2.  If you configured a different Directory Base DN than`dc=example,dc=com`,
    you have to replace all occurrences of `dc=example,dc=com` in LDIF files
 
        $DCM4CHEE_ARC/ldap/init-baseDN.ldif
        $DCM4CHEE_ARC/ldap/init-config.ldif
        $DCM4CHEE_ARC/ldap/sample-config.ldif

    by your Directory Base DN, e.g.:

        > cd $DCM4CHEE_ARC/ldap
        > sed -i s/dc=example,dc=com/dc=my-domain,dc=com/ init-baseDN.ldif
        > sed -i s/dc=example,dc=com/dc=my-domain,dc=com/ init-config.ldif
        > sed -i s/dc=example,dc=com/dc=my-domain,dc=com/ sample-config.ldif

3.  If there is not already a base entry in the directory data base, import
    `$DCM4CHEE_ARC/ldap/init-baseDN.ldif` using the LDIF import function of
    Apache Directory Studio LDAP Browser.

4.  If there are not already DICOM configuration root entries in the directory
    data base, import `$DCM4CHEE_ARC/ldap/init-config.ldif` using the LDIF import
    function of Apache Directory Studio LDAP Browser.  

5.  Import `$DCM4CHEE_ARC/ldap/sample-config.ldif` using the LDIF import function
    of Apache Directory Studio LDAP Browser.  

6.  By default configuration, DCM4CHEE Archive does not accept remote connections.
    To enable remote connections, replace the value of attribute

        dicomHostname=localhost
    
    of the 4 `dicomNetworkConnection` objects

        dc=example,dc=com
        + cn=DICOM Configuration
          + cn=Devices
            + dicomDeviceName=dcm4chee-arc
                cn=dicom
                cn=dicom-tls
                cn=hl7,dicomDeviceName
                cn=hl7-tls,dicomDeviceName

    by the actual hostname of your system, using Apache Directory Studio LDAP Browser. 

7.  To change the default AE Title: `DCM4CHEE`, modify the _Relative Distinguished Name_
    (RDN) of the `dicomNetworkAE` object
        
        dc=example,dc=com
        + cn=DICOM Configuration
          + cn=Devices
            + dicomDeviceName=dcm4chee-arc
                dicomAETitle=DCM4CHEE

     to

        dicomAETitle=<aet>

     You may also modify the AE Title: `DCM4CHEE_ADMIN` of the additional AE:

        dc=example,dc=com
        + cn=DICOM Configuration
          + cn=Devices
            + dicomDeviceName=dcm4chee-arc
                dicomAETitle=DCM4CHEE_ADMIN

    which is configured to provide also Access to Rejected Instances for
    Quality Reasons, as specified by IHE Radiology Technical Framework 
    Supplement [Imaging Object Change Management (IOCM)][2].

[2]: http://www.ihe.net/Technical_Framework/upload/IHE_RAD_Suppl_IOCM_Rev1-1_TI_2011-05-17.pdf

8.  By default configuration, DCM4CHEE Archive stores received objects below
    `$JBOSS_HOME/standalone/data`. To specify a different storage location,
    replace the value of attribute
    
        dcmInitFileSystemURI=file:${jboss.server.data.dir}

    of the `dicomNetworkAE` object

        dc=example,dc=com
        + cn=DICOM Configuration
          + cn=Devices
            + dicomDeviceName=dcm4chee-arc
                dicomAETitle=DCM4CHEE

    by `file:<absolute-directory-path>`, using Apache Directory Studio LDAP Browser.

    **Attention**: The value of attribute `dcmInitFileSystemURI` is used to initialize
    the (first) record in the `filesystem` table of the archive data base on receive of
    the first object. It is not effective, if the `filesystem` table already contains
    such record. 


Setup JBoss AS 7
----------------

1.  Adjust DCM4CHEE Archive LDAP Connection configuration file
    `$DCM4CHEE_ARC/configuration/dcm4chee-arc/ldap.properties`:

        java.naming.factory.initial=com.sun.jndi.ldap.LdapCtxFactory
        java.naming.ldap.attributes.binary=dicomVendorData
        java.naming.provider.url=ldap://localhost:1389/dc=example,dc=com
        java.naming.security.principal=cn=Directory Manager
        java.naming.security.credentials=secret

    to your LDAP Server configuration.

2.  Copy configuration files into JBoss AS 7 installation:

        > cp -r $DCM4CHEE_ARC/configuration/dcm4chee-arc $JBOSS_HOME/standalone/configuration [UNIX]
        > xcopy %DCM4CHEE_ARC%\configuration\dcm4chee-arc %JBOSS_HOME%\standalone\configuration [Windows]

    *Note*: Beside LDAP Connection configuration, the private key used in TLS connections
    and XSLT stylesheets specifing attribute coercion in incoming or outgoing DICOM messages
    and mapping of HL7 fields in incoming HL7 messages are not stored in LDAP.

3.  Install DCM4CHE 3.0.1 libraries as JBoss AS 7 module:

        > cd  $JBOSS_HOME
        > unzip $DCM4CHEE_ARC/jboss-module/dcm4che-jboss-modules-3.0.0.zip

4.  Install QueryDSL 2.8.1 libraries as JBoss AS 7 module:

        > cd  $JBOSS_HOME
        > unzip $DCM4CHEE_ARC/jboss-module/querydsl-jboss-modules-2.8.1.zip

5.  Install JDBC Driver. DCM4CHEE Archive 4.x binary distributions do not include
    a JDBC driver for the database for license issues. You may download it from:
    -   [MySQL](http://www.mysql.com/products/connector/)
    -   [PostgreSQL]( http://jdbc.postgresql.org/)
    -   [Firebird](http://www.firebirdsql.org/en/jdbc-driver/)
    -   [DB2](http://www-306.ibm.com/software/data/db2/java/), also included in DB2 Express-C
    -   [Oracle](http://www.oracle.com/technology/software/tech/java/sqlj_jdbc/index.htm),
        also included in Oracle 11g XE)
    -   [Microsoft SQL Server](http://msdn.microsoft.com/data/jdbc/)

    The JDBC driver can be installed either as a deployment or as a core module.
    [See](https://docs.jboss.org/author/display/AS71/Developer+Guide#DeveloperGuide-InstalltheJDBCdriver)
    
    Installation as deployment is limited to JDBC 4-compliant driver consisting of **one** JAR.

    For installation as a core module, `$DCM4CHEE_ARC/jboss-module/jdbc-jboss-modules-1.0.0-<database>.zip`
    already provides a module definition file `module.xml`. You just need to extract the ZIP file into
    $JBOSS_HOME and copy the JDBC Driver file(s) into the sub-directory, e.g.:

        > cd $JBOSS_HOME
        > unzip $DCM4CHEE_ARC/jboss-module/jdbc-jboss-modules-1.0.0-db2.zip
        > cd $DB2_HOME/java
        > cp db2jcc4.jar db2jcc_license_cu.jar $JBOSS_HOME/modules/com/ibm/db2/main/

    Verify, that the actual JDBC Driver file(s) name matches the path(s) in the provided
    `module.xml`, e.g.:

         <?xml version="1.0" encoding="UTF-8"?>
         <module xmlns="urn:jboss:module:1.1" name="com.ibm.db2">
             <resources>
                 <resource-root path="db2jcc4.jar"/>
                 <resource-root path="db2jcc_license_cu.jar"/>
             </resources>
         
             <dependencies>
                 <module name="javax.api"/>
                 <module name="javax.transaction.api"/>
             </dependencies>
         </module>


6.  Start JBoss AS 7 in standalone mode with the Java EE 6 Full Profile configuration.
    To preserve the original JBoss AS 7 configuration you may copy the original
    configuration file for JavaEE 6 Full Profile:

        > cd $JBOSS_HOME/standalone/configuration/
        > cp standalone-full.xml dcm4chee-arc.xml

    and start JBoss AS 7 specifying the new configuration file:
        
        > $JBOSS_HOME/bin/standalone.sh -c dcm4chee-arc.xml [UNIX]
        > %JBOSS_HOME%\bin\standalone.bat -c dcm4chee-arc.xml [Windows]
   
    Verify, that JBoss AS 7 started successfully, e.g.:

        =========================================================================

          JBoss Bootstrap Environment

          JBOSS_HOME: /home/gunter/jboss7

          JAVA: /usr/lib/jvm/java-6-openjdk/bin/java

          JAVA_OPTS:  -server -XX:+UseCompressedOops -XX:+TieredCompilation ...

        =========================================================================

        13:01:48,788 INFO  [org.jboss.modules] JBoss Modules version 1.1.1.GA
        13:01:48,926 INFO  [org.jboss.msc] JBoss MSC version 1.0.2.GA
        13:01:48,969 INFO  [org.jboss.as] JBAS015899: JBoss AS 7.1.1.Final "Brontes" starting
        :
        13:01:51,239 INFO  [org.jboss.as] (Controller Boot Thread) JBAS015874: JBoss AS 7.1.1.Final "Brontes" started ...
                
    Running JBoss AS 7 in domain mode should work, but was not yet tested.

7.  Add JDBC Driver into the server configuration using JBoss AS 7 CLI in a new console window:

        > $JBOSS_HOME/bin/jboss-cli.sh -c [UNIX]
        > %JBOSS_HOME%\bin\jboss-cli.bat -c [Windows]
        [standalone@localhost:9999 /] /subsystem=datasources/jdbc-driver=<driver-name>:add(driver-module-name=<module-name>)

    You may choose any `<driver-name>` for the JDBC Driver, `<module-name>` must match the name
    defined in the module definition file `module.xml` of the JDBC driver, e.g.:

        [standalone@localhost:9999 /] /subsystem=datasources/jdbc-driver=db2:add(driver-module-name=com.ibm.db2)

8.  Create and enable a new Data Source bound to JNDI name `java:/PacsDS` using JBoss AS 7 CLI:

        [standalone@localhost:9999 /] data-source add --name=PacsDS \
        >     --driver-name=<driver-name> \
        >     --connection-url=<jdbc-url> \
        >     --jndi-name=java:/PacsDS \
        >     --user-name=<user-name> \
        >     --password=<user-password>
        [standalone@localhost:9999 /] data-source enable --name=PacsDS

    The format of `<jdbc-url>` is JDBC Driver specific, e.g.:
    -  MySQL: `jdbc:mysql://localhost:3306/<database-name>`
    -  PostgreSQL: `jdbc:postgresql://localhost:5432/<database-name>`
    -  Firebird: `jdbc:firebirdsql:localhost/3050:<database-name>`
    -  DB2: `jdbc:db2://localhost:50000/<database-name>`
    -  Oracle: `jdbc:oracle:thin:@localhost:1521:<database-name>`
    -  Microsoft SQL Server: `jdbc:sqlserver://localhost:1433;databaseName=<database-name>`

9.  Create JMS Queues using JBoss AS 7 CLI:

        [standalone@localhost:9999 /] jms-queue add --queue-address=ianscu --entries=queue/ianscu
        [standalone@localhost:9999 /] jms-queue add --queue-address=mppsscu --entries=queue/mppsscu
        [standalone@localhost:9999 /] jms-queue add --queue-address=stgcmtscp --entries=queue/stgcmtscp

10. At default, DCM4CHEE Archive 4.x will look for the LDAP connection configuration file at

        $JBOSS_HOME/standalone/configuration/dcm4chee-arc/ldap.properties

    You may specify a different location by system property `org.dcm4chee.archive.ldapPropertiesURL`
    using JBoss AS 7 CLI:

        [standalone@localhost:9999 /] /system-property=org.dcm4chee.archive.ldapPropertiesURL:add(value=<url>)

    If DCM4CHEE Archive 4.x cannot find the LDAP connection configuration on the specified location, it
    will try to fetch the Archive configuration from Java Preferences.

11. At default, DCM4CHEE Archive 4.x will assume `dcm4chee-arc` as its Device Name, used to find its
    configuration in the configuration backend (LDAP Server or Java Preferences). You may specify a different
    Device Name by system property `org.dcm4chee.archive.deviceName` using JBoss AS 7 CLI:

        [standalone@localhost:9999 /] /system-property=org.dcm4chee.archive.deviceName:add(value=<device-name>)

12. At default, DCM4CHEE Archive 4.x will register a JMX MBean under `org.dcm4chee.archive:type=Service` to enable
    to start and stop the archive and to reload its configuration from the configuration backend using the
    Java Monitoring and Management Console `jconsole`. You may specify a different JMX name by system property
    `org.dcm4chee.archive.jmxName` using JBoss AS 7 CLI:

        [standalone@localhost:9999 /] /system-property=org.dcm4chee.archive.jmxName:add(value=<jmx-name>)
   
13. Deploy DCM4CHEE Archive 4.x using JBoss AS 7 CLI, e.g.:

        [standalone@localhost:9999 /] deploy $DCM4CHEE_ARC/deploy/dcm4chee-arc-4.1.0.Alpha1-mysql.war

    Verify that DCM4CHEE Archive was deployed and started successfully, e.g.:

        13:11:01,711 INFO  [org.jboss.as.server.deployment] (MSC service thread 1-1) JBAS015876: Starting deployment of "dcm4chee-arc-4.1.0.Alpha1-mysql.war"
        13:11:01,763 INFO  [org.jboss.as.jpa] (MSC service thread 1-8) JBAS011401: Read persistence.xml for dcm4chee-arc
        :
        13:11:02,706 INFO  [org.dcm4che.net.Connection] (pool-18-thread-1) Start listening on localhost/127.0.0.1:11112
        13:11:02,713 INFO  [org.dcm4che.net.Connection] (pool-18-thread-2) Start listening on localhost/127.0.0.1:2762
        13:11:02,713 INFO  [org.dcm4che.net.Connection] (pool-18-thread-3) Start listening on localhost/127.0.0.1:2575
        13:11:02,714 INFO  [org.dcm4che.net.Connection] (pool-18-thread-4) Start listening on localhost/127.0.0.1:12575
        13:11:02,726 INFO  [org.jboss.web] (MSC service thread 1-1) JBAS018210: Registering web context: /service
        13:11:02,771 INFO  [org.jboss.as.server] (management-handler-thread - 1) JBAS018559: Deployed "dcm4chee-arc-4.1.0.Alpha1-mysql.war"

14. You may undeploy DCM4CHEE Archive at any time using JBoss AS 7 CLI, e.g.:

        [standalone@localhost:9999 /] undeploy dcm4chee-arc-4.1.0.Alpha1-mysql.war

        14:06:09,874 INFO  [org.dcm4che.net.Connection] (pool-18-thread-2) Stop listening on localhost/127.0.0.1:2762
        14:06:09,874 INFO  [org.dcm4che.net.Connection] (pool-18-thread-4) Stop listening on localhost/127.0.0.1:12575
        14:06:09,874 INFO  [org.dcm4che.net.Connection] (pool-18-thread-1) Stop listening on localhost/127.0.0.1:11112
        14:06:09,874 INFO  [org.dcm4che.net.Connection] (pool-18-thread-3) Stop listening on localhost/127.0.0.1:2575
        14:06:09,955 INFO  [org.jboss.as.jpa] (MSC service thread 1-7) JBAS011403: Stopping Persistence Unit Service 'dcm4chee-arc-4.1.0.Alpha1-mysql.war#dcm4chee-arc'
        14:06:10,000 INFO  [org.jboss.as.server.deployment] (MSC service thread 1-5) JBAS015877: Stopped deployment dcm4chee-arc-4.1.0.Alpha1-mysql.war in 132ms
        14:06:10,561 INFO  [org.jboss.as.repository] (management-handler-thread - 7) JBAS014901: Content removed from location ... 
        14:06:10,563 INFO  [org.jboss.as.server] (management-handler-thread - 7) JBAS018558: Undeployed "dcm4chee-arc-4.1.0.Alpha1-mysql.war"


Java Monitoring and Management Console `jconsole`
-------------------------------------------------

1.  Invoke `jconsole` using the launcher script provided by JBoss AS 7:

        > $JBOSS_HOME/bin/jconsole.sh -c [UNIX]
        > %JBOSS_HOME%\bin\jconsole.bat -c [Windows]
          
2.  Select Local Process: `jboss-modules.jar ...` in the New Connection dialog.

3.  Select `org.dcm4chee.archive/Service/Operations` in the MBeans tab.

4.  Invoke the `Start`, `Stop` or `ReloadConfiguration` operation to start or stop
    DCM4CHEE Archive 4.x, or to reload its configuration from the configuration
    backend (LDAP Server or Java Preferences).


Source Build With Eclipse
--------------------------

0. Use Eclipse Juno, and install Jboss Tools 4.0


1. Clone dcm4che, dcm4chee-arc, querydsl-jboss-modules and jdbc-jboss-modules from github

2. run mvn install inside dcm4che, querydsl-jboss-modules and jdbc-jboss-modules projects.

3. In install instructions, use dcm4che project for dcm4che-jboss-modules zip file, and querydsl-jboss-modules project for querydsl jboss modules zip file.

4. import dcm4che and dcm4chee-arc into eclipse as maven projects

5. under dcm4chee-arc project settings -> validation, disable JPA validation 

6. under dcm4chee-arc-entity, add target/generated-sources/annotations as build path source folder

7. set up a run configuration for dcm4chee-arc-parent, with install goal, and
db = psql, for example.

8. verify that the run configuration will build the WAR

9. Open Window -> Show View -> Servers, and create a new jboss 7.1 server

10. add dcm4chee-arc-service when creating the server

11. now, you should be able to debug the archive from inside eclipse




Testing DCM4CHEE Archive 4.x
----------------------------

For testing DCM4CHEE Archive 4.x, you may use DICOM and HL7 utilities provided by
[DCM4CHE 3.x](https://sourceforge.net/projects/dcm4che/files/dcm4che3/). After
extraction, you will find launcher scripts for Linux and Windows in directory 
`dcm4che-<version>/bin/`.


### Test Storage of DICOM Composite Objects

Use DCM4CHE 3.x's `storescu` utility to send DICOM Composite Objects to DCM4CHEE Archive 4.x, e.g.:

    > $DCM4CHE_HOME/bin/storescu -cDCM4CHEE@localhost:11112 ~/MESA-storage-A_12_5_0/modality/CT/CT1

Verify the success status=0H in logged `C-STORE-RSP` messages, e.g.:

    15:22:00,133 INFO  - DCM4CHEE-1 >> 1:C-STORE-RSP[pcid=3, status=0H
      cuid=1.2.840.10008.5.1.4.1.1.2 - CT Image Storage
      iuid=1.2.840.113674.950809132354242.100 - ?
      tsuid=1.2.840.10008.1.2 - Implicit VR Little Endian

You may try to send another set of objects via dicom-tls:

    > $DCM4CHE_HOME/bin/storescu -cDCM4CHEE@localhost:2762 --tls ~/MESA-storage-A_12_5_0/modality/CT/CT2


### Test Query for DICOM Composite Objects

Use DCM4CHE 3.x's `dcmdump` utility to determine the `Patient ID` of one of the
previous stored DICOM Composite Objects:

    > $DCM4CHE_HOME/bin/dcmdump ~/MESA-storage-A_12_5_0/modality/CT/CT1/CT1S1/CT1S1IM1.dcm | grep PatientID
    360: (0010,0020) LO #6 [GE0514] PatientID

and invoke a query for studies of this patient using DCM4CHE 3.x's `findscu` utility:

    > $DCM4CHE_HOME/bin/findscu -cDCM4CHEE@localhost:11112 \
      -mPatientID=GE0514 -rStudyInstanceUID

Verify, that there is at least one C-FIND-RSP with pending status=ff00H, e.g.:

    16:02:46,488 INFO  - DCM4CHEE-1 >> 1:C-FIND-RSP[pcid=1, status=ff00H
      cuid=1.2.840.10008.5.1.4.1.2.2.1 - Study Root Query/Retrieve Information Model - FIND
      tsuid=1.2.840.10008.1.2 - Implicit VR Little Endian
    16:02:46,489 DEBUG - Command:
    (0000,0002) UI [1.2.840.10008.5.1.4.1.2.2.1] AffectedSOPClassUID
    (0000,0100) US [32800] CommandField
    (0000,0120) US [1] MessageIDBeingRespondedTo
    (0000,0800) US [0] CommandDataSetType 06801411525
    (0000,0900) US [65280] Status

    16:02:46,489 DEBUG - Dataset:
    (0008,0052) CS [STUDY] QueryRetrieveLevel
    (0008,0054) AE [DCM4CHEE] RetrieveAETitle
    (0008,0056) CS [ONLINE] InstanceAvailability
    (0010,0020) LO [GE0514] PatientID
    (0020,000D) UI [1.2.840.113674.514.212.200] StudyInstanceUID

and that there is a final C-FIND-RSP with success status=0H, e.g.:

    16:02:46,492 INFO  - DCM4CHEE-1 >> 1:C-FIND-RSP[pcid=1, status=0H
      cuid=1.2.840.10008.5.1.4.1.2.2.1 - Study Root Query/Retrieve Information Model - FIND
      tsuid=1.2.840.10008.1.2 - Implicit VR Little Endian

### Test Retrieve of DICOM Composite Objects

Start DCM4CHE 3.x's `storescp` utility in a new console window for
acting as retrieve destination, listening on port 11115:

    > $DCM4CHE_HOME/bin/storescp -b11115
    16:28:46,910 INFO  - Start listening on 0.0.0.0/0.0.0.0:11115

Invoke a retrieve request for a study with given Study Instance UID
to STORESCP using DCM4CHE 3.x's `movescu` utility:

    > $DCM4CHE_HOME/bin/movescu -cDCM4CHEE@localhost:11112 --dest STORESCP \
      -mStudyInstanceUID=1.2.840.113674.514.212.200

Verify the success status=0H in the final `C-MOVE-RSP` messages, e.g.:

    16:40:13,698 INFO  - DCM4CHEE-1 >> 1:C-MOVE-RSP[pcid=1, completed=11, failed=0, warning=0, status=0H
      cuid=1.2.840.10008.5.1.4.1.2.2.2 - Study Root Query/Retrieve Information Model - MOVE
      tsuid=1.2.840.10008.1.2 - Implicit VR Little Endian
    16:40:13,698 DEBUG - Command:
    (0000,0002) UI [1.2.840.10008.5.1.4.1.2.2.2] AffectedSOPClassUID
    (0000,0100) US [32801] CommandField
    (0000,0120) US [1] MessageIDBeingRespondedTo
    (0000,0800) US [257] CommandDataSetType
    (0000,0900) US [0] Status
    (0000,1021) US [11] NumberOfCompletedSuboperations
    (0000,1022) US [0] NumberOfFailedSuboperations
    (0000,1023) US [0] NumberOfWarningSuboperations

and that `storescp` actually received the objects of the study, e.g.:

    16:40:13,687 INFO  - DCM4CHEE+1 >> 11:C-STORE-RQ[pcid=1, prior=0
      orig=MOVESCU >> 1:C-MOVE-RQ
      cuid=1.2.840.10008.5.1.4.1.1.2 - CT Image Storage
      iuid=1.2.840.113674.950809132339103.100 - ?
      tsuid=1.2.840.10008.1.2 - Implicit VR Little Endian
    16:40:13,688 DEBUG - Command:
    (0000,0002) UI [1.2.840.10008.5.1.4.1.1.2] AffectedSOPClassUID
    (0000,0100) US [1] CommandField
    (0000,0110) US [11] MessageID
    (0000,0700) US [0] Priority
    (0000,0800) US [0] CommandDataSetType
    (0000,1000) UI [1.2.840.113674.950809132339103.100] AffectedSOPInstanceUID
    (0000,1030) AE [MOVESCU] MoveOriginatorApplicationEntityTitle
    (0000,1031) US [1] MoveOriginatorMessageID

You may also try to retrieve the same study by C-GET instead of C-MOVE, using
DCM4CHE 3.x's `getscu` utility:

    > $DCM4CHE_HOME/bin/getscu -cDCM4CHEE@localhost:11112 \
      -mStudyInstanceUID=1.2.840.113674.514.212.200

