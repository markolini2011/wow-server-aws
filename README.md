# wow-server-aws
WoW-WoTlk-Server
World Of Warcraft server setup on AWS

Overview
This document outlines the setup of a private World of Warcraft server using AzerothCore on Amazon Web Services (AWS). The setup uses an EC2 instance for the server, an RDS instance for the database, and specific tools for compilation, data extraction, and database management.

  AWS Components
  
    •	Amazon EC2: Hosts the AzerothCore server binaries.
    •	Amazon RDS: Manages MySQL databases (ac_auth, ac_characters, ac_world).
    •	VPC Configuration:
    CIDR: 10.0.0.0/16
    •	Subnets: 10.0.1.0/24 and 10.0.2.0/24 (configured for inter-subnet communication via route tables and security groups).
    •	Security Groups: Allow MySQL (port 3306) for RDS access and necessary ports for WoW server (e.g., 8085 for game client).
    Tools Used
    •	AzerothCore: Open-source WoW server framework.
    •	Git: For cloning the AzerothCore repository.
    •	CMake: For configuring and generating build files.
    •	Make: For compiling AzerothCore with make -j2.
    •	MySQL Client: For connecting to RDS and importing SQL files.
    •	WoW Client Data: Extracted DBC, maps, vmaps, and mmaps from a WoW client (version compatible with AzerothCore).
    •	AWS CLI: For managing EC2 and RDS resources.
    •	Linux (Amazon Linux 2): Operating system on EC2 for server hosting.


Setup Steps:

    1. EC2 Instance Setup
    •	Launched an EC2 instance (e.g., m7i-flex-large) with Amazon Linux 2.
    •	Installed dependencies: git, cmake, make, gcc, mysql-client, and others required by AzerothCore.
    •	Cloned AzerothCore repository: git clone https://github.com/azerothcore/azerothcore-wotlk.git.
    •	Configured build with CMake: cmake ../ -DCMAKE_INSTALL_PREFIX=/home/ec2-user/azerothcore -DTOOLS=1.
    •	Compiled server binaries: make -j2 && make install.

2. Game Data Extraction

    •	Downloaded WoW client data (DBC, maps, vmaps, mmaps) from a compatible WoW client or community sources.
    •	Extracted data using AzerothCore’s map extractor tools and placed it in the server’s data directory (/home/ec2-user/azerothcore/data).
    
3. RDS Database Setup
    
    •	Created an RDS MySQL instance with endpoint (e.g., wow-db.<region>.rds.amazonaws.com).
    •	Configured security groups to allow EC2-to-RDS communication (port 3306).
    •	Created databases: ac_auth, ac_characters, ac_world using MySQL client:
    
    CREATE DATABASE ac_auth;
    CREATE DATABASE ac_characters;
    CREATE DATABASE ac_world;

4. SQL Data Import

    •	Manually imported ac_auth SQL file to RDS from EC2:
    mysql -h <RDS_ENDPOINT> -u <USERNAME> -p ac_auth < /path/to/ac_auth.sql
    
    •	Automated import for ac_characters and ac_world using a Bash script:
    
    #!/bin/bash
    RDS_ENDPOINT="wow-db.<region>.rds.amazonaws.com"
    USERNAME="<your_username>"
    PASSWORD="<your_password>"
    SQL_DIR="/path/to/azerothcore/sql"
    
    for db in ac_characters ac_world; do
    mysql -h $RDS_ENDPOINT -u $USERNAME -p$PASSWORD $db < $SQL_DIR/$db.sql
    done
    
    •	Executed script: bash import_sql.sh


5. Network Configuration

    •	Configured VPC with subnets 10.0.1.0/24 and 10.0.2.0/24.
    •	Updated route tables to enable inter-subnet communication
    •	Set up security groups to allow traffic:
    EC2: Inbound rules for WoW server ports (e.g., 8085).
    RDS: Inbound rule for MySQL (port 3306) from EC2’s security group


6. Server Configuration

    •	Edited AzerothCore configuration files (worldserver.conf, authserver.conf) to point to RDS endpoint:
    MySQLHost = wow-db.<region>.rds.amazonaws.com
    MySQLUser = <your_username>
    MySQLPassword = <your_password>
    •	Started authserver and worldserver: ./authserver and ./worldserver.
   	•	Connected to the WoW Server with a WoW game client and verified connectivity
