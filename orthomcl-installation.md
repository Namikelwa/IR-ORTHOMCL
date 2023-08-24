# Installing the OrthoMCL Pipeline
## Installing the OrthoMCL Pipeline can be accomplished by downloading the code with the following command and then following the steps below.

'''
$ git clone https://github.com/apetkau/orthomcl-pipeline.git
'''

**Step 1: Perl Dependencies**
The OrthoMCL Pipeline requires Perl as well as the following Perl modules.

BioPerl
DBD::mysql
DBI
Parallel::ForkManager
Schedule::DRMAAc
YAML::Tiny
Set::Scalar
Text::Table
Moose
SVG
Algorithm::Combinatorics
These can be installed with cpanm using:


'''
sudo apt install perl
sudo apt-get update
sudo apt-get install build-essential
sudo cpan YAML
sudo cpan App::cpanminus
'''
'''
cpanm BioPerl DBD::mysql DBI Parallel::ForkManager YAML::Tiny Set::Scalar Text::Table Exception::Class Test::Most Test::Warn Test::Exception Test::Deep Moose SVG Algorithm::Combinatorics
'''

If the module installation code above throws errors, go through the error messages and install the required dependencies as instructed. For my case, I ran the codes below to install dependencies required by the modules needed and then I ran the installation code again.
 
'''
sudo apt-get install libexpat1-dev
sudo cpanm XML::Parser
sudo apt-get install libssl-dev
sudo apt-get install zlib1g-dev
sudo cpanm Net::SSLeay
sudo cpanm IO::Socket::SSL
sudo apt-get install libmysqlclient-dev #to solve mysql_config issue
'''



If you wish to use a grid engine to submit jobs then Schedule::DRMAAc must be installed and the parameter sge must be used for the scheduler in the config file and tests. This must be done manually and requires installing a grid engine. A useful guide for how to install a grid engine on Ubuntu can be found at http://scidom.wordpress.com/2012/01/18/sge-on-single-pc/.

Step 2: Other Dependencies
Additional software dependencies for the pipeline are as follows:

OrthoMCL or OrthoMCL Custom (changes to characters defining sequence identifiers)
BLAST (blastall, formatdb). We have found version 2.2.26 to work best. Older verions may not work correctly (see issue #7).
MCL
The paths to the software dependencies must be setup within the etc/orthomcl-pipeline.conf file. These software dependencies can be checked and the configuration file created using the scripts/setup.pl script as below:

$ perl scripts/orthomcl-pipeline-setup.pl
Checking for Software dependencies...
Checking for OthoMCL ... OK
Checking for formatdb ... OK
Checking for blastall ... OK
Checking for mcl ... OK
Wrote new configuration to orthomcl-pipeline/scripts/../etc/orthomcl-pipeline.conf
Wrote executable file to orthomcl-pipeline/scripts/../bin/orthomcl-pipeline
Please add directory orthomcl-pipeline/scripts/../bin to PATH
The configuration file etc/orthomcl-pipeline.conf generated looks like:

---
blast:
  F: 'm S'
  b: 100000
  e: 1e-5
  v: 100000
filter:
  max_percent_stop: 20
  min_length: 10
mcl:
  inflation: 1.5
path:
  blastall: '/usr/bin/blastall'
  formatdb: '/usr/bin/formatdb'
  mcl: '/usr/local/bin/mcl'
  orthomcl: '/home/aaron/software/orthomcl/bin'
scheduler: fork
split: 4
The parameters in this file can be adjusted to fine-tune the pipeline. In particular, you may want to adjust the split: 4 parameter to a reasonable value. This corresponds to the default number of processing cores to use for the BLAST stage (defines the number of chunks to split the FASTA file into).

You may also want to adjust the scheduler: fork to scheduler: sge if you are attempting to use a grid scheduler (with DRMAAc) to run OrthoMCL.

Step 3: Database Setup
The OrthoMCL also requires a MySQL database to be setup in order to load and process some of the results. An account needs to be created specifically for OrthoMCL. A special OrthoMCL configuration file needs to be generated with parameters and database connection information. This can be generated automatically with the script scripts/setup_database.pl. There are two options for running this script as outlined below:

Option 1: If you have a previously created database you can run the script with the option: --no-create-database

$ perl scripts/orthomcl-setup-database.pl --user orthomcl --password orthomcl --host localhost --database orthomcl --outfile orthomcl.conf --no-create-database
Connecting to database orthmcl on host orthodb with user orthomcl ...
OK
Config file **orthomcl.conf** created.
Option 2: If you want the script to create the datbase for you, run the script without --no-create-database. Prior to running the script the OrthoMCL account must be granted SELECT, INSERT, UPDATE, DELETE, CREATE, CREATE VIEW, INDEX and DROP permissions by logging into the MySQL server as root and executing the following command:

mysql> GRANT SELECT, INSERT, UPDATE, DELETE, CREATE, CREATE VIEW, INDEX, DROP on *.* to orthomcl;
Once the user account is setup, the database can be generated with the same script that creates the configuration file as follows:

$ perl scripts/orthomcl-setup-database.pl --user orthomcl --password orthomcl --host localhost --database orthomcl --outfile orthomcl.conf
Connecting to mysql and creating database **orthmcldb** on host orthodb with user orthomcl ...
OK
database orthmcl created ...OK
Config file **orthomcl.conf** created.
Either option will generate a file orthomcl.conf with database connection information and other parameters. This file looks like:

coOrthologTable=CoOrtholog
dbConnectString=dbi:mysql:orthomcl:localhost:mysql_local_infile=1
dbLogin=orthomcl
dbPassword=orthomcl
dbVendor=mysql 
evalueExponentCutoff=-5
inParalogTable=InParalog
interTaxonMatchView=InterTaxonMatch
oracleIndexTblSpc=NONE
orthologTable=Ortholog
percentMatchCutoff=50
similarSequencesTable=SimilarSequences
Step 4: Testing
Once the OrthoMCL configuration file is generated a full test of the pipeline can be run as follows:

$ perl t/test_pipeline.pl -m orthomcl.conf -s fork -t /tmp
Test using scheduler fork

TESTING NON-COMPLIANT INPUT
TESTING FULL PIPELINE RUN 3
README:
Tests case of one gene (in 1.fasta and 2.fasta) not present in other files.
ok 1 - Expected matched returned groups file
...
Once all tests have passed then you are ready to start using the OrthoMCL pipeline. If you wish to test the grid scheduler mode of the pipeline please change -s fork to -s sge and re-run the tests.

Step 5: Running
You should now be able to run the pipeline with:

$ ./bin/orthomcl-pipeline
Error: no input-dir defined
Usage: orthomcl-pipeline -i [input dir] -o [output dir] -m [orthmcl config] [Options]
...
You can now follow the main instructions for how to perform OrthoMCL analyses.






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


**IF YOU GET AN ERROR RUNNING CHECKINSTALL, TRY THIS:**
```
make install

sudo checkinstall --install=no --pkgname orthomcl --pkgversion 2.0.9 --pkgrelease 1 --pkglicense gpl --maintainer "selinemnams@gmail.com" --provides orthomcl --exclude /usr/local/lib/perl/5.34.0/perllocal.pod
```

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
