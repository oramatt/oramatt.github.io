# Setting Up Oracle REST Data Services (ORDS) with the Oracle Database API for MongoDB

This document provides 'quick-start' instructions for installing and configuring **Oracle REST Data Services (ORDS)** to enable the **Oracle Database API for MongoDB** on an Oracle Database running on **Oracle Exadata Cloud Service (ExaCS)** or any supported Oracle environment. Please be aware this document is intended to assist in setting up a development environment and is not a replacement for official documentation. It is highly recommended to review official documentation prior to implementation on production systems.

---

## Download and Extract ORDS

- This option downloads from Oracle.com and extracts to the assumed oracle user's home directory
- Alternatively, one can download from an internet connected system and sftp to the Oracle system

```bash
wget -O /home/oracle/ords.zip https://download.oracle.com/otn_software/java/ords/ords-latest.zip
unzip /home/oracle/ords.zip -d /home/oracle/ords/
```

- This option installs ORDS with yum (requires yum repostiory setup)
```bash
sudo yum install -y ords
```

- This option install ORD with dnf (requires yum repostiory setup)
```bash
sudo dnf install -y ords
```

---

## Configure and Install ORDS
- This option, and the rest of this document, assumes installation in /home/oracle/ords
- Modify hostname, port, and other settings per site specific implementation
- This install includes features such as SQL Developer Web (feature-sdw) and Database API (feature-db-api)
- The following command will conduct a silent installation using a password redirect

```bash
echo 'Oradoc_db1' > /tmp/orclpwd
/home/oracle/ords/bin/ords --config /home/oracle/ords_config install \
  --admin-user SYS \
  --db-hostname localhost \
  --db-port 1521 \
  --db-servicename FREEPDB1 \
  --log-folder /tmp/ords_logs/ \
  --feature-sdw true  \
  --feature-db-api true \
  --feature-rest-enabled-sql true \
  --password-stdin </tmp/orclpwd
```

---

## Enable the MongoDB API
- This step will enable the MongoDB API listner on port 27017

```bash
/home/oracle/ords/bin/ords --config /home/oracle/ords_config config set mongo.enabled true
/home/oracle/ords/bin/ords --config /home/oracle/ords_config config set mongo.port 27017
```

Verify the configuration:
```bash
/home/oracle/ords/bin/ords --config /home/oracle/ords_config config info mongo.enabled
/home/oracle/ords/bin/ords --config /home/oracle/ords_config config info mongo.port
```

---

## Grant Necessary Database Privileges

### **Grant SODA and Required Permissions**
- Change user per site requirements
```bash
sqlplus sys/Oradoc_db1@'(DESCRIPTION=(ADDRESS=(PROTOCOL=TCP)(HOST=127.0.0.1)(PORT=1521))
    (CONNECT_DATA=(SERVER=DEDICATED)(SERVICE_NAME=FREEPDB1)))' as sysdba <<EOF
    grant soda_app, create session, create table, create view, create sequence, create procedure, create job, unlimited tablespace to matt;
    exit;
EOF
```

### **Enable ORDS for the Schema**
- Enabling the prior command's user
```bash
sqlplus matt/matt@'(DESCRIPTION=(ADDRESS=(PROTOCOL=TCP)(HOST=127.0.0.1)(PORT=1521))
    (CONNECT_DATA=(SERVER=DEDICATED)(SERVICE_NAME=FREEPDB1)))'<<EOF
    exec ords.enable_schema(true);
    exit;
EOF
```

### **Enable to connect to MongoDB API from different users**
- This assumes the user tim is created and has required permissions
```bash
sqlplus matt/matt@'(DESCRIPTION=(ADDRESS=(PROTOCOL=TCP)(HOST=127.0.0.1)(PORT=1521))
    (CONNECT_DATA=(SERVER=DEDICATED)(SERVICE_NAME=FREEPDB1)))'<<EOF
    execute ords.enable_schema('tim');
    exit;
EOF
```
---

## Start ORDS
- This command will start ORDS in standalone mode

```bash
echo "Attempting to start ORDS..."
/home/oracle/ords/bin/ords --config /home/oracle/ords_config serve > /dev/null 2>&1; sleep 10
```

---

## Verify MongoDB API Functionality
- These commands will create collections, insert a document, and query a document with MongoDB based tools connecting to the Oracle Database API for MongoDB

### **Create a JSON collection**
```bash
mongosh --tlsAllowInvalidCertificates 'mongodb://matt:matt@127.0.0.1:27017/matt?authMechanism=PLAIN&ssl=true&retryWrites=false&loadBalanced=true'<<EOF
    db.createCollection('test123');
EOF
```

### **Insert a JSON document**
```bash
mongosh --tlsAllowInvalidCertificates 'mongodb://matt:matt@127.0.0.1:27017/matt?authMechanism=PLAIN&ssl=true&retryWrites=false&loadBalanced=true'<<EOF
    db.test123.insertOne({ name: 'Matt DeMarco', email: 'matthew.demarco@oracle.com', notes: 'It is me' });
EOF
```

### **Read a JSON document**
```bash
mongosh --tlsAllowInvalidCertificates 'mongodb://matt:matt@127.0.0.1:27017/matt?authMechanism=PLAIN&ssl=true&retryWrites=false&loadBalanced=true'<<EOF
    db.test123.find().pretty();
EOF
```

---

## Summary

This guide walks through the installation, configuration, and verification of **ORDS with the Oracle Database API for MongoDB**. The **MongoDB API** allows MongoDB clients to interact directly with an Oracle Database instance using **MongoDB commands**, enabling **hybrid MongoDB and Oracle workloads**.

### **Next Steps:**
- Ensure **ORDS is running** and configured correctly.
- Integrate MongoDB-based applications with **Oracle Database API for MongoDB**.
- Migrate MongoDB JSON data to **Oracle Database API for MongoDB**.
- Monitor performance and optimize **JSON workloads** in Oracle.


### **Questions/problems/errors/need help**
- Please feel free to contact [Matt DeMarco](mailto:matthew.demarco@oracle.com)

---

## Appendix
1. ORDS system requirements checklist
    - https://docs.oracle.com/en/database/oracle/oracle-rest-data-services/24.4/ordig/installing-REST-data-services.html

2. Installing & Configuring ORDS
    - https://docs.oracle.com/en/database/oracle/oracle-rest-data-services/24.4/ordig/installing-and-configuring-oracle-rest-data-services.html

3. Enabling & Configuring Oracle Database API for MongoDB
    - https://docs.oracle.com/en/database/oracle/oracle-rest-data-services/24.4/ordig/enabling-and-configuring-oracle-database-api-mongodb.html

4. Useful settings for high performance
    - https://docs.oracle.com/en/database/oracle/oracle-rest-data-services/24.4/ordig/enabling-and-configuring-oracle-database-api-mongodb.html#GUID-8A4B6CB7-CDA3-49F6-AFBA-D4CBEA1A188F

```bash
/home/oracle/ords/bin/ords config set jdbc.MaxConnectionReuseCount 5000
/home/oracle/ords/bin/ords config set jdbc.MaxConnectionReuseTime 900
/home/oracle/ords/bin/ords config set jdbc.SecondsToTrustIdleConnection 1
/home/oracle/ords/bin/ords config set jdbc.InitialLimit 100
/home/oracle/ords/bin/ords config set jdbc.MaxLimit 100
```

5. Oracle Database API for MongoDB
    - https://docs.oracle.com/en/database/oracle/mongodb-api/mgapi/index.html

6. Oracle Database API for MongoDB launch blog post
    - https://blogs.oracle.com/database/post/installing-database-api-for-mongodb-for-any-oracle-database

7. Modify various ORDS features as needed
```bash
# not needed for mongoapi
/home/oracle/ords/bin/ords --config /home/oracle/ords_config config set feature.sdw false
/home/oracle/ords/bin/ords --config /home/oracle/ords_config config info feature.sdw
```
```bash
# not needed for mongoapi
/home/oracle/ords/bin/ords --config /home/oracle/ords_config config set database.api.enabled false
/home/oracle/ords/bin/ords --config /home/oracle/ords_config config info database.api.enabled
```
```bash
# needed for mongoapi, do not disable
/home/oracle/ords/bin/ords --config /home/oracle/ords_config config set restEnabledSql.active true
/home/oracle/ords/bin/ords --config /home/oracle/ords_config config info restEnabledSql.active
```

8. ORDS uninstall command
```bash
/home/oracle/ords/bin/ords --config /home/oracle/ords_config uninstall \
--admin-user SYS \
--db-hostname localhost \
--db-port 1521 \
--db-servicename FREEPDB1 \
--log-folder /tmp/ords_logs/ \
--force \
--password-stdin </tmp/orclpwd
```
