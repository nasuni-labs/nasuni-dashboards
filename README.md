# About

Nasuni Dashboards provides a framework for viewing time series data across a customer's entire Nasuni stack. Supported data sources currently include Edge Appliance SNMP data as well as the Global File Acceleration (GFA) Telemetry REST API.

# Components

*   Telegraf is an agent for collecting, processing, aggregating, and writing metrics.
    
*   InfluxDB is an open-source time series database for saving data.
    
*   Grafana is a multi-platform open-source analytics and interactive visualization web application for displaying data.
    

![TIGStackComponents](/images/TIGStackComponents.png)

# Installation Guidelines

## Validated Component Versions

Nasuni Dashboards has been validated with these component versions: 

* Telegraf: 1.20.4

* InfluxDB: 1.8.10
    
* Grafana: 8.4.4

## VM Requirements for Single-Node Installation 

* OS: Rocky Linux 8.x minimal or cloud marketplace offering
    
* Sizing: 4 vCPU, 16GB RAM, and 100GB boot disk

## Firewall Requirements

The data sources used with Nasuni Dashboards should be accessible from the VM:

* SNMP: UDP 161 access to monitored Edge Appliances

* GFA Telemetry REST API: TCP 443 access to https://gfa-telemetry-us1.api.nasuni.com

## Sizing Considerations

### InfluxDB and Telegraf

InfluxDB is designed to run on solid-state drives (SSDs) and memory-optimized instances. The Telegraf agent collects data for InfluxDB and does not require independent sizing.

<table data-layout="default" data-local-id="8e403bb5-545e-4876-8e57-082bc184df20" class="confluenceTable"><colgroup><col style="width: 113.33px;"><col style="width: 113.33px;"><col style="width: 113.33px;"><col style="width: 113.33px;"><col style="width: 113.33px;"><col style="width: 113.33px;"></colgroup><tbody><tr><th class="confluenceTh"><p style="text-align: center;"><strong>vCPU or CPU</strong></p><p style="text-align: center;">(Cores)</p></th><th class="confluenceTh"><p style="text-align: center;"><strong>RAM</strong></p><p style="text-align: center;">(GB)</p></th><th class="confluenceTh"><p style="text-align: center;"><strong>IOPS</strong></p></th><th class="confluenceTh"><p style="text-align: center;"><strong>Writes per second</strong></p></th><th class="confluenceTh"><p style="text-align: center;"><strong>Queries per second</strong></p></th><th class="confluenceTh"><p style="text-align: center;"><strong>Unique</strong></p><p style="text-align: center;"><strong>series</strong></p></th></tr><tr><td class="confluenceTd"><p style="text-align: center;">2-4</p></td><td class="confluenceTd"><p style="text-align: center;">2-4</p></td><td class="confluenceTd"><p style="text-align: center;">500</p></td><td class="confluenceTd"><p style="text-align: center;">&lt; 5,000</p></td><td class="confluenceTd"><p style="text-align: center;">&lt; 5</p></td><td class="confluenceTd"><p style="text-align: center;">&lt; 100,000</p></td></tr><tr><td class="confluenceTd"><p style="text-align: center;">4-6</p></td><td class="confluenceTd"><p style="text-align: center;">8-32</p></td><td class="confluenceTd"><p style="text-align: center;">500-1000</p></td><td class="confluenceTd"><p style="text-align: center;">&lt; 250,000</p></td><td class="confluenceTd"><p style="text-align: center;">&lt; 25</p></td><td class="confluenceTd"><p style="text-align: center;">&lt; 1,000,000</p></td></tr><tr><td class="confluenceTd"><p style="text-align: center;">8+</p></td><td class="confluenceTd"><p style="text-align: center;">32+</p></td><td class="confluenceTd"><p style="text-align: center;">1000+</p></td><td class="confluenceTd"><p style="text-align: center;">&gt; 250,000</p></td><td class="confluenceTd"><p style="text-align: center;">&gt; 25</p></td><td class="confluenceTd"><p style="text-align: center;">&gt; 1,000,000</p></td></tr></tbody></table>

### Grafana

*   Grafana is very lightweight in its use of memory and CPU.
    
*   Minimum recommended memory: 255 MB
    
*   Minimum recommended CPU: 1
    

# Configure Nasuni Data Sources

## SNMP for Edge Appliances
    
1.  In the NMC, click **Filers** in the top bar.
    
2.  Click **SNMP Settings** on the left navigation menu under **Filer Settings**.
    
3.  For each Edge Appliance that should be monitored, select the Edge Appliances. (Note: Only select multiple Edge Appliances if all values are shared.)
    
4.  Click the **Edit Filers** button in the upper right.
    
5.  Enable your desired SNMP protocol versions:

    a. SNMPv3 (most secure): Set **Enable v3 Support** to **On**. Enter a **Username** and **Password**.

    b. SNMPv1/v2: Set **Enable v1,v2c Support**  to **On**. Set the **Community Name** to **public** (or to what you plan to use).
    
6.  Enter your preferred **System Location** and **System Contact**.
    
7.  Click **Save SNMP Settings**.

## GFA Telemetry API (Optional)
To use the GFA Telemetry API:

*   You must have a volume under GFA management (Active or Observation mode)

*   You must create a [Global File Accelerator API key on the Nasuni dashboard](https://account.nasuni.com/account/gfa/). Note: If the link is not accessible, you may not be licensed for GFA. Contact your Nasuni Account Manager for assistance.
    
# Install and Configure Nasuni Dashboards

## Download Nasuni Dashboards Zip
Click the green **Code** button within the Nasuni Labs nasuni-dashboards repository and select the **Download ZIP** option to download a local copy of the repository and extract the contents to your computer.

## Deploy the VM
Deploy a Rocky Linux VM that meets the requirements for Nasuni Dashboards. 

## Configure DNS
To use hostnames rather than IP addresses for the Telegraf SNMP agent configuration, DNS is required. 
Two common scenarios for DNS configuration:

*   DHCP: DHCP is typically used for Cloud VMs, but can also be used on-premises. If the DNS server provided by DHCP is not authoritative for Edge Appliance hostnames, you can override the DNS server settings.

    *   To enable static DNS with DHCP, ssh to the VM and run the following command to disable DHCP updates to DNS:

        ```shell
        sudo tee -a /etc/NetworkManager/conf.d/disable-resolve.conf-managing.conf > /dev/null <<EOT
        [main]
        dns=none
        EOT
        ```

    *   Edit **/etc/resolv.conf**, specifying nameserver entries for the DNS servers that are authoritative for Edge Appliance DNS.
    
        ```shell
        sudo vi /etc/resolv.conf
        ```

*   Static IP: Ensure that the DNS servers you specify are authoritative for Edge Appliance DNS.


## Install and Configure InfluxDB

### Install InfluxDB

To install InfluxDB, ssh to the VM and run the following commands:

1.  Update all packages (*-y* argument skips confirmation; for manual confirmation, remove it):
    
    ```shell
    sudo yum -y update
    ```
    
2.  Install wget (if not present):
    
    ```shell
    sudo yum -y install wget
    ```
    
3.  Switch to your home directory before downloading packages:

    ```shell
    cd ~
    ```    
    
4.  Use wget to download InfluxDB:
    
    ```shell
    wget https://dl.influxdata.com/influxdb/releases/influxdb-1.8.10.x86_64.rpm
    ```
    
5.  Install InfluxDB:
    
    ```shell
    sudo yum -y localinstall influxdb-1.8.10.x86_64.rpm
    ```
    
6.  Start and enable the InfluxDB service:
    
    ```shell
    sudo systemctl start influxdb && sudo systemctl enable influxdb
    ```
    
7.  Add firewall rules if the firewall service is running. First, check if the firewall service is running:

    ```shell
    systemctl status firewalld --no-pager
    ```
    
    If the firewall service is running, the output will be similar to the following:
    
    `firewalld.service - firewalld - dynamic firewall daemon`</br>
    `Loaded: loaded (/usr/lib/systemd/system/firewalld.service; enabled; vendor pr`</br>
    `Active: active (running) since Mon 2021-12-18 16:05:15 CET; 50min ago`</br>
    `Docs: man:firewalld(1)`

    If the firewall is not running, the output will be similar to the following:
    
    `Unit firewalld.service could not be found.`
    
    If the firewall service is running, configure the firewall for InfluxDB (only required if the firewall was running), then reload the firewall configuration:
    
    ```shell
    sudo firewall-cmd --add-port=8086/tcp --permanent
    sudo firewall-cmd --reload
    ```
    
### Create the InfluxDB database:
    
1.  Open the InfluxDB shell:

    ```shell
    influx
    ```

2.  Create the database for use by Grafana:

    ```shell
    create database nasuni
    ```

3.  Confirm that the database was created:

    ```shell
    show databases
    ```

4.  Confirm that **nasuni** is listed.

5.  Exit the InfluxDB shell:

    ```shell
    exit
    ```
        

## Install and Configure Telegraf

### Install Telegraf

1.  Use wget to download Telegraf:
    
    ```shell
    wget https://dl.influxdata.com/telegraf/releases/telegraf-1.20.4-1.x86_64.rpm
    ```
    
2.  Install Telegraf:
    
    ```shell
    sudo yum -y localinstall telegraf-1.20.4-1.x86_64.rpm
    ```
    
3.  Install SNMP dependencies for Telegraf:
    
    ```shell
    sudo yum -y install net-snmp net-snmp-utils
    ```
  

### Telegraf Configuration

1.  Back up the default Telegraf configuration file:
    ```shell
    sudo mv /etc/telegraf/telegraf.conf /etc/telegraf/telegraf.conf.bak
    ```

2.  On your computer, open the **telegraf.conf** file from the extracted repository zip archive in a text editor, select all, and copy the contents to your clipboard.

3.  Return to the VM ssh session, open vi, and paste the contents from step 2, customizing the values for the sections below:
    
    ```shell
    sudo vi /etc/telegraf/telegraf.conf
    ```
    
4.  If you installed influxDB on a dedicated host (single-node installs can skip this step), go the **[outputs.influxdb]** section, update the **urls** value with the InfluxDB IP address, and confirm the port. You can use the search function in vi to jump to the appropriate section of the file. Type **/** followed by the string you want to search for, and then press **Return**. The following example assumes the default port and InfluxDB running on the same host as Telegraf:</br>
    - `urls = ["http://127.0.0.1:8086"]`
        
5.  In the **[inputs.snmp]** section, update the **agents** value with the FQDN of all Edge Appliances to be monitored. For example (Note: The last entry does not require a trailing comma):</br>
    
    ```shell
       agents = [
            "edge1.yourdomain.com",
            "edge2.yourdomain.com",
            "edge3.yourdomain.com"    
        ]
    ```
            
6.  In the **[inputs.snmp]** section, update protocol-specific values based on the protocol configured for your Edge Appliances and NMC: 
    - SNMPv3

        - `sec_name = "<username>"` (match the **Username** supplied in the Nasuni SNMP UI)

        - `auth_password = "<password>"` (match the **Password** supplied in the Nasuni SNMP UI)
    
    - SNMPv2: The included telegraf.conf assumes SNMPv3, so some values must be changed from the defaults and commented/uncommented (comment lines begin with **#**) to use SNMPv2:

        - `version = 2`

        - `community = "public"` (uncomment this line and set the value if not using the **public** default)

        - comment out (add **#** to the beginning) lines SNMPv3-related lines that start with the following:

            - `sec_name`

            - `auth_protocol`

            - `auth_password`

            - `sec_level`

7.  In the **[inputs.snmp.tags]** section, update the **customer** and **country** values with appropriate information for your environment. Use two-letter country codes for the country value:</br>

    - `customer  = "<CustomerName>"`

    - `country = "<CountryCode>"`
        
8.  To use the optional Global File Acceleration (GFA) Telemetry API data source, make the following changes to the **[inputs.http]** section (located directly above the **[inputs.snmp]** section):

    - `urls (uncomment this line)`

    - `headers (uncomment the first headers line and enter the <GFA API Key> from the NOC Dashboard)`

    - `headers (uncomment the second headers line and enter a string/name to use as a unique identifier for this connection)`

9.  Save and close the file. Press **Esc**, type **:x**, and press **Enter**.

10.  Set the ownership and permissions for telegraf.conf:

        ```shell
        sudo chown root:root /etc/telegraf/telegraf.conf
        sudo chmod 644 /etc/telegraf/telegraf.conf
        ```
        
11.  Start/Restart the Telegraf service:
    
        ```shell
        sudo systemctl restart telegraf
        ```
    
12.  Confirm the Telegraf service is running. Look for **Active: active (running)**:

        ```shell
        systemctl status telegraf --no-pager
        ```

## Install and Configure Grafana

### Install Grafana

1.  Switch to your home directory before downloading Grafana:
    
    ```shell
    cd ~
    ```

2.  Use wget to download Grafana:
    
    ```shell
    wget https://dl.grafana.com/oss/release/grafana-8.4.4-1.x86_64.rpm
    ```
    
3.  Install Grafana:
    
    ```shell
    sudo yum -y install grafana-8.4.4-1.x86_64.rpm
    ```
    
4.  Install two Grafana plugins:

    ```shell
    sudo grafana-cli plugins install grafana-clock-panel
    sudo grafana-cli plugins install marcusolsson-gantt-panel
    ```
    
5.  Start the Grafana service:
    
    ```shell
    sudo systemctl start grafana-server
    ```
    
6.  Set Grafana to run on startup:

    ```shell
    sudo systemctl enable grafana-server
    ```
    
7.  Confirm the Grafana service is running. Look for **Active: active (running)**:

    ```shell
    systemctl status grafana-server --no-pager
    ```
    
8.  Configure the firewall for Grafana (only required if the firewall is enabled):
    
    ```shell
    sudo firewall-cmd --permanent --zone=public --add-port=3000/tcp
    sudo firewall-cmd --reload
    ```
    

### Configure Grafana Data Source

1.  Use a web browser to connect to Grafana (http://fqdn-of-vm:3000).
    
2.  Log in to Grafana. The default username is **admin**, and the default password is **admin**. Grafana will prompt you to change the default password after logging in.
    
3.  Under the Configuration (gear) icon, add InfluxDB as a data source using the details from your install, and test the connection. For the default single-node installation, use the following information:

    - URL: http://localhost:8086

    - Database: nasuni
    
![Add Data Source](/images/AddGrafanaDataSource.png)

4.  Click **Save & test** to complete adding the InfluxDB data source. A green checkbox should appear alongside a message that the **Data source is working**.

### Configure Grafana Dashboards

1.  From the Grafana left navigation bar, click the plus icon, then click the **Import** link. On the Import page, click **Import** and upload the **PerformanceDashboard.json** file from the extracted repository zip archive.

2.  The **Options** screen opens. Select the **InfluxDB** data source name, which should be **InfluxDB (default)**, and click **Import**.

3.  Navigate to the newly-imported dashboard to view the performance information for your Edge Appliances. Here is an example of expected output for the dashboard:


![Performance Dashboard Screenshot](/images/PerformanceDashboardScreenshot.png)

4. If using the GFA Telemetry API data source, import the associated GFA dashboards. From the Grafana left navigation bar, click the plus icon, then click the **Import** link. On the Import page, click **Import** and upload the **GFAVolumes.json** file from the extracted repository zip archive accepting the defaults and saving the configuration. Repeat for the **GFAAppliances.json** and **GFAMiscellaneous.json** files.

5. Navigate to the newly-imported GFA Volumes dashboard. Here is an example of expected output for the dashboard:

![GFA Volumes Dashboard Screenshot](/images/GFAVolumesDashboardScreenshot.png)

6. If you are using the GFA Miscellaneous dashboard you will need to install the Gantt Chart plugin. Follow the instructions https://grafana.com/grafana/plugins/marcusolsson-gantt-panel/?tab=installation to download and install the plugin.

# Troubleshooting

## Data Unvailable in Performance Dashboard
Telegraf depends on SNMP data from the Edge Appliances to work. You can confirm that SNMP is configured and working on the Edge Appliance by connecting to the server where you installed the TIG stack over SSH and using the snmpwalk utility to load SNMP data. 

### Snmpwalk using SNMPv3

```shell
snmpwalk -v3 -l authnoPriv -u <snmpUsername> -a SHA -A '<snmpPassword>' -On <edgeApplianceHostnameOrIP> .1.3.6.1.4.1.42040
```
     
You will need to populate snmpUsername, snmpPassword (enclosed in single quotes to handle special characters), and edgeApplianceHostnameOrIP with the relevant values for your environment. Here's an example of the populated command:  

```shell
snmpwalk -v3 -l authnoPriv -u admin -a SHA -A 'myPassword123' -On edge1.domain.com .1.3.6.1.4.1.42040
```
     
The OID at the end corresponds to the root OID of the Nasuni MIB and will list all values in it.

### Snmpwalk using SNMPv2
```shell
snmpwalk -v 2c -c <communityName> -On  <edgeApplianceHostnameOrIP> .1.3.6.1.4.1.42040
```

You will need to populate communityName and edgeApplianceHostnameOrIP with the relevant values for your environment. Here's an example of the populated command:  

```shell
snmpwalk -v 2c -c public -On edge1.domain.com .1.3.6.1.4.1.42040
```

## Telegraf

### Issues Starting
If Telegraf reports errors in **/var/log/telegraf/telegraf.log**, the source of the problem is most likely with the contents of telegraf.conf. 
* Run the following command to test telegraf.conf:
     ```shell
     telegraf --config /etc/telegraf/telegraf.conf --test
     ```
### JSON_V2 Errors
If Telegraf reports an error parsing JSON_V2 (used for the GFA Telemetry data source), your version of Telegraf is too old. Telegraf added JSON_V2 parsing in version 1.19. Update to the validated version of Telegraf for Nasuni Dashboards to correct this issue.


# Maintenance Tasks

## Adding Edge Appliances

It is easy to add performance monitoring for newly-deployed Edge Appliances.

1.  To add the Nasuni Edge Appliance IP or FQDN to Telegraf, edit the telegraf.conf file:
    
    ```shell
    sudo vi /etc/telegraf/telegraf.conf
    ```
    
2.  In the **[inputs.snmp]** section, update the **agents** value to include the additional Edge Appliance monitor. (Note that the last entry does not need a trailing comma.)

3.  Start/Restart the Telegraf service:
    
    ```shell
    systemctl restart telegraf
    ```

## Removing Stale Edge Appliances

When decommissioning a Nasuni Edge Appliance, it is good to clean up the InfluxDB database to save disk space and remove stale data from Grafana.

1.  To remove the Nasuni Edge Appliance IP or FQDN from Telegraf, edit the telegraf.conf file:
    
    ```shell
    sudo vi /etc/telegraf/telegraf.conf
    ```
    
2.  In the **[inputs.snmp]** section, update the **agents** value to remove the unwanted Edge Appliance monitor. (Note that the last entry does not need a trailing comma.)
    
3.  Start/Restart the service:
    
    ```shell
    systemctl restart telegraf
    ```
    
4.  Confirm that the service is running. Look for “Active: active (running)”:
    
    ```shell
    systemctl status telegraf --no-pager
    ```
    
5.  Open the InfluxDB shell:
    
    ```shell
    influx
    ```
    
6.  Identify the correct database for Nasuni:
    
    ```shell
    show databases
    ```
    
7.  Access the database:
    
    ```shell
    use <database_name>
    ```
    The database_name is "nasuni", which was created in the "Creating the InfluxDB database" above.
    
8.  Display the series (a logical grouping of data defined by shared measurement, tag set, and field key):
    
    ```shell
    show series
    ```
    
9.  Identify the agent\_host to delete.
    
10.  Delete all series for the agent\_host:

        ```shell
        drop series where "agent_host" = <IP or FQDN>
        ```

11. Verify if the command was successful:
    
    ```shell
    show series
    ```

12. Exit:
    
    ```shell
    exit
    ```
    
    
    

# Support Statement

*   This package stack has only been validated with the component versions documented in the README file.
    
*   Nasuni Support is limited to Nasuni APIs and Protocols (SMB, NFS, SNMP, GFA Telemetry).
    
*   Nasuni API and Protocol bugs or feature requests should be communicated to Nasuni Customer Success.
    
*   GitHub project to-do's, bugs, and feature requests should be submitted as “Issues” in GitHub under its repositories.
    

