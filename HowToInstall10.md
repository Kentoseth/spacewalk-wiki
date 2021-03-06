# Spacewalk Installation Instructions



These are installation instruction for Spacewalk 1.0.

Spacewalk 0.8 installation instructions are available at [HowToInstall08](HowToInstall08)

----
## Prerequisites



 * Outbound open ports 80, 443, 4545 (only if you want to enable monitoring)
 * Inbound open ports 80, 443, 5222 (only if you want to push actions to client machines) and 5269 (only for push actions to a Spacewalk Proxy), 69 udp if you want to use tftp
 * Storage for database: 128KB per client system + 64MB per channel
 * Storage for packages (default /var/satellite): Depends on what you're storing; Red Hat recommend 6GB per channel for their channels
 * 2GB RAM minimum, 4GB recommended
 * If you use LDAP as a central identity service and wish to pull user and group information from it, see [[SpacewalkWithLDAP]]
# Setting up Spacewalk repo



RPM downloads of the project are available through a yum repository

  * http://spacewalk.redhat.com/yum/ - Binary RPMs
  * http://spacewalk.redhat.com/source/ - Source RPMs and tarballs

To use this repository install spacewalk-repo with commands below:
### Red Hat Enterprise Linux, CentOS




    rpm -Uvh http://spacewalk.redhat.com/yum/1.0/RHEL/5/i386/spacewalk-repo-1.0-2.el5.noarch.rpm
### Fedora 11




    rpm -Uvh http://spacewalk.redhat.com/yum/1.0/Fedora/11/i386/spacewalk-repo-1.0-2.fc11.noarch.rpm
### Fedora 12




    rpm -Uvh http://spacewalk.redhat.com/yum/1.0/Fedora/12/i386/spacewalk-repo-1.0-2.fc12.noarch.rpm
### Nightly builds



If you want to use the nightly builds, add a .repo file pointing to the nightly repository:


    cat > /etc/yum.repos.d/spacewalk.repo << 'EOF'
    [spacewalk]
    name=Spacewalk
    # RHEL 5 / CentOS 5
    baseurl=http://koji.spacewalkproject.org/spacewalk/split/spacewalk-5E/server/spacewalk-5E-1.1/$basearch/os/
    # Fedora 12
    #baseurl=http://koji.spacewalkproject.org/spacewalk/split/spacewalk-f12/server/spacewalk-f12-1.1/$basearch/os
    # Fedora 13
    #baseurl=http://koji.spacewalkproject.org/spacewalk/split/spacewalk-f13/server/spacewalk-f13-1.1/$basearch/os
    gpgkey=http://spacewalk.redhat.com/yum/RPM-GPG-KEY-spacewalk
    enabled=1
    gpgcheck=0
    EOF

*NOTE:* Nigthly repo contains developers snapshot and is not suitable in production environment.
# Additional repos & packages

### Fedora




For Spacewalk on Fedora, a couple of additional dependencies are needed from jpackage. Please create the following yum repository before beginning your Spacewalk installation:


    cat > /etc/yum.repos.d/jpackage.repo << EOF
    [jpackage]
    name=jpackage-f10
    baseurl=http://mirrors.dotsrc.org/jpackage/6.0/fedora-10/free/
    gpgkey=http://www.jpackage.org/jpackage.asc
    gpgcheck=1
    EOF

Note: the dirs for f11 and f12 do not contain the rpms yet
### Red Hat Enterprise Linux 5



Spacewalk requires a Java Virtual Machine with version 1.6.0 or greater.  [EPEL - Extra Packages for Enterprise Linux](http://fedoraproject.org/wiki/EPEL) contains a version of the openjdk that works with Spacewalk.  Other dependencies can get installed from EPEL as well.
To get packages from EPEL just install this RPM:


    BASEARCH=$(uname -i)
    rpm -Uvh http://download.fedora.redhat.com/pub/epel/5/$BASEARCH/epel-release-5-3.noarch.rpm

Please make sure you have only one java version installed. 
Otherwise you get error messeges like :
 * " bad major version at off set=6" or 
 * "INFO: validateJarFile(/usr/share/tomcat5/webapps/rhn/WEB-INF/lib/jspapi.jar) - jar not loaded."
### CentOS 5



Follow the instructions for *Red Hat Enterprise Linux 5* with the additions:
 * Necessary packages rhn-client-tools and rhnlib were removed from CentOS, they can be found in spacewalk-client repo. Setup it by installing spacewalk-client-repo package.

    rpm -Uvh http://spacewalk.redhat.com/yum/1.0/RHEL/5/i386/spacewalk-client-repo-1.0-2.el5.noarch.rpm
 *  Import Redhat's RPM GPG key:

    # wget -O /etc/pki/rpm-gpg/RPM-GPG-KEY-redhat-release http://www.redhat.com/security/37017186.txt
    # rpm --import /etc/pki/rpm-gpg/RPM-GPG-KEY-redhat-release
# Oracle Pre-Requisites



In order to get Spacewalk to run you need the Oracle client and an Oracle 10g database server. The easiest way to do this is to install Oracle XE and the Oracle Instant Client. Instructions on how to do this can be found on the [Oracle 10g Express Edition Setup](OracleXeSetup) page.
# Conflicting Packages

There is an open [bug 593402](https://bugzilla.redhat.com/show_bug.cgi?id=593402) that identifies the package cobbler-web as conflicting with Spacewalk's Taskomatic process. Until the issue is addressed, it is recommended that the cobbler-web package be removed from the Spacewalk server.


    yum remove cobbler-web

Yum can also be configured to exclude that package in the future by adding the following entry to the [[main]] section in /etc/yum.conf:

    vi /etc/yum.conf
    
    [main]
    ....
    exclude=cobbler-web
# Installing Spacewalk



Just ask yum to install the necessary packages. This will pull down and install the set of RPMs required to get Spacewalk to run.

*1.0 installation:*

If you tend to use the Oracle backend:

    yum install spacewalk-oracle 

If you prefer the PostgreSQL backend:

    yum install spacewalk-postgresql 

*Note:* PostgreSQL support is in early development stage. You may want to take a look at the [[PostgreSQL]] page for further information.
# Configuring Spacewalk



*Note*: Your Spacewalk server should have a resolvable FQDN such as 'hostname.domain.com'.  If the installer complains that the hostname is not the FQDN, do not use the --skip-fqdn-test flag to skip !  

Also, this will not work unless the spacewalk account in the database has a password.  You can set up a password by logging in to sqlplus with the sys account and using the command 


    PASSWORD user_name

where user_name = spacewalk.
*Don't use '#' (number sign/pound/hash) in your database password otherwise installation will fail.*

Once the Spacewalk RPM is installed you need to configure the application, you can run:


    spacewalk-setup --disconnected



An example session is as follows:


    [root@fjs-0-02 yum.repos.d]# spacewalk-setup --disconnected
    * Setting up Oracle environment.
    * Setting up database.
    ** Database: Setting up database connection.
    DB User? spacewalk
    DB Password? 
    DB SID? XE
    DB hostname? localhost
    DB port [1521]? 
    DB protocol [TCP]? 
    ** Database: Testing database connection.
    ** Database: Populating database.
    *** Progress: ##########################################################
    * Setting up users and groups.
    ** GPG: Initializing GPG and importing key.
    ** GPG: Creating /root/.gnupg directory
    You must enter an email address.
    Admin Email Address? tito@example.com
    * Performing initial configuration.
    * Activating Spacewalk.
    ** Loading Spacewalk Certificate.
    ** Verifying certificate locally.
    ** Activating Spacewalk.
    * Enabling Monitoring.
    * Configuring apache SSL virtual host.
    Should setup configure apache's default ssl server for you (saves original ssl.conf) y/n? y
    ** /etc/httpd/conf.d/ssl.conf has been backed up to ssl.conf-swsave
    * Creating SSL certificates.
    CA certificate password? 
    Re-enter CA certificate password? 
    Organization? Tito Puente Solutions
    Organization Unit [fjs-0-02.rhndev.redhat.com]? 
    Email Address [jesusr@redhat.com]? 
    City? Raleigh
    State? NC
    Country code (Examples: "US", "JP", "IN", or type "?" to see a list)? US
    ** SSL: Generating CA certificate.
    ** SSL: Deploying CA certificate.
    ** SSL: Generating server certificate.
    ** SSL: Storing SSL certificates.
    * Deploying configuration files.
    * Update configuration in database.
    * Setting up Cobbler..
    Cobbler requires tftp and xinetd services be turned on for PXE provisioning functionality. Enable these services (y/n, default = 'y')?y
    * Restarting services.
    Installation complete.
    Visit https://fjs-0-02.rhndev.redhat.com to create the Spacewalk administrator account.
### Configuring Spacewalk with an Answer File

You can also configure spacewalk by using an answer file.  To configure spacewalk by using an answer you just need to run the following


    spacwalk-setup --disconnected --answer-file=<FILENAME>
An example answer file is as follows

    admin-email = root@localhost
    ssl-set-org = Spacewalk Org
    ssl-set-org-unit = spacewalk
    ssl-set-city = My City
    ssl-set-state = My State
    ssl-set-country = US
    ssl-password = spacewalk
    ssl-set-email = root@localhost
    ssl-config-sslvhost = Y
    db-backend=oracle
    db-user=spacewalk
    db-password=spacewalk
    db-sid=xe
    db-host=localhost
    db-port=1521
    db-protocol=TCP
    enable-tftp=Y
*NOTE:* If you do not supply a value or leave out a key you will be prompted to supply that answer

After `spacewalk-setup` is complete your application is ready to go!
*Be sure to use the FQDN of the host when you connect.*

*NOTE:* If you notice you missing some buttons and channels that have been created aren't visible in you UI edit `/etc/sysconfig/httpd` and change:

    export ORACLE_HOME=/opt/oracle
    export NLS_LANG=english.AL32UTF8
to

    export ORACLE_HOME=/usr/lib/oracle/xe/app/oracle/product/10.2.0/server
    export NLS_LANG=AMERICAN_AMERICA.AL32UTF8

Once you've made sure you can login to the Spacewalk web UI, you can then proceed to the next step: [Uploading Content](UploadFedoraContent).
# Managing Spacewalk

Spacewalk consists of several services. Each of them has its own init.d script to stop/start/restart. If you want manage all spacewalk services at once use 


    /usr/sbin/rhn-satellite [stop|start|restart].
# Known Issues



 * For issues with Oracle XE, see [Oracle XE Setup Known Issues](OracleXeSetup)
 * For "Certificate Expiring" messages appearing in the GUI please see [this article](KnownIssues_CertExpiring).