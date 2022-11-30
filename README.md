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

* Telegraf: 1.23.4

* InfluxDB: 1.8.10
    
* Grafana: 9.1.0

## VM Requirements for Single-Node Installation 

* OSes: 

  - Rocky Linux 8.6 or 9.0 minimal, sourced from cloud marketplace offerings or Rocky Linux ISO

  - Windows Server 2019 64-bit (x86)

* Sizing: 4 vCPU, 16GB RAM, and 100GB boot disk

## Firewall Requirements

The data sources used with Nasuni Dashboards should be accessible from the VM:

* Outbound:

  - SNMP: UDP 161 access to monitored Edge Appliances

  - GFA Telemetry REST API: TCP 443 access to https://gfa-telemetry-1.api.nasuni.com

* Inbound:

  - Grafana: TCP Port 3000 for access to the default Grafana UI

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

## Deploy the VM
Deploy a Rocky Linux or Windows VM that meets the requirements for Nasuni Dashboards. 

NOTE: Be sure to update all the OS packages by running 

        sudo dnf update

## Configure DNS
To use hostnames rather than IP addresses for the Telegraf SNMP agent configuration, DNS is required. 
Two common scenarios for DNS configuration:

*   DHCP: DHCP is typically used for Cloud VMs, but can also be used on-premises. If the DNS server provided by DHCP is not authoritative for Edge Appliance hostnames, you can override the DNS server settings.

    *   To enable static DNS with DHCP for Rocky Linux, ssh to the VM and run the following command to disable DHCP updates to DNS:

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

## Download Nasuni Dashboards Zip
From your computer (Rocky Linux) or the Windows VM (Windows Install), click the green **Code** button within the Nasuni Labs nasuni-dashboards repository and select the **Download ZIP** option to download a local copy of the repository and extract the contents.

## Install and Configure InfluxDB

### Rocky Linux InfluxDB Installation Instructions
<details>
    <summary>Expand Rocky Linux Installation Instructions</summary>
<br/>

To install InfluxDB on Rocky Linux, ssh to the VM and run the following commands:

1.  Update all packages (*-y* argument skips confirmation; for manual confirmation, remove it):
    
    ```shell
    sudo yum -y update
    ```
    
2.  Install wget (if not present):
    
    ```shell
    sudo yum -y install wget
    ```
    
3.  Switch to your home directory and use wget to download InfluxDB :

    ```shell
    cd ~ && wget https://dl.influxdata.com/influxdb/releases/influxdb-1.8.10.x86_64.rpm
    ```    
    
4.  Install InfluxDB:
    
    ```shell
    sudo yum -y localinstall influxdb-1.8.10.x86_64.rpm
    ```
    
5.  Start and enable the InfluxDB service:
    
    ```shell
    sudo systemctl start influxdb && sudo systemctl enable influxdb
    ```
    
6.  Add firewall rules if the firewall service is running. First, check if the firewall service is running:

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
    sudo firewall-cmd --add-port=8086/tcp --permanent && sudo firewall-cmd --reload
    ```
</details>
    
### Windows InfluxDB Installation Instructions
<details>
    <summary>Expand Windows Installation Instructions</summary>
    
   <br/>

To install InfluxD on Windows, connect to the Windows VM and follow these instructions:
    
1.  Run PowerShell as an administrator, then enter the following commands to download and install InfluxDB:

    ```PowerShell
    cd ~\Downloads ; Invoke-WebRequest https://dl.influxdata.com/influxdb/releases/influxdb-1.8.10_windows_amd64.zip -UseBasicParsing -OutFile influxdb-1.8.10_windows_amd64.zip ;Expand-Archive .\influxdb-1.8.10_windows_amd64.zip -DestinationPath .\ ;mkdir 'c:\Program Files\InfluxData\InfluxDB' ;mv .\influxdb-1.8.10-1\* 'c:\Program Files\InfluxData\InfluxDB\'
    ```    
    
2. Download and configure the NSSM service manager to make InfluxDB run as a Windows Service (The final output should show "Status: Running", "Name: InfluxDB"):
   
   ```PowerShell
   cd ~\Downloads ;Invoke-WebRequest https://nssm.cc/release/nssm-2.24.zip -UseBasicParsing -OutFile nssm-2.24.zip ;Expand-Archive .\nssm-2.24.zip -DestinationPath .\ ;mkdir 'c:\Program Files\NSSM' ;mv .\nssm-2.24\win64\nssm.exe 'c:\Program Files\NSSM\' ;cd 'c:\Program Files\NSSM\' ;.\nssm.exe install InfluxDB "C:\Program Files\InfluxData\InfluxDB\influxd.exe" ;.\nssm.exe set InfluxDB Description "InfluxDB Time-Series Database" ;Start-Service InfluxDB ; Get-Service InfluxDB
   ```
</details>    
    
### Create the InfluxDB database:
    
1.  Open the InfluxDB shell:

    * Rocky Linux - ssh to the VM and run the following commands:

        ```shell
        influx
        ```

    * Windows - Use PowerShell to open the InfluxDB shell:

        ```PowerShell
        Start-Process "C:\Program Files\InfluxData\InfluxDB\influx.exe"
        ```

2.  Create the database for use by Grafana using the InfluxDB (Rocky Linux and Windows):

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

### Rocky Linux Telegraf Installation Instructions
<details>
    <summary>Expand Rocky Linux Installation Instructions</summary>
<br/>

1.  Use wget to download Telegraf:
    
    ```shell
    wget https://dl.influxdata.com/telegraf/releases/telegraf-1.23.4-1.x86_64.rpm
    ```
    
2.  Install Telegraf and SNMP dependencies:
    
    ```shell
    sudo yum -y localinstall telegraf-1.23.4-1.x86_64.rpm && sudo yum -y install net-snmp net-snmp-utils
    ```
    
3.  Backup the default Telegraf configuration file:

    ```shell
    sudo mv /etc/telegraf/telegraf.conf /etc/telegraf/telegraf.conf.bak
    ```
    
</details>
  
### Windows Telegraf Installation Instructions
<details>
    <summary>Expand Windows Installation Instructions</summary>
    
1.  Run PowerShell as an administrator, then run commands to download and install Telegraf:
    
    ```PowerShell
    cd ~\Downloads ;Invoke-WebRequest https://dl.influxdata.com/telegraf/releases/telegraf-1.23.4_windows_amd64.zip -UseBasicParsing -OutFile telegraf-1.23.4_windows_amd64.zip ;Expand-Archive .\telegraf-1.23.4_windows_amd64.zip -DestinationPath .\ ;mkdir 'c:\Program Files\Telegraf' ;mv .\telegraf-1.23.4\* 'C:\Program Files\Telegraf\' ;mv 'C:\Program Files\Telegraf\telegraf.conf' 'C:\Program Files\Telegraf\telegraf.conf.bak'
    ```    
    
</details>
    
### Telegraf Configuration

1.  Edit the telegraf.conf file:

    - Rocky Linux: On your computer, open the **telegraf.conf** file from the extracted Nasuni Dashboards repository zip archive in a text editor, select all, and copy       the contents to your clipboard. Return to the VM ssh session, open vi, and paste the contents from step 2, customizing the values for the             sections below:

       ```shell
       sudo vi /etc/telegraf/telegraf.conf
       ```
       
    - Windows: In the Windows VM, copy the contents of the **telegraf.conf** file from the extracted Nasuni Dashboards repository zip archive to "C:\Program Files\Telegraf\telegraf.conf" and make the edits outlined in the next set of steps. One way to do this is to run Notepad or Notepad++ as an administrator, and paste the contents of telegraf.conf. Example to open telegraf.conf for editing in Notepad using PowerShell (running PowerShell as administrator):

        ```PowerShell
        notepad.exe "C:\Program Files\Telegraf\telegraf.conf"
        ```   
    
2.  If you installed influxDB on a dedicated host (single-node installs can skip this step), go the **[outputs.influxdb]** section, update the **urls** value with the InfluxDB IP address, and confirm the port. You can use the search function in vi to jump to the appropriate section of the file. Type **/** followed by the string you want to search for, and then press **Return**. The following example assumes the default port and InfluxDB running on the same host as Telegraf:</br>
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

9.  Save and close the file. For Rocky Linux editors using vi, press **Esc**, type **:x**, and press **Enter**.

10. Make final changes to the telegraf.conf file permissions (Linux) and start Telegraf:

     * Rocky Linux
     
        * Set the ownership and permissions for telegraf.conf:

            ```shell
            sudo chown root:root /etc/telegraf/telegraf.conf && sudo chmod 644 /etc/telegraf/telegraf.conf
            ```
        
        * Restart the Telegraf service and confirm the Telegraf service is running. Look for **Active: active (running)** :

            ```shell
            sudo systemctl restart telegraf && systemctl status telegraf --no-pager
            ```
            
      * Windows
       
         *  Run PowerShell as an administrator to install Telegraf as a Windows service and start it (The final output should show "Status: Running","Name: Telegraf"):

         ```Powershell
            cd "C:\Program Files\Telegraf" ;.\telegraf.exe --service install --config "C:\Program Files\Telegraf\telegraf.conf" ;Start-Service Telegraf ; Get-Service Telegraf
         ```

## Install and Configure Grafana

### Rocky Linux Grafana Installation Instructions
<details>
    <summary>Expand Rocky Linux Installation Instructions</summary>


1.  Switch to your home directory and use wget to download Grafana:
    
    ```shell
    cd ~ && wget https://dl.grafana.com/oss/release/grafana-9.1.0-1.x86_64.rpm
    ```
    
2.  Install Grafana and two plugins:
    
    ```shell
    sudo yum -y install grafana-9.1.0-1.x86_64.rpm && sudo grafana-cli plugins install grafana-clock-panel && sudo grafana-cli plugins install marcusolsson-gantt-panel
    ```
    
3.  Start and enable the Grafana service:
    
    ```shell
    sudo systemctl start grafana-server && sudo systemctl enable grafana-server
    ```
    
4.  Confirm the Grafana service is running. Look for **Active: active (running)**:

    ```shell
    systemctl status grafana-server --no-pager
    ```
    
5.  Configure the firewall for Grafana (only required if the firewall is enabled):
    
    ```shell
    sudo firewall-cmd --permanent --zone=public --add-port=3000/tcp && sudo firewall-cmd --reload
    ```
 </details>
 
 ### Windows Grafana Installation Instructions
<details>
    <summary>Expand Windows Installation Instructions</summary>
    
1.  Run PowerShell as an administrator, and run commands to download and install Grafana. Accept all the defaults (Grafana will automatically be installed as a Windows service and started) :
    
    ```PowerShell
    cd ~\Downloads ;Invoke-WebRequest https://dl.grafana.com/oss/release/grafana-9.1.0.windows-amd64.msi -UseBasicParsing -OutFile grafana-9.1.0.windows-amd64.msi ;.\grafana-9.1.0.windows-amd64.msi
    ```
    
2.  If Windows firewall is active, open TCP port 3000 for inbound access so remote computers can access the Grafana Dashboard:

    ```PowerShell
    New-NetFirewallRule -DisplayName “Allow Grafana Port 3000” -Direction Inbound -LocalPort 3000 -Protocol TCP -Action Allow
    ```
    
</details>

### Configure Grafana Data Source

1.  Use a web browser to connect to Grafana (http://fqdn-of-vm:3000).
    
2.  Log in to Grafana. The default username is **admin**, and the default password is **admin**. Grafana will prompt you to change the default password after logging in.
    
3.  Under the Configuration (gear) icon, add InfluxDB as a data source using the details from your install, and test the connection. For the default single-node installation, use the following information:

    - URL: http://localhost:8086

    - Database: nasuni
    
![Add Data Source](/images/AddGrafanaDataSource.png)

4.  Click **Save & test** to complete adding the InfluxDB data source. A green checkbox should appear alongside a message that the **Data source is working**.

### Configure Grafana Dashboards

1.  From the Grafana left navigation bar, click the Dashboards (four squares) icon, then click the **Import** link. On the Import page, click **Import** and upload the **PerformanceDashboard.json** file from the extracted repository zip archive.

![TIGStackComponents](/images/GrafanaImport.png)

2.  The **Options** screen opens. Select the **InfluxDB** data source name, which should be **InfluxDB (default)**, and click **Import**.

3.  Navigate to the newly-imported dashboard to view the performance information for your Edge Appliances. Here is an example of expected output for the dashboard:


![Performance Dashboard Screenshot](/images/PerformanceDashboardScreenshot.png)

4. If using the GFA Telemetry API data source, import the associated GFA dashboards. From the Grafana left navigation bar, click the Dashboards (four squares) icon, then click the **Import** link. On the Import page, click **Import** and upload the **GFAVolumes.json** file from the extracted repository zip archive accepting the defaults and saving the configuration. Repeat for the **GFAAppliances.json** and **GFAMiscellaneous.json** files.

5. Navigate to the newly-imported GFA Volumes dashboard. Here is an example of expected output for the dashboard:

![GFA Volumes Dashboard Screenshot](/images/GFAVolumesDashboardScreenshot.png)

6. If you are using the GFA Miscellaneous dashboard, you will need to install the Gantt Chart plugin. Follow the instructions https://grafana.com/grafana/plugins/marcusolsson-gantt-panel/?tab=installation to download and install the plugin.

# Troubleshooting

## Data Unavailable in Performance Dashboard
Telegraf depends on SNMP data from the Edge Appliances to work. You can confirm that SNMP is configured and working on the Edge Appliance by connecting to the server where you installed the TIG stack over SSH (Rocky Linux) or RDP (Windows) and using SNMP utilities to load SNMP data. 

## SNMP Troubleshooting

## Rocky Linux SNMP Troubleshooting
<details>
    <summary>Expand Rocky Linux Snmpwalk Instructions</summary>
<br/>
    
Snmpwalk can be used to test SNMP connectivity and inspect values returned for an SNMP Object Identifier (OID). Nasuni publishes a full list of [OIDs for the Nasuni Edge Appliance](http://b.link/Nasuni_SNMP_OIDs_for_NASUNI-FILER-MIB). Snmpwalk for Linux is automatically installed when installing Nasuni Dashboards, although the syntax to use it differs by SNMP version.
    
### Rocky Linux Snmpwalk using SNMPv3

```shell
snmpwalk -v3 -l authnoPriv -u <snmpUsername> -a SHA -A '<snmpPassword>' -On <edgeApplianceHostnameOrIP> .1.3.6.1.4.1.42040
```
     
Populate snmpUsername, snmpPassword (enclosed in single quotes to handle special characters), and edgeApplianceHostnameOrIP with the relevant values for your environment. Here's an example of the populated command:  

```shell
snmpwalk -v3 -l authnoPriv -u admin -a SHA -A 'myPassword123' -On edge1.domain.com .1.3.6.1.4.1.42040
```
     
The OID at the end corresponds to the root OID of the Nasuni MIB and will list all values in it.

### Rocky Linux Snmpwalk using SNMPv2
```shell
snmpwalk -v 2c -c <communityName> -On  <edgeApplianceHostnameOrIP> .1.3.6.1.4.1.42040
```

Populate communityName and edgeApplianceHostnameOrIP with the relevant values for your environment. Here's an example of the populated command:  

```shell
snmpwalk -v 2c -c public -On edge1.domain.com .1.3.6.1.4.1.42040
```
    
</details>
    
## Windows SNMP Troubleshooting
<details>
    <summary>Expand Windows SNMP Troubleshooting Info</summary>
<br/>

Windows does not include SNMP troubleshooting tools, although a number of third-party tools are available. [SnmpGet](https://ezfive.com/snmpsoft-tools/snmp-get/) is an easy to use command-line SNMP utility for Windows. It can be used to verify SNMP connectivity and inspect values returned for an SNMP Object Identifier (OID). Nasuni publishes a full list of [OIDs for the Nasuni Edge Appliance](http://b.link/Nasuni_SNMP_OIDs_for_NASUNI-FILER-MIB). After downloading SnmpGet, CD to the directory where you saved the file using PowerShell and run it by using the version specific SNMP commands below.
    
### Windows SNMP Troubleshooting using SNMPv3
    
```PowerShell
.\SnmpGet.exe -v:3 -sn:"<snmpUsername>" -ap:SHA -aw:"<snmpPassword>" -r:"<edgeApplianceHostnameOrIP>" -o:<OidToQuery>
```
    
Populate snmpUsername, snmpPassword, edgeApplianceHostnameOrIP, and OidToQuery with the relevant values for your environment. Here's an example of the populated command:
    
```PowerShell
.\SnmpGet.exe -v:3 -sn:"admin" -ap:SHA -aw:"myPassword123" -r:"edge1.domain.com" -o:.1.3.6.1.4.1.42040.1.1.0
```
    
### Windows SNMP Troubleshooting using SNMPv2   
    
```PowerShell
.\SnmpGet.exe -v:2 -c:"<communityName>" -r:"<edgeApplianceHostnameOrIP>" -o:<OidToQuery>
```

Populate communityName, edgeApplianceHostnameOrIP, and OidToQuery with the relevant values for your environment. Here's an example of the populated command:  

```Powershell
.\SnmpGet.exe -v:2 -c:"nasuni" -r:"edge1.domain.com" -o:1.3.6.1.4.1.42040.1.1.0    
```
    
</details>
    
## Telegraf

### Issues Starting
If Telegraf reports errors in **/var/log/telegraf/telegraf.log**, the source of the problem is most likely with the contents of telegraf.conf. Run the following command to test telegraf.conf (the command will report a verbose error if it encounters a problem):
    
* Rocky Linux
    
  ```shell
   telegraf --config /etc/telegraf/telegraf.conf --test
   ```
    
 * Windows
    
   Run the following command in PowerShell:

   ```PowerShell
   cd "c:\Program Files\telegraf" ; .\telegraf.exe --config "C:\Program Files\Telegraf\telegraf.conf" --test
   ```
    
### JSON_V2 Errors
If Telegraf reports an error parsing JSON_V2 (used for the GFA Telemetry data source), your version of Telegraf is too old. Telegraf added JSON_V2 parsing in version 1.19. Update to the validated version of Telegraf for Nasuni Dashboards to correct this issue.

# Maintenance Tasks

## Editing Telegraf.conf

To edit edit the telegraf.conf file:
    
* Rocky Linux
    
  ```shell
  sudo vi /etc/telegraf/telegraf.conf
  ```
    
 * Windows
    
   Run the following command in PowerShell to open telegraf.conf for editing in Notepad:
    
   ```PowerShell
   notepad.exe "C:\Program Files\Telegraf\telegraf.conf"
   ```
 
 ## Restart the Telegraf service
 Always restart the Telegraf service after editing telegraf.conf to load the new settings.
    
 * Rocky Linux
    
   ```shell
   systemctl restart telegraf
   ```
    
* Windows
    
  Run PowerShell as an administrator and run the following command:
    
  ```PowerShell
  Restart-Service Telegraf ; Get-Service Telegraf
  ```
   
## Adding Edge Appliances
It is easy to add performance monitoring for newly-deployed Edge Appliances.
    
1.  Edit telegraf.conf using the instructions above. In the **[inputs.snmp]** section, update the **agents** value to include the additional Edge Appliance monitor and save the changes. (Note that the last entry does not need a trailing comma.)
    
2.  Restart the Telegraf servcie using the instructions above.

## Removing Stale Edge Appliances

When decommissioning a Nasuni Edge Appliance, it is good to clean up the InfluxDB database to save disk space and remove stale data from Grafana.

1.  To remove the Nasuni Edge Appliance IP or FQDN from Telegraf, edit the telegraf.conf file.
    
2.  In the **[inputs.snmp]** section, update the **agents** value to remove the unwanted Edge Appliance monitor and save telegraf.conf. (Note that the last entry does not need a trailing comma.)
    
3.  Start/Restart the Telegraf service using the instructions above.
    
4.  Open the InfluxDB shell:
    
    * Rocky Linux
    
      ```shell
      influx
      ```
    
    * Windows
    
      Run PowerShell as an administrator and run the following command to open the InfluxDB shell:
    
      ```PowerShell
      Start-Process "C:\Program Files\InfluxData\InfluxDB\influx.exe"
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
        drop series where "agent_host" = '<IP or FQDN>'
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
    

