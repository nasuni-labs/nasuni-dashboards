# About

Nasuni Dashboards provide a framework for viewing time series data across a customer's entire Nasuni stack. Supported data sources include Edge Appliance SNMP data, Global File Acceleration (GFA) Telemetry REST API, and the Nasuni Access Anywhere (NAA) monitoring API.

# Components

*   Telegraf is an agent for collecting, processing, aggregating, and writing metrics.
    
*   InfluxDB is an open-source time series database for saving data.
    
*   Grafana is a multi-platform open-source analytics and interactive visualization web application for displaying data.
    

![TIGStackComponents](/images/TIGStackComponents.png)

# Support Statement

*   This package stack has only been validated with the component versions documented in the README file.
    
*   Nasuni Support is limited to Nasuni APIs and Protocols (SMB, NFS, SNMP, GFA Telemetry, Nasuni Access Anywhere Monitoring API). Nasuni API and Protocol bugs should be reported to Nasuni Customer Support.
    
*   GitHub project bugs, questions, and feature requests should be submitted as “Issues” in GitHub under its repositories.


# Installation Guidelines

## Validated Component Versions

Nasuni Dashboards has been validated with these component versions: 

* Telegraf: 1.28.2

* InfluxDB: 2.7.1
    
* Grafana: 10.1.5

## VM Requirements for Single-Node Installation 

* OSes: 

  - Rocky Linux 8.6, 9.0, or 9.2 minimal, sourced from cloud marketplace offerings or Rocky Linux ISO

  - Windows Server 2019 or 2022 64-bit (x86)

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

<table data-layout="default" data-local-id="8e403bb5-545e-4876-8e57-082bc184df20" class="confluenceTable"><colgroup><col style="width: 113.33px;"><col style="width: 113.33px;"><col style="width: 113.33px;"><col style="width: 113.33px;"><col style="width: 113.33px;"><col style="width: 113.33px;"></colgroup><tbody><tr><th class="confluenceTh"><p style="text-align: center;"><strong>vCPU or CPU</strong></p><p style="text-align: center;">(Cores)</p></th><th class="confluenceTh"><p style="text-align: center;"><strong>RAM</strong></p><p style="text-align: center;">(GB)</p></th><th class="confluenceTh"><p style="text-align: center;"><strong>IOPS</strong></p></th><th class="confluenceTh"><p style="text-align: center;"><strong>Writes per second</strong></p></th><th class="confluenceTh"><p style="text-align: center;"><strong>ueries per second</strong></p></th><th class="confluenceTh"><p style="text-align: center;"><strong>Unique</strong></p><p style="text-align: center;"><strong>series</strong></p></th></tr><tr><td class="confluenceTd"><p style="text-align: center;">2-4</p></td><td class="confluenceTd"><p style="text-align: center;">2-4</p></td><td class="confluenceTd"><p style="text-align: center;">500</p></td><td class="confluenceTd"><p style="text-align: center;">&lt; 5,000</p></td><td class="confluenceTd"><p style="text-align: center;">&lt; 5</p></td><td class="confluenceTd"><p style="text-align: center;">&lt; 100,000</p></td></tr><tr><td class="confluenceTd"><p style="text-align: center;">4-6</p></td><td class="confluenceTd"><p style="text-align: center;">8-32</p></td><td class="confluenceTd"><p style="text-align: center;">500-1000</p></td><td class="confluenceTd"><p style="text-align: center;">&lt; 250,000</p></td><td class="confluenceTd"><p style="text-align: center;">&lt; 25</p></td><td class="confluenceTd"><p style="text-align: center;">&lt; 1,000,000</p></td></tr><tr><td class="confluenceTd"><p style="text-align: center;">8+</p></td><td class="confluenceTd"><p style="text-align: center;">32+</p></td><td class="confluenceTd"><p style="text-align: center;">1000+</p></td><td class="confluenceTd"><p style="text-align: center;">&gt; 250,000</p></td><td class="confluenceTd"><p style="text-align: center;">&gt; 25</p></td><td class="confluenceTd"><p style="text-align: center;">&gt; 1,000,000</p></td></tr></tbody></table>

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

    b. SNMPv1/v2: Set **Enable v1,v2c Support**  to **On**. Set the **Community Name** to **public** (or what you plan to use).
    
6.  Enter your preferred **System Location** and **System Contact**.
    
7.  Click **Save SNMP Settings**.

## GFA Telemetry API (Optional)
To use the GFA Telemetry API:

*   You must have a volume under GFA management (Active or Observation mode)

*   You must create a [Global File Accelerator API key on the Nasuni dashboard](https://account.nasuni.com/account/gfa/). Note: If the link is not accessible, you may not be licensed for GFA. Contact your Nasuni Account Manager for assistance.

## Nasuni Access Anywhere Monitoring API (Optional)
Nasuni Access Anywhere can be extended to provide a rich metrics API that can provide a dashboard of system and application metrics.

<details>
    <summary>Expand Nasuni Access Anywhere Monitoring API Configuration Instructions</summary>
<br/>

### Configuring Nasuni Access Anywhere Monitoring API (Naamon)
The monitoring API will be installed from the CLI via the standard package deployments (yum/rpm). Steps:

1. SSH to the NAA VM and run the following commands:

   ```shell
   sudo yum -y install naamon
   ```
2. Update the firewall to allow the default port for Naamon:

   * Add the following line into /etc/sysconfig/iptables before the "-j REJECT" line and save the changes:
     ```shell
     -A RH-Firewall-1-INPUT -p tcp -m state --state NEW -m tcp --dport 8010 -j ACCEPT
     ```

   * Restart the firewall and docker
     ```shell
     sudo systemctl restart iptables docker
     ```
### Naamon Configuration File Changes
Naamon leverages the following configuration file: **/etc/naa/naamon.hcl**.
There is one required change that is required for this setup, and other optional changes as well.

1. AllowList IP Addresses (required)
   - The default config file has just localhost in the allow list:

     ```shell
        	 allowlist {
        	    ip = [
        	        "127.0.0.1"
        	        ]
    	     }
     ```

   - Add your TIG server or any other servers/networks that should be allowed to access the APIs using the following example for reference:

        ```shell
            	allowlist {
            	    ip = [
            	        "127.0.0.1",
                        "172.20.25.0/24",
            	        "172.20.1.128"
            	        ]
    	       }
        ```
2. HTTPS (Optional)

   The default configuration has all API metrics data transferred in plain text (HTTP). To ensure encryption for all API data in transit, update the SSL setting, replacing **false** with **true** like this example:
   ```shell
	      ssl "true" {
	          certfile = "/etc/ssl/fullchain.pem"
	          keyfile = "/etc/ssl/privkey.pem"
	      }
   ```
   Edit **certfile** and **keyfile** to point to your relevant SSL files.

   **Note**: When renewing or updating certs, you must restart the Naamon service for the new certificates to be available. 

4. Database Replication (Optional)
   - The default Naamon configuration does not check for database replication status. If your system is using a replicated  database, and you want to monitor the status of database replication via Naamon, update the configuration file like this:
     
     ```shell
	     dbrepl "true" {
	         username = "replicationcheckuser"
	         password = "replicationcheckerpasswordhere"
	     }
     ```
     
   - The user in the configuration must be created in the database and have permission to check the replication status. Here is an example command that will setup the user in the database with the correct permissions:
     
     ```shell
     sudo mysql -e "CREATE USER 'replicationcheckuser'@'localhost' IDENTIFIED BY 'ReplicationCheckerPasswordHere';"
	 sudo mysql -e "GRANT REPLICATION SLAVE ON *.* TO 'replicationcheckuser'@'localhost';"
     ```
### Starting the Naamon service
The Naamon application is installed as a systemd service. 

* To start the service and enable it, run the following command after updating the configuration:

  ```shell
  sudo systemctl enable --now naamon
  ```
* Check the Naamon service status:

  ```shell
  systemctl status naamon
  ```
</details>  
    
# Install and Configure Nasuni Dashboards

## Deploy the VM
Deploy a Rocky Linux or Windows VM that meets the requirements for Nasuni Dashboards. 

NOTE: Be sure to update all the OS packages by running (Linux)

```shell
sudo yum -y update
```

## Configure DNS
DNS is required to use hostnames rather than IP addresses for the Telegraf SNMP agent configuration. 
Two common scenarios for DNS configuration:

*   DHCP: DHCP is typically used for Cloud VMs, but can also be used on-premises. If the DNS server provided by DHCP is not authoritative for Edge Appliance hostnames, you can override the DNS server settings.

    *   To enable static DNS with DHCP for Rocky Linux, SSH to the VM and run the following command to disable DHCP updates to DNS:

        ```shell
        sudo tee -a /etc/NetworkManager/conf.d/disable-resolve.conf-managing.conf > /dev/null <<EOT
        [main]
        dns=none
        EOT
        ```

    *   Edit **/etc/resolv.conf**, specifying nameserver entries for the authoritative DNS servers for Edge Appliance DNS.
    
        ```shell
        sudo vi /etc/resolv.conf
        ```

*   Static IP: Ensure that the specified DNS servers are authoritative for Edge Appliance DNS.

## Download Nasuni Dashboards Zip
From your computer (Rocky Linux) or the Windows VM (Windows Install), click the green **Code** button within the Nasuni Labs nasuni-dashboards repository. Select the **Download ZIP** option to download a local copy of the repository and extract the contents.

## Install and Configure InfluxDB

#### Rocky Linux InfluxDB 2.7+ Installation Instructions
<details>
    <summary>Expand Rocky Linux Installation Instructions</summary>
<br/>

To install InfluxDB on Rocky Linux, ssh to the VM and run the following commands:

1.  Update all packages if you haven't already (*-y* argument skips confirmation; for manual confirmation, remove it):
    
    ```shell
    sudo yum -y update
    ```
    
2.  Install wget, download, install InfluxDB and client tools, and start the service:

    ```shell
    sudo yum -y install wget && cd ~ && wget https://dl.influxdata.com/influxdb/releases/influxdb2-2.7.1.x86_64.rpm && wget https://dl.influxdata.com/influxdb/releases/influxdb2-client-2.7.3.x86_64.rpm && sudo yum -y localinstall influxdb2-2.7.1.x86_64.rpm influxdb2-client-2.7.3.x86_64.rpm && sudo systemctl start influxdb && sudo systemctl enable influxdb
    ```    
    
3.  Add firewall rules if the firewall service is running. First, check if the firewall service is running:

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

#### Windows InfluxDB 2.7+ Installation Instructions
<details>
    <summary>Expand Windows Installation Instructions</summary>
<br/>

To install InfluxDB on Windows, connect to the Windows VM and follow these instructions:

1.  Run PowerShell as an administrator, then enter the following commands to download and install InfluxDB:

    ```PowerShell
    mkdir ~\Downloads\InfluxInstall; cd ~\Downloads\InfluxInstall; Invoke-WebRequest https://dl.influxdata.com/influxdb/releases/influxdb2-2.7.1-windows-amd64.zip -UseBasicParsing -OutFile influxdb2-2.7.1-windows-amd64.zip ; Expand-Archive .\influxdb2-2.7.1-windows-amd64.zip -DestinationPath .\ ;rm influxdb2-2.7.1-windows-amd64.zip ;mkdir 'c:\Program Files\InfluxData\InfluxDB' ;mv .\influxdb2_windows_amd64\* 'c:\Program Files\InfluxData\InfluxDB\'; rmdir .\influxdb2_windows_amd64\; Invoke-WebRequest https://dl.influxdata.com/influxdb/releases/influxdb2-client-2.7.3-windows-amd64.zip -OutFile influxdb2-client-2.7.3-windows-amd64.zip ; Expand-Archive .\influxdb2-client-2.7.3-windows-amd64.zip -DestinationPath .\;rm .\influxdb2-client-2.7.3-windows-amd64.zip; mkdir 'c:\Program Files\InfluxData\influxdb2-client' ;mv .\* 'c:\Program Files\InfluxData\influxdb2-client\'
    ```
    
2. Download and configure the NSSM service manager to make InfluxDB run as a Windows Service (The final output should show "Status: Running", "Name: InfluxDB"):
   
   ```PowerShell
   mkdir ~\Downloads\NssmInstall; cd ~\Downloads\NssmInstall; Invoke-WebRequest https://nssm.cc/release/nssm-2.24.zip -UseBasicParsing -OutFile nssm-2.24.zip ;Expand-Archive .\nssm-2.24.zip -DestinationPath .\ ;mkdir 'c:\Program Files\NSSM' ;mv .\nssm-2.24\win64\nssm.exe 'c:\Program Files\NSSM\' ;cd 'c:\Program Files\NSSM\' ;.\nssm.exe install InfluxDB "C:\Program Files\InfluxData\InfluxDB\influxd.exe" ;.\nssm.exe set InfluxDB Description "InfluxDB Time-Series Database" ;Start-Service InfluxDB ; Get-Service InfluxDB
   ```
</details>    

#### Create the InfluxDB 2.7+ database:
    
1.  Open the InfluxDB shell:

    * Rocky Linux - ssh to the VM and run the following command, substituting values for the variables in brackets (do not include the brackets):

        ```shell
        influx setup --name <databaseName> --host http://localhost:8086 -u <username> -p <password> -o <orgName> -b <bucketName> -r 0 -f
        ```
        
        Note: Telegraf and Grafana InfluxDB authentication do not use the username or password defined here (both use an automatically generated token)

        Populated Example:
      
        `influx setup --name nasuni --host http://localhost:8086 -u admin -p Password123 -o MyCompany -b nasuni-bucket -r 0 -f`

    * Windows - Use PowerShell to configure InfluxDB, substituting values for the variables in brackets (do not include the brackets):
  
       ```PowerShell
       cd "c:\Program Files\InfluxData\influxdb2-client"; .\influx.exe setup --name nasuni --host http://localhost:8086 -u <username> -p <password> -o <orgName> -b <bucketName> -r 0 -f
       ```

       Note: Telegraf and Grafana InfluxDB authentication do not use the username or password defined here (both use an automatically generated token)
      
       Populated Example:
      
        `cd "c:\Program Files\InfluxData\influxdb2-client"; .\influx.exe setup --name nasuni --host http://localhost:8086 -u admin -p Password123 -o MyCompany -b nasuni-bucket -r 0 -f`

    The output will be similar to the following:
    
      `User Organization Bucket`</br>
      `admin MyCompany nasuni-bucket`</br>

3.  List the automatically created database token and make a note of the token since it will be required for configuring Grafana database authentication:

    * Rocky Linux - SSH to the VM and run the following command to list Influx authentication info:
      ```shell
      influx auth list
      ```
    * Windows - Use PowerShell to list Influx authentication info:
      ```Powershell
      cd "c:\Program Files\InfluxData\influxdb2-client"; .\influx.exe auth list
      ```
    The output will be similar to the following (ellipsis added for readability):
    
    `[~]$ influx auth list`</br>
    `ID Description Token User Name	User ID Permissions`</br>
    `0b7c38544ff51000 admin's Token 5xi_9lX...GFg== admin 0b7c3...1000 [read:/authorizations...]`</br>

        

## Install and Configure Telegraf

### Rocky Linux Telegraf Installation Instructions
<details>
    <summary>Expand Rocky Linux Installation Instructions</summary>
<br/>

1.  Download Telegraf, install it, and backup the default configuration file:
    
    ```shell
    cd ~ && wget https://dl.influxdata.com/telegraf/releases/telegraf-1.28.2-1.x86_64.rpm && sudo yum -y localinstall telegraf-1.28.2-1.x86_64.rpm && sudo yum -y install net-snmp net-snmp-utils && sudo mv /etc/telegraf/telegraf.conf /etc/telegraf/telegraf.conf.bak
    ```
    
</details>
  
### Windows Telegraf Installation Instructions
<details>
    <summary>Expand Windows Installation Instructions</summary>
    
1.  Run PowerShell as an administrator, then run commands to download and install Telegraf:
    
    ```PowerShell
    mkdir ~\Downloads\TelegrafInstall ;cd ~\Downloads\TelegrafInstall; Invoke-WebRequest https://dl.influxdata.com/telegraf/releases/telegraf-1.28.2_windows_amd64.zip -UseBasicParsing -OutFile telegraf-1.28.2_windows_amd64.zip ;Expand-Archive .\telegraf-1.28.2_windows_amd64.zip -DestinationPath .\ ;mkdir 'c:\Program Files\Telegraf' ;mv .\telegraf-1.28.2\* 'C:\Program Files\Telegraf\' ;mv 'C:\Program Files\Telegraf\telegraf.conf' 'C:\Program Files\Telegraf\telegraf.conf.bak'
    ```    
    
</details>
    
### Telegraf Configuration

1.  Edit the telegraf.conf file:

    - Rocky Linux: On your computer, open the **telegraf.conf** file from the extracted Nasuni Dashboards repository zip archive in a text editor, select all, and copy the contents to your clipboard. Return to the VM SSH session, open vi, and paste the contents from step 2, customizing the values for the sections below:

       ```shell
       sudo vi /etc/telegraf/telegraf.conf
       ```
       
    - Windows: In the Windows VM, copy the contents of the **telegraf.conf** file from the extracted Nasuni Dashboards repository zip archive to "C:\Program Files\Telegraf\telegraf.conf" and make the edits outlined in the next set of steps. One way to do this is to run Notepad or Notepad++ as an administrator and paste the contents of telegraf.conf. Example to open telegraf.conf for editing in Notepad using PowerShell (running PowerShell as administrator):

        ```PowerShell
        notepad.exe "C:\Program Files\Telegraf\telegraf.conf"
        ```   
    
2.  In the **[outputs.influxdb_v2]** make the following changes
       - token (replace **token** and brackets with the token from the **influx auth list** command)
       - organization (replace **myOrg** and brackets with the case-sensitive organization you specified during **influx setup**)
       - bucket (replace **bucket** and brackets with the bucket you specified during **influx setup**. Bucket name is case-sensitive.)
        
3.  In the **[inputs.snmp]** section, update the **agents** value with the FQDN of all Edge Appliances to be monitored. For example (Note: The last entry does not require a trailing comma):</br>
    
    ```shell
       agents = [
            "edge1.yourdomain.com",
            "edge2.yourdomain.com",
            "edge3.yourdomain.com"    
        ]
    ```
            
4.  In the **[inputs.snmp]** section, update protocol-specific values based on the protocol configured for your Edge Appliances and NMC: 
    - SNMPv3

        - `sec_name = "<username>"` (replace **username** and brackets with the **Username** set in the Nasuni SNMP UI)

        - `auth_password = "<password>"` (replace **password** and brackets with the **Password** set in the Nasuni SNMP UI)
    
    - SNMPv2: The included telegraf.conf assumes SNMPv3, so some values must be changed from the defaults and commented/uncommented (comment lines begin with **#**) to use SNMPv2:

        - `version = 2`

        - `community = "public"` (uncomment this line and set the value if not using the **public** default)

        - comment out (add **#** to the beginning) lines SNMPv3-related lines that start with the following:

            - `sec_name`

            - `auth_protocol`

            - `auth_password`

            - `sec_level`

5.  In the **[inputs.snmp.tags]** section, update the **customer** and **country** values with appropriate information for your environment. Use two-letter country codes for the country value:</br>

    - `customer  = "<CustomerName>"`

    - `country = "<CountryCode>"`
        
6.  To use the optional Global File Acceleration (GFA) Telemetry API data source, change the **GFA Telemetry HTTP API Input Plugin [[inputs.http]]** section:

    - `urls` (uncomment this line--the URL value does not need to change)

    - `headers` (uncomment the first headers line and enter the GFA API Key from the NOC Dashboard)

    - `headers` (uncomment the second headers line and enter a string/name to use as a unique identifier for this connection)

7.  To use the optional Nasuni Access Anywhere (NAA) API data source, change the **NAA HTTP API Input Plugin [[inputs.http]]** section:

    - uncomment all lines in this section, including the `[[inputs.http]]` section header

    - `urls` enter the hostname(s) of your NAA servers, replacing **NaaHostName** and brackets. If monitoring multiple NAA servers, use commas to separate each hostname entry.


8.  Save and close the file. For Rocky Linux editors using vi, press **Esc**, enter **x** at the prompt, and press **Enter**.

9. Make final changes to the telegraf.conf file permissions (Linux) and start Telegraf:

     * Rocky Linux
     
        * Set the ownership and permissions for telegraf.conf, restart Telegraf, and confirm the Telegraf service is running. Look for **Active: active (running)**:

          ```shell
          sudo chown root:root /etc/telegraf/telegraf.conf && sudo chmod 644 /etc/telegraf/telegraf.conf && sudo systemctl restart telegraf && systemctl status telegraf --no-pager
          ```
            
     * Windows
       
         * Run PowerShell as an administrator to install Telegraf as a Windows service and start it (The final output should show "Status: Running","Name: Telegraf"):

           ```Powershell
           cd "C:\Program Files\Telegraf" ;.\telegraf.exe --service install --config "C:\Program Files\Telegraf\telegraf.conf" ;Start-Service Telegraf ; Get-Service Telegraf
           ```

## Install and Configure Grafana

### Rocky Linux Grafana Installation Instructions
<details>
    <summary>Expand Rocky Linux Installation Instructions</summary>


1.  Download Grafana, install, start, enable the service, and confirm the Grafana service is running. Look for **Active: active (running)**:
    
    ```shell
    cd ~ && wget https://dl.grafana.com/oss/release/grafana-10.1.5-1.x86_64.rpm && sudo yum -y install grafana-10.1.5-1.x86_64.rpm && sudo grafana-cli plugins install grafana-clock-panel && sudo systemctl start grafana-server && sudo systemctl enable grafana-server && systemctl status grafana-server --no-pager
    ```
    
2.  Configure the firewall for Grafana (only required if the firewall is enabled):
    
    ```shell
    sudo firewall-cmd --permanent --zone=public --add-port=3000/tcp && sudo firewall-cmd --reload
    ```
 </details>
 
 ### Windows Grafana Installation Instructions
<details>
    <summary>Expand Windows Installation Instructions</summary>
    
1.  Run PowerShell as an administrator and run commands to download and install Grafana. Accept all the defaults (Grafana will automatically be installed as a Windows service and started) :
    
    ```PowerShell
    cd ~\Downloads ;Invoke-WebRequest https://dl.grafana.com/oss/release/grafana-10.1.5.windows-amd64.msi -UseBasicParsing -OutFile grafana-10.1.5.windows-amd64.msi ;.\grafana-10.1.5.windows-amd64.msi
    ```
    
2.  If Windows firewall is active, open TCP port 3000 for inbound access so remote computers can access the Grafana Dashboard:

    ```PowerShell
    New-NetFirewallRule -DisplayName 'Allow Grafana Port 3000' -Direction Inbound -LocalPort 3000 -Protocol TCP -Action Allow
    ```
    
</details>

### Configure Grafana Data Source

1.  Use a web browser to connect to Grafana (http://fqdn-of-vm:3000).
    
2.  Log in to Grafana. The default username is **admin**, and the default password is **admin**. Grafana will prompt you to change the default password after logging in.
    
3.  Under the Connections (two overlapping circles) icon, add InfluxDB as a data source using the details from your install:

    - Name(1): InfluxDB (Make sure default is toggled on)
    - URL(2): http://localhost:8086 (if all components are installed on the same computer, use **localhost** as the value otherwise, replace localhost with the FQDN of the InfuxDB server)
    - Custom HTTP Headers (3): Click **Add Header**. Provide your InfluxDB API token:
         - Header: Enter **Authorization**
         - Value: Use the Token schema (the word **Token** followed by a space and the token value) and provide your InfluxDB API token (the same token you entered telegraf.conf). For example:</br>
           `Token y0uR5uP3rSecr3tT0k3n`
        
    - Database (4): your InfluxDB bucket name (**nasuni-bucket** if you follow installation examples). Bucket name is case-sensitive.
    
![Add Data Source](/images/AddGrafanaDataSource.png)

4.  Click **Save & test** (5) to complete adding the InfluxDB data source. A green checkbox should appear alongside a message that the **Data source is working**.

### Configure Grafana Dashboards

1.  From the Grafana left navigation bar, click the Dashboards (four squares) icon, then click the **Import** link. On the Import page, click **Import** and upload the **PerformanceDashboard.json** file from the extracted repository zip archive.

![TIGStackComponents](/images/GrafanaImport.png)

2.  The **Options** screen opens. Select the **InfluxDB** data source name, which should be **InfluxDB (default)**, and click **Import**.

3.  Navigate to the newly imported dashboard to view the performance information for your Edge Appliances. Here is an example of the expected output for the dashboard:


![Performance Dashboard Screenshot](/images/PerformanceDashboardScreenshot.png)

4. If using the GFA Telemetry API data source, import the associated GFA dashboards. From the Grafana left navigation bar, click the Dashboards (four squares) icon, then click the **Import** link. On the Import page, click **Import** and upload the **GFAVolumes.json** file from the extracted repository zip archive, accepting the defaults and saving the configuration. Repeat for the **GFAAppliances.json** and **GFAMiscellaneous.json** files.

5. If using the NAA API data source, import the NAA dashboard. From the Grafana left navigation bar, click the Dashboards (four squares) icon, then click the **Import** link. On the Import page, click **Import** and upload the **NasuniAccessAnywhere.json** file from the extracted repository zip archive, accepting the defaults and saving the configuration.

6. Navigate to the newly imported GFA Volumes dashboard. Here is an example of the expected output for the dashboard:

![GFA Volumes Dashboard Screenshot](/images/GFAVolumesDashboardScreenshot.png)

6. If you use the GFA Miscellaneous dashboard, install the Gantt Chart plugin. Follow the instructions https://grafana.com/grafana/plugins/marcusolsson-gantt-panel/?tab=installation to download and install the plugin.

# Troubleshooting

## Data Unavailable in Performance Dashboard
Telegraf depends on SNMP data from the Edge Appliances to work. You can confirm that SNMP is configured and working on the Edge Appliance by connecting to the server where you installed the TIG stack over SSH (Rocky Linux) or RDP (Windows) and using SNMP utilities to load SNMP data. 

## SNMP Troubleshooting

## Rocky Linux SNMP Troubleshooting
<details>
    <summary>Expand Rocky Linux Snmpwalk Instructions</summary>
<br/>
    
Snmpwalk can test SNMP connectivity and inspect values returned for an SNMP Object Identifier (OID). Nasuni publishes a full list of [OIDs for the Nasuni Edge Appliance](http://b.link/Nasuni_SNMP_OIDs_for_NASUNI-FILER-MIB). Snmpwalk for Linux is automatically installed when installing Nasuni Dashboards, although the syntax to use it differs by SNMP version.
    
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

Windows does not include SNMP troubleshooting tools, although several third-party tools are available. [SnmpGet](https://ezfive.com/snmpsoft-tools/snmp-get/) is an easy-to-use command-line SNMP utility for Windows. It can verify SNMP connectivity and inspect values returned for an SNMP Object Identifier (OID). Nasuni publishes a full list of [OIDs for the Nasuni Edge Appliance](http://b.link/Nasuni_SNMP_OIDs_for_NASUNI-FILER-MIB). After downloading SnmpGet, CD to the directory where you saved the file using PowerShell and run it using the version-specific SNMP commands below.
    
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
If Telegraf reports errors in **/var/log/telegraf/telegraf.log**, the source of the problem is most likely with the formatting or contents of telegraf.conf. 

#### Telegraf.conf Format Validation
The telegraf.conf file uses the [TOML](https://toml.io/en/) format. Omitting or adding unexpected characters to telegraf.conf can invalidate the configuration file. You can validate telegraf.conf using a TOML file validator like the [TOML Lint](https://www.toml-lint.com/) website.
    
#### Validating Telegraf.conf Contents
Run the following command to test telegraf.conf (the command will report a verbose error if it encounters a problem):
    
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

## Upgrading from InfluxDB 1.8 to 2.7

When it launched, Nasuni Dashboards only supported InfluxDB 1.8 and now supports InfluxDB 2.7. You can use these instructions to update from InfluxDB 1.8 to 2.7 while retaining performance data.

#### Rocky Linux InfluxDB 1.8 to 2.7 Upgrade Instructions
<details>
    <summary>Expand Rocky Linux InfluxDB Upgrade Instructions</summary>
<br/>

To upgrade InfluxDB on Rocky Linux, SSH to the VM and run the following commands:

1.  Export historical data, replacing **database** and brackets with the database you specified during the InfluxDB 1.8 install (usually **nasuni**). Confirm the export has been completed. Look for **writing out wal file data for nasuni\autogen...complete.** Depending on how long you've been using Nasuni Labs and how many appliances you have, export and import (later in the steps) could take quite a while.
    
    ```shell
    sudo influx_inspect export -database <database> -datadir /var/lib/influxdb/data -waldir /var/lib/influxdb/wal -out $HOME/influxExport.lp -lponly
    ```
    
2.  Stop InfluxDB 1.8, uninstall it, and remove the associated application folders:

    ```shell
    sudo systemctl stop influxdb && sudo yum  -y remove influxdb.x86_64 && sudo rm -rf /var/lib/influxdb/ && sudo rm -rf /etc/influxdb/ && sudo rm -rf /var/log/influxdb/
    ```    
    
3. Complete the steps to [install and configure InfluxDB 2.7 on Rocky Linux](#rocky-linux-influxDB-2.7+-installation-instructions).
   
4. Restore the historical data you exported in step 1, replacing **bucketname** and brackets with the bucket name you specified during the InfluxDB 2.7 install:

   ```shell
   sudo influx write --bucket <bucketname> --file $HOME/influxExport.lp
   ```
   
</details>

#### Windows InfluxDB 1.8 to 2.7 Upgrade Instructions
<details>
    <summary>Expand Windows InfluxDB Upgrade Instructions</summary>
<br/>

To upgrade InfluxDB on Windows, RDP into the VM, run PowerShell as an administrator, and run the following commands:

1.  Export historical data, replacing **database** and brackets with the database you specified during the InfluxDB 1.8 install (usually **nasuni**). Confirm the export has been completed. Look for **writing out wal file data for nasuni\autogen...complete.** Depending on how long you've been using Nasuni Labs and how many appliances you have, export and import (later in the steps) could take quite a while.
    
    ```PowerShell
    cd 'C:\Program Files\InfluxData\InfluxDB'; .\influx_inspect.exe export -database <database> -datadir C:\Windows\System32\config\systemprofile\.influxdb\data -waldir C:\Windows\System32\config\systemprofile\.influxdb\wal -out $HOME\influxExport.lp -lponly
    ```
    
2.  Stop InfluxDB 1.8, uninstall it, and remove the associated application folders:

    ```PowerShell
    cd $HOME; Stop-Service InfluxDB; Remove-Item -Path 'C:\Program Files\InfluxData' -Recurse -Force; Remove-Item -Path 'C:\Windows\System32\config\systemprofile\.influxdb' -Recurse -Force
    ```    
    
3. Complete the steps to [install and configure InfluxDB 2.7 on Windows](#windows-influxDB-2.7+-installation-instructions). Skip the last step to install InfluxDB as a service since the 1.8 service configuration remains valid for 2.7.

4. Start the InfluxDB service

   ```PowerShell
   Start-Service InfluxDB
   ```
   
5. Restore the historical data you exported in step 1, replacing **bucketname** and brackets with the bucket name you specified during the InfluxDB 2.7 install:

   ```PowerShell
   cd "c:\Program Files\InfluxData\influxdb2-client"; .\influx.exe write --bucket <bucketname> --file $HOME\influxExport.lp
   ```

</details>

#### Telegraf and Grafana Changes

Update telegraf.conf to use InfluxDB 2.7:

1. Open telegraf.conf for editing using the [Editing Telegraf Configuration](#editing-telegraf-configuration) instructions and go to the **[outputs.influxdb]** section for Influx 1.8 and add a **#** to the beginning of each line (including the **[outputs.influxdb]** section header) to comment them out.

2. In the **[outputs.influxdb_V2]** section for Influx 2.7 (if your telegraf.conf does not include this section, copy from telegraf.conf in Nasuni Labs) and remove the **#** from the beginning of the following lines to uncomment them and populate them with the following values:

   - **[outputs.influxdb_v2]** section header (uncomment only)
   - URLs (uncomment only)
   - token (uncomment and replace **token** and brackets with the token from the **influx auth list** command you ran during setup)
   - organization (uncomment and replace **myOrg** and brackets with the organization you specified during **influx 2.7 setup**)
   - bucket (uncomment and replace **bucket** and brackets with the bucket you specified during **influx 2.7 setup**)
    
3. Save and close the file.

4. [Restart Telegraf](#restart-the-telegraf-service) to load the changes.
  
5. Reconfigure your Grafana Data source to use InfluxDB 2.7 (edit the existing InfluxDB Grafana data source rather than adding a new one) using the [Configure Grafana Data Source instructions](#configure-grafana-data-source) in Nasuni Labs. InfluxDB 2.7 requires a new Authorization header for Grafana, and the database name should now reference the InfluxDB bucket name.

## Editing Telegraf Configuration

To edit the telegraf.conf file:
    
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
It is easy to add performance monitoring for newly deployed Edge Appliances.
    
1.  Edit telegraf.conf using the instructions above. In the **[inputs.snmp]** section, update the **agents** value to include the additional Edge Appliance monitor and save the changes. (Note that the last entry does not need a trailing comma.)
    
2.  Restart the Telegraf service using the instructions above.

## Removing Stale Edge Appliances

When decommissioning a Nasuni Edge Appliance, cleaning up the InfluxDB database to save disk space and remove stale data from Grafana is good.

1.  To remove the Nasuni Edge Appliance IP or FQDN from Telegraf, edit the telegraf.conf file.
    
2.  In the **[inputs.snmp]** section, update the **agents** value to remove the unwanted Edge Appliance monitor and save telegraf.conf. (Note that the last entry does not need a trailing comma.)
    
3.  Start/Restart the Telegraf service:

    ```shell
    sudo systemctl restart telegraf
    ```
    
4.  Launch the influx v1 shell:

    * Rocky Linux
    
        ```shell
        sudo influx v1 shell
        ```
    * Windows - Run the following PowerShell Command:
    
        ```PowerShell
        cd "c:\Program Files\InfluxData\influxdb2-client"; .\influx.exe v1 shell
        ```
    
5.  Access the database, replacing **bucket_name** and brackets with the value from your install:
    
    ```shell
    use <bucket_name>
    ```
    The bucket_name is **nasuni-bucket**, or the value you supplied when "Creating the InfluxDB database" above.
    
6.  Display the series (a logical grouping of data defined by shared measurement, tag set, and field key):
    
    ```shell
    select * from Nasuni
    ```
    
7.  Identify the agent\_host to delete.
    
8.  Delete all data for the agent_host, replacing the **FQDN** and brackets with the case-sensitive FQDN:

    ```shell
    DELETE FROM "Nasuni" where agent_host = '<FQDN>'
    ```

9. Verify if the command was successful:
    
    ```shell
    select * from Nasuni
    ```

10. Exit:
    
    ```shell
    exit
    ```
    


    

