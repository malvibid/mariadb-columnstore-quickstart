# MariaDB ColumnStore Quick Start

[MariaDB ColumnStore](https://mariadb.com/docs/features/mariadb-columnstore/) is a columnar storage engine that utilizes a massively parallel distributed data architecture. ColumnStore is designed for big data scaling to process petabytes of data, linear scalability and exceptional performance with real-time response to analytical queries.

**This README provides steps to get you up and running with MariaDB ColumnStore using a [Docker](https://www.docker.com/) container**

## Prerequisites

Before getting started with this walkthrough you will need:

-   [Docker Desktop](https://www.docker.com/get-started)
-   [MariaDB command-line client](https://mariadb.com/products/skysql/docs/connect/clients/mariadb-client/) (optional)
-   [git](https://git-scm.com/) (optional)

## Overview

This walkthrough will step you through the process of installing, accessing and configuring single-node MariaDB ColumnStore instance, made available through MariaDB Community Server within a Docker container.

**Note:** To use this download, import and use this data on an existing, non-container MariaDB Server instance (including [MariaDB SkySQL](https://mariadb.com/skyview)) jump to step #4.

## 1. INITIAL SET UP

### Pull Docker image and create a Docker container

Pull down the [MariaDB Community Server (with ColumnStore) image](https://hub.docker.com/r/mariadb/columnstore) and create a new container by executing the following command in a terminal window:

Add `-d` if you don't want to observe its output.

```bash
$ docker run -p 3306:3306 --name mcs_container mariadb/columnstore
```

---

### USEFUL COMMANDS

### To enter the newly created container:

```bash
$ docker exec -it mcs_container bash
```

### To launch the MariaDB client in the container:

```bash
$ docker exec -it mcs_container mariadb
```

If already inside the container can simply launch the MariaDB client by typing the command: `mariadb`

### In order to log in to a specific user account:

```
mariadb -u<user> -p
```

### In order to exit MariaDB client:

```
exit
```

---

**Note:** The next several steps involve work within the Docker container, but that is not a hard requirement. The scripts within this repository will also work outside of the container as well.

### **Inside the MariaDB client in the container:**

### Set password for root user

```
mariadb ALTER USER 'root'@'localhost' IDENTIFIED BY '<new_password>';
```

### Create new user and grant priviledges

```
mariadb GRANT ALL PRIVILEGES ON travel.* TO '<user>'@'%' IDENTIFIED BY '<password>';
```

### Test access from outside the container - for instance with MariaDB client from the host

```
mariadb --protocol tcp --host localhost -u <user> --password=<password>
```

## 2. SET UP THE DATABASE

### **Within the container (in bash) (should have exited the MariaDB client):**

### Install [git](https://git-scm.com/) and [wget](https://www.gnu.org/software/wget/) using [yum](http://yum.baseurl.org/)

```
yum install git wget
```

### Clone the MariaDB Columnstore repository

```
git clone https://github.com/malvibid/mariadb-columnstore-quickstart
```

### Change directory to the project folder

```
cd mariadb-columnstore-quickstart
```

### Download flight data

The sample data used in this example comes from the United States Department of Transportation, which provides millions of records of flight data spanning many years.

Use the following command to execute a script that will download US domestic flight data between a `start` and `end` year.

```
./get_flight_data.sh
```

**Note:** Keep in mind that there are millions of flight records that can take up gigabytes of storage space.

### Create schemas and load data

This repository includes the following schema:

-   travel (`database`)
    -   **airlines** (`ColumnStore table`) - airlines providing flights within the United States
    -   **airports** (`ColumnStore table`) - airports within the United States
    -   **flights** (`ColumnStore table`) - US domestic flight records

In this sample, the [create_and_load.sh](create_and_load.sh) script will be used to create the schemas (via [schema.sql](schema.sql)) and load the following tables:

-   travel.airlines - using [data/airlines.csv](data/airlines.csv)
-   travel.airports - using [data/airports.csv](data/airports.csv)
-   travel.flights - using data that is downloaded with the [get_flight_data.sh](get_flight_data.sh) script

The [create_and_load.sh](create_and_load.sh) script can be used with or without specifying database details like `host`, `port`, `user`, and `password`, that is:

./create_and_load.sh

or

./create_and_load.sh [host] [port_number] [user] [password]

Execute the following command to execute a script to create the schema and load data.

```
./create_and_load.sh 127.0.0.1 3306 <user> <password>
```

#### **Additional uses**

The script can also be used with a MariaDB SkySQL database (by including a path to the [Certificate authority chain file](https://mariadb.com/products/skysql/docs/instructions/connecting/)). For example:

./create_and_load.sh [host] [port_number] [user] [password] [ca_file_path]

```bash
$ ./create_and_load.sh analytics-demo.mdb0001390.db.skysql.net 5001 DB00003799 Password123 skysql_chain.pem
```

## 3. TEST CONNECTION

### **Test access from outside the container - with MariaDB client from the host**

```
mariadb --protocol tcp --host localhost -u <user> --password=<password>
```

### **Test access from outside the container - with Python client from the host**

Using TestPythonConnector.ipynb

### Clone project in host machine and open in VSCode

```
git clone https://github.com/malvibid/mariadb-columnstore-quickstart
```

### Create Python Virtual Environment and activate it (Configure auto activate for venv in vscode)

```
py -m venv venv

venv\Scripts\activate
```

### Install dependencies inside venv (Interpreter should be pointed to python in the venv)

```
pip install -r requirements.txt
```

### Set the Database Credentials in colStore.properties

```
DB_HOST=127.0.0.1
DB_PORT=3306
DB_USER=<user>
DB_PASSWORD=<password>
DB_DATABASE=travel
```

### Run and test connection

Should connect to 'travel' database and get query results successfully.

## Support and Contribution <a name="support-contribution"></a>

This is a fork strictly for personal and academic use. This fork is **not** officially affiliated or endorsed by MariaDB. Therefore, any issues from changes made in this repository **must not** be reported to MariaDB.

## License <a name="license"></a>

[![License](https://img.shields.io/badge/License-MIT-blue.svg?style=plastic)](https://opensource.org/licenses/MIT)
