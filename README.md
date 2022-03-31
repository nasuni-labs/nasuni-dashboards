About
=====

Nasuni Performance Monitoring - A modern monitoring architecture.

Components
===========

*   Telegraf is an agent for collecting, processing, aggregating, and writing metrics.
    
*   InfluxDB is an open-source time series database for saving data.
    
*   Grafana is a multi-platform open-source analytics and interactive visualization web application for displaying data.
    

![TIGStackComponents](/images/TIGStackComponents.png)

Sizing Guidelines
==================

InfluxDB
--------

InfluxDB is designed to run-on solid-state drives (SSDs) and memory-optimized instances.

<table data-layout="default" data-local-id="8e403bb5-545e-4876-8e57-082bc184df20" class="confluenceTable"><colgroup><col style="width: 113.33px;"><col style="width: 113.33px;"><col style="width: 113.33px;"><col style="width: 113.33px;"><col style="width: 113.33px;"><col style="width: 113.33px;"></colgroup><tbody><tr><th class="confluenceTh"><p style="text-align: center;"><strong>vCPU or CPU</strong></p><p style="text-align: center;">(Cores)</p></th><th class="confluenceTh"><p style="text-align: center;"><strong>RAM</strong></p><p style="text-align: center;">(GB)</p></th><th class="confluenceTh"><p style="text-align: center;"><strong>IOPS</strong></p></th><th class="confluenceTh"><p style="text-align: center;"><strong>Writes per second</strong></p></th><th class="confluenceTh"><p style="text-align: center;"><strong>Queries per second</strong></p></th><th class="confluenceTh"><p style="text-align: center;"><strong>Unique</strong></p><p style="text-align: center;"><strong>series</strong></p></th></tr><tr><td class="confluenceTd"><p style="text-align: center;">2-4</p></td><td class="confluenceTd"><p style="text-align: center;">2-4</p></td><td class="confluenceTd"><p style="text-align: center;">500</p></td><td class="confluenceTd"><p style="text-align: center;">&lt; 5,000</p></td><td class="confluenceTd"><p style="text-align: center;">&lt; 5</p></td><td class="confluenceTd"><p style="text-align: center;">&lt; 100,000</p></td></tr><tr><td class="confluenceTd"><p style="text-align: center;">4-6</p></td><td class="confluenceTd"><p style="text-align: center;">8-32</p></td><td class="confluenceTd"><p style="text-align: center;">500-1000</p></td><td class="confluenceTd"><p style="text-align: center;">&lt; 250,000</p></td><td class="confluenceTd"><p style="text-align: center;">&lt; 25</p></td><td class="confluenceTd"><p style="text-align: center;">&lt; 1,000,000</p></td></tr><tr><td class="confluenceTd"><p style="text-align: center;">8+</p></td><td class="confluenceTd"><p style="text-align: center;">32+</p></td><td class="confluenceTd"><p style="text-align: center;">1000+</p></td><td class="confluenceTd"><p style="text-align: center;">&gt; 250,000</p></td><td class="confluenceTd"><p style="text-align: center;">&gt; 25</p></td><td class="confluenceTd"><p style="text-align: center;">&gt; 1,000,000</p></td></tr></tbody></table>

Grafana
-------

*   Grafana does not use a lot of resources and is very lightweight in use of memory and CPU.
    
*   Minimum recommended memory: 255 MB
    
*   Minimum recommended CPU: 1
    

Single Node Installation VM Requirements
----------------------------------------

*   OS: Rocky Linux 8.5
    
*   Sizing: 4 vCPU, 16GB RAM, and 100GB boot disk
    

Installation
============

Configure SNMP for Nasuni Appliances
------------------------------------

### Configure SNMP for the NMC

1.  Log in to the Nasuni Management Console.
    
2.  Click **Console Settings** in the top bar.
    
3.  Click **SNMP Monitoring** on the left navigation menu under **Console Settings**.
    
4.  Enable your desired SNMP protocol version(s):

    a. SNMPv3 (most secure): Ensure that **Enable v3 Support** is set to **On** and configure a **Username** and **Password**.

    b. SNMPv1/v2: Ensure that **Enable v1,v2c Support** is set to **On** and set the **Community Name** to **public** (or what you plan to use).
    
6.  Set your preferred **System Location** and **System Contact**.
    
7.  Click **Save SNMP Settings**.

### Configure SNMP for Edge Appliances
    
1.  In the NMC, click **Filers** in the top bar.
    
2.  Click **SNMP Settings** on the left navigation menu under **Filer Settings**.
    
3.  For each Edge Appliance that should be monitored, select the Edge Appliance(s) (Note: you can choose multiple Edge Appliances if all values are shared).
    
4.  Click the **Edit {x} Filers** button in the upper right.
    
5.  Enable your desired SNMP protocol version(s):

    a. SNMPv3 (most secure): Ensure that **Enable v3 Support** is set to **On** and configure a **Username** and **Password**.

    b. SNMPv1/v2: Ensure that **Enable v1,v2c Support** is set to **On** and set the **Community Name** to **public** (or what you plan to use).
    
6.  Set your preferred **System Location** and **System Contact**.
    
7.  Click **Save SNMP Settings**.
    

Install & Configure Influx DB
-----------------------------

### Install InfluxDB

To install InfluxDB, ssh to the VM and run the following commands:

1.  Update all packages:
    
    ```shell
    sudo yum update
    ```
    
2.  Install wget (if not present):
    
    ```shell
    sudo yum install wget
    ```
    
3.  Use wget to download InfluxDB:
    
    ```shell
    wget https://dl.influxdata.com/influxdb/releases/influxdb-1.7.9.x86_64.rpm
    ```
    
4.  Install InfluxDB:
    
    ```shell
    sudo yum localinstall influxdb-1.7.9.x86_64.rpm
    ```
    
5.  Start and enable the InfluxDB service:
    
    ```shell
    sudo systemctl start influxdb && sudo systemctl enable influxdb
    ```
    
6.  Add firewall rules if the firewall service is running. Check the firewall service:

    ```shell
    systemctl status firewalld
    ```
    
    If the firewall is running, the output will be similar to the following:
    ```shell
    firewalld.service - firewalld - dynamic firewall daemon
    Loaded: loaded (/usr/lib/systemd/system/firewalld.service; enabled; vendor pr
    Active: active (running) since Mon 2021-12-18 16:05:15 CET; 50min ago
    Docs: man:firewalld(1)
    ```
    
    Configure the firewall for InfluxDB (only required if the firewall was running) and reload the firewall configuration:
    
    ```shell
    sudo firewall-cmd --add-port=8086/tcp --permanent
    sudo firewall-cmd --reload
    ```
    
### Create the InfluxDB database:
    
1.  Open the InfluxDB shell:

    ```shell
    influx
    ```

2.  Create the database for use by Grafana.

    ```shell
    create database nasuni
    ```

3.  Confirm that the database was created:

    ```shell
    show databases
    ```

4.  Confirm that “nasuni” is listed.

5.  Exit InfluxDB shell:

    ```shell
    exit
    ```
        

Install & Configure Telegraf
----------------------------

### Install Telegraf

1.  Use wget to download Telegraf:
    
    ```shell
    wget https://dl.influxdata.com/telegraf/releases/telegraf-1.13.0-1.x86_64.rpm
    ```
    
2.  Install Telegraf:
    
    ```shell
    sudo yum localinstall telegraf-1.13.0-1.x86_64.rpm
    ```
    
3.  Install SNMP dependencies for Telegraf:
    
    ```shell
    sudo yum -y install net-snmp net-snmp-utils
    ```
    

### Add Nasuni SNMP MIBs to Telegraf

1.  On your computer, log in to the NMC and navigate to **Filers > SNMP Settings**.
    
2.  Click the **NASUNI-FILER-MIB** link to open the file using a text editor such as Notepad.
    
3.  Select all and copy text.
    
4.  Return to VM ssh session and navigate to the mibs directory:
    
    ```shell
    cd /usr/share/snmp/mibs
    ```
    
5.  Create a new file for the Nasuni MIB:
    
    ```shell
    sudo vi NASUNI-FILER-MIB.txt
    ```
    
6.  Paste the text copied in step 3.
    
7.  Save and close the file. Press the **Esc** key, type **:x**, and hit **Enter**

8.  Confirm that the other Nasuni-required MIBs are installed:

    ```shell
    ls NASUNI-FILER-MIB.txt SNMPv2-MIB.txt HOST-RESOURCES-MIB.txt UCD-SNMP-MIB.txt UCD-DISKIO-MIB.txt IF-MIB.txt
    ```

    The output should match the following and not return any lines that read **cannot access x: no such file or directory**:

    ```shell
    NASUNI-FILER-MIB.txt SNMPv2-MIB.txt HOST-RESOURCES-MIB.txt UCD-SNMP-MIB.txt UCD-DISKIO-MIB.txt IF-MIB.txt
    ```
        
9.  If any MIBs were missing from the list, repeat steps 5-7 for each missing MIBs and confirm they are installed using the command in step 8.
    

### Telegraf Configuration

1.  Backup the default telegraf configuration file
    ```shell
    sudo mv /etc/telegraf/telegraf.conf /etc/telegraf/telegraf.conf.bak
    ```

2.  On your computer, download [telegraf.conf](telegraf.conf) from Nasuni Labs, open it in a text editor, select all, and copy the contents to your clipboard.

3.  Return to VM ssh session, open vi, and paste the contents from step 2, customizing the values for the sections below:
    
    ```shell
    sudo vi /etc/telegraf/telegraf.conf
    ```
    
4.  In the **[outputs.influxdb]** section update the **urls** value with the InfluxDB IP address & confirm the port. You can use the search function in vi to jump to the appropriate section of the file. Type "/" followed by the string you want to search for, and then press Return. The following example assumes the default port and InfluxDB running on the same host as Telegraf.</br>
    
    ```shell
    urls = ["http://127.0.0.1:8086"]
    ```
        
5.  In the **[inputs.snmp]** section, update the **agents** value with the FQDN of all filers to be monitored. For example (note that the last entry does not need a trailing comma):</br>
    
    ```shell
       agents = [
            "Edge1.yourdomain.com",
            "Edge2.yourdomain.com",
            "Edge3.yourdomain.com"    
        ]
    ```
            
6.  In the **[inputs.snmp]** section, update protocol-specific values:</br></br>
    A. SNMPv3</br></br>
        ```shell
        sec_name = "username"</br>
        auth_password = "password"
        ```
        </br></br>
    B. SNMPv2</br>
        ```shell
        version = 2</br>
        community = "communityName"
        ```
        </br></br>

7.  In the **[inputs.snmp.tags]** section, update the **customer** and **country** values with appropriate information for your environment. Use two-letter country codes for the country value.</br>
    
    ```shell
    customer  = "MyCompanyName"
    country = "US"
    ```
        
8.  Save and close the file. Press the **Esc** key, type **:x** and hit **Enter**.
        
9.  Start/Restart the service:
    
    ```shell
    sudo systemctl restart telegraf
    ```
    
10.  Confirm the service is running. Look for **Active: active (running)**:

        ```shell
        systemctl status telegraf
        ```

Install & Configure Grafana
---------------------------

### Install Grafana

1.  Use wget to download Grafana:
    
    ```shell
    wget wget https://dl.grafana.com/oss/release/grafana-8.4.4-1.x86_64.rpm
    ```
    
2.  Install Grafana:
    
    ```shell
    sudo yum install grafana-8.4.4-1.x86_64.rpm
    ```
    
3.  Start the Grafana service:
    
    ```shell
    sudo service grafana-server start
    ```
    
4.  Configure the firewall for Grafana (only required if the firewall is enabled):
    
    ```shell
    firewall-cmd --permanent --zone=public --add-port=3000/tcp
    firewall-cmd --reload
    ```
    
5.  Install the following Grafana plugin:
    
    ```shell
    sudo grafana-cli plugins install grafana-clock-panel
    ```
    
6.  Restart the Grafana service:
    
    ```shell
    sudo service grafana-server restart
    ```
    
7.  Set Grafana to run on startup:
    
    ```shell
    sudo systemctl enable grafana-server
    ```
    

### Configure Grafana Data Source

1.  Use a web browser to connect to Grafana (http://fqdn:3000).
    
2.  The default username is “admin”, and the default password is “admin”. Grafana will prompt you to change the default password after logging in.
    
3.  Under the Configuration (gear) icon, add InfluxDB as a data source using the details from your install and test the connection. For the default single node installation, use the following information:

    URL: http://localhost:8086 <br/>
    Database: nasuni
    
![Add Data Source](/images/AddGrafanaDataSource.png)

4.  Click "Save & Test" to complete adding the InfluxDB data source.

### Configure Grafana Dashboard

1.  Download the [PerformanceDashboard.json](PerformanceDashboard.json) dashboard template from Nasuni Labs

2.  From the Grafana left navigation, click the plus icon, then click the **Import** link. On the Import page, click **Import** and upload the JSON file downloaded in step 1.

3.  Navigate the newly-imported dashboard to view the performance information for your Edge Appliances. The dashboard should display the following information:


![Performance Dashboard Screenshot](/images/PerformanceDashboardScreenshot.png)



Maintenance Tasks
=================

## Adding Edge Appliances

It is easy to add performance monitoring for newly-deployed Edge Appliances.

1.  Add the Nasuni Edge Appliance IP or FQDN to Telegraf:
    
    ```shell
    sudo vi /etc/telegraf/telegraf.conf
    ```
    
2.  In the **[inputs.snmp]** section, update the **agents** value to include the additional Edge Appliances monitor (note that the last entry does not need a trailing comma)

3.  Start/Restart the Telegraf service:
    
    ```shell
    systemctl restart telegraf
    ```

## Removing Stale Edge Appliances

It is good to clean up the InfluxDB database when decommissioning a Nasuni Edge Appliance to save disk space and remove stale data from Grafana.

1.  Remove Nasuni Edge Appliance IP or FQDN from Telegraf:
    
    ```shell
    sudo vi /etc/telegraf/telegraf.conf
    ```
    
2.  Start/Restart the service:
    
    ```shell
    systemctl restart telegraf
    ```
    
3.  Confirm the service is running. Look for “Active: active (running)”:
    
    ```shell
    systemctl status telegraf
    ```
    
4.  Open the InfluxDB shell:
    
    ```shell
    influx
    ```
    
5.  Identify the correct database for Nasuni:
    
    ```shell
    show databases
    ```
    
6.  Access the database:
    
    ```shell
    use <database_name>
    ```
    
7.  Display
    
    ```shell
    show series
    ```
    
8.  Identify the agent\_host to delete.
    
9.  Delete all series for the agent\_host:
    
    ```shell
    drop series where "agent_host" = <IP or FQDN>
    ```
    
10. Verify if the command was successful:
    
    ```shell
    show series
    ```

11. Exit
    
    ```shell
    exit
    ```
    
    
    

Support Statement
=================

*   This package stack has only been validated with the component versions documented in the README file.
    
*   Nasuni Support is limited to Nasuni APIs and Protocols (SMB, NFS, SNMP).
    
*   Nasuni API and Protocol bugs or feature requests should be communicated to Nasuni Customer Success.
    
*   GitHub project to-do's, bugs, and feature requests should be submitted as “Issues” in GitHub under its repositories.
    

