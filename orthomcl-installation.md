# These are the steps I followed to install OrthoMCL on an Ubuntu 14.04 Linux server using MySQL as the database system.

## Requirements

Ubuntu 14.04 (tested). Might work on other versions.
All these steps are done with an user account with admin privileges (sudo).
Install MySQL database server (This is an OrthoMCL requirement):
```
  sudo apt install mysql-server
  sudo service mysql start
```
The install step will prompt for a password for the MySQL administrator.

Install wget and checkinstall. We will use checkinstall to build a debian package (.deb) of OrthoMCL that can be installed, updated or removed easily:
```
sudo apt install wget checkinstall

```
Download and uncompress the OrthoMCL archive
```
wget http://orthomcl.org/common/downloads/software/v2.0/orthomclSoftware-v2.0.9.tar.gz
tar zxvf orthomclSoftware-v2.0.9.tar.gz
```
Create a Makefile

To create a debian package using checkinstall, an install step is required (ex: make install). The make command requires a Makefile. As OrthoMCL is a PERL package, we can generate a Makefile by first creating a Makefile.PL.

Change directory:
```
cd orthomclSoftware-v2.0.9
```
Using a text editor (ex: nano), create a new file with the following content and save it as Makefile.PL:
```
use ExtUtils::MakeMaker;

WriteMakefile(
NAME       => 'OrthoMCLEngine::Main::Base',
VERSION    => '2.0.9',
PM         => {'lib/perl/OrthoMCLEngine/Main/Base.pm' => '$(INST_LIB)/OrthoMCLEngine/Main/Base.pm'},
EXE_FILES => ['bin/orthomclAdjustFasta',
'bin/orthomclBlastParser',
'bin/orthomclDropSchema',
'bin/orthomclDumpPairsFiles',
'bin/orthomclExtractProteinIdsFromGroupsFile',
'bin/orthomclExtractProteinPairsFromGroupsFile',
'bin/orthomclFilterFasta',
'bin/orthomclInstallSchema',
'bin/orthomclInstallSchema.sql',
'bin/orthomclLoadBlast',
'bin/orthomclLoadBlast.sql',
'bin/orthomclMclToGroups',
'bin/orthomclPairs',
'bin/orthomclReduceFasta',
'bin/orthomclReduceGroups',
'bin/orthomclRemoveIdenticalGroups',
'bin/orthomclSingletons',
'bin/orthomclSortGroupMembersByScore',
'bin/orthomclSortGroupsFile']
);
```
Create a Debian package using checkinstall

Use PERL to generate a Makefile
```
 perl Makefile.PL
```
Use the generated Makefile to compile source code:
```
 make
```
Create a text file called description-pak with a description for the package like the following:

 An algorithm for grouping proteins into ortholog groups based on their sequence similarity
Create a directory called doc-pak and copy included documentation.
```
 mkdir doc-pak
 cp -r doc/OrthoMCLEngine/Main/* doc-pak
```
Run checkinstall to create debian package (note: change email address under maintainer)

```
 sudo checkinstall --install=no --pkgname orthomcl \
 --pkgversion 2.0.9 --pkgrelease 1 --pkglicense gpl \
 --maintainer "selinenams@gmail.com" --provides orthomcl \
 --exclude /usr/local/lib/perl/5.18.2/perllocal.pod make install
```
Press ENTER to build the package.


IF YOU GET AN ERROR TRY THIS; 
```
make install
```
sudo checkinstall --install=no --pkgname orthomcl --pkgversion 2.0.9 --pkgrelease 1 --pkglicense gpl --maintainer "selinemnams@gmail.com" --provides orthomcl --exclude /usr/local/lib/perl/5.34.0/perllocal.pod


Install generated package
```
sudo dpkg -i orthomcl_2.0.9-1_amd64.deb
```
Create a database, configuration file and install schema

Create the database, the database user, and grant this user permissions to use/update the database.
```
  mysql -u root -p
  or
  sudo mysql --user=root mysql

 create database `orthomcl` character set = 'utf8';

 create user 'orthomcl'@'localhost' identified by 'your-password-here';

 grant SELECT, INSERT, UPDATE, DELETE, CREATE VIEW, CREATE, INDEX, DROP on `orthomcl`.* to `orthomcl`@localhost;

 quit
```
For the orthoMclLoadBlast command to work, the local-infile option has to be enabled. Without this option, the command fails with an error - used command is not allowed with this MySQL version. To prevent this, create /etc/mysql/conf.d/client.cnf with the following content (creating this file requires sudo/root privileges):

```
[client]
loose-local-infile=1
```
Restart MySQL server for the change to take effect:
```
 sudo service mysql restart
```
Most OrthoMCL commands require a configuration file. We can copy the configuration template that is distributed with the software, make changes according to our configuration (i.e., database name, user, password) and then provide that configuration file to the commands.

Copy template:
```
 sudo cp /usr/share/doc/orthomcl/orthomcl.config.template /etc/orthomcl.config
```
Edit this file using a text editor and update configuration:
```
  dbConnectString=dbi:mysql:orthomcl:mysql_local_infile=1
  dbLogin=orthomcl
  dbPassword=your-password-here
```
Install schema:
```
 sudo orthomclInstallSchema /etc/orthomcl.config install_schema.log
```
This command should complete without any errors.

(Optional) Create an orthomcl group

This step is optional and is not in the official documentation. When multiple users require access to the database, they can be added to an user group. Only members in this group will have read access to /etc/orthomcl.config. Here is an example:
```
sudo groupadd orthomcl
sudo gpasswd -a vimal orthomcl
```
Now update permissions on the file /etc/orthomcl.config:
```
sudo chgrp orthomcl /etc/orthomcl.config
sudo chmod 640 /etc/orthomcl.config
```


Download MCL
```
sudo apt install mcl
```
