# Python-For-IT-Automation - Scripts

This holds the python scripts I wrote for my class assignments. 

One big thing to remember about Python, indents matter... **a lot**!

The way this is setup:
- Tasks #
  - Task Introduction
  - Task Scenario
  - Problem
  - Scripts
    - Learning Objective of the script
    - Pseudocode for the scrip layout
    - The actual script (I have the collapsed so you can try it for yourself and check it with mine)
    
---

<details>
<summary><strong>Task 1: Incident Response and Remediation</strong></summary>

## Task 1 – Incident Response and Remediation

### Introduction  
In this task, you will act as a company's network administrator during a cybersecurity incident in which the
internal Domain Name System (DNS) service is down and devices are resolving through a rogue DNS
address. Your mission is to quickly identify the root cause, verify and correct configurations on impacted
devices, and implement enduring safeguards to prevent recurrence. You will manage a GitLab-based project
with working branches, develop Python scripts to enumerate devices, verify connectivity and DNS settings,
automatically notify stakeholders, create remediation tickets, and restore the DNS service while ensuring all
affected devices are properly reconfigured.

### Scenario  
As the network administrator for your company, you are alerted to a cybersecurity incident involving a DNS
service outage. The internal DNS service is currently down, and you discover that several network devices
have been reconfigured to use an unauthorized, potentially malicious DNS address. Immediate action is
required to restore proper DNS functionality and secure the network.  
You are responsible for identifying the root cause of the DNS resolution issue and restoring normal
operations. This includes investigating the source of the unauthorized DNS changes, verifying and correcting
DNS configurations on all affected devices, and implementing immediate remediation steps to secure the
network against further compromise.

### Problem Statement
Design and implement a solution that:
- Determine the source of the issue
- Notify stakeholders and create ticket entries
- Remediate the DNS service and affected devices
- Notify stakeholders of the resolution


---

<details>
<summary><strong>Script 1 - Read Network Devices from CSV</strong></summary>
  
  ## Script 1 – Read Network Devices from CSV
  
  **Learning Objectives:**
  - Import required modules
  - Variables
  - Open, read and display a CSV file
  
  **Pseudocode**
  1. Import required modules
  2. Set variables
  3. Define the path to the CSV file
  4. Open the CSV file
  5. Read each row
  6. Extract and display the device name
  7. (Bonus) Format the output for readability
  
  <details>
    <summary><strong>Actual Script</strong></summary>
      
      ```Python
      # This script will be used to read and list all the devices in the csv file.

      # Imports
      import os                                       # Because I like a clean screen
      from utility_get_devices import get_devices
      from pathlib import Path
      
      # Constant
      CSV_FILE = Path("/home/student/d522/d522-python-for-it-automation/network_devices.csv")
      
      devices = get_devices(CSV_FILE)
      
      # Set up
      os.system("clear")
      print("-" * 60)
      print("Network Devices")
      print("-" * 60)
      
      # Print devices
      for device in devices:
          print(f"{device['name']:13}, {device['ip']}")
      
      
      # Closure
      print("-" * 60)
      print("List Completed - Goodbye")
      print("-" * 60)
      
      
      # Ends the code block for B1
      
      ```
      
  </details> <!-- Ends the actual script -->

  ---
  
  </details> <!-- Ends Script 1 Section-->

  <details>
  <summary><strong>Script 2 - Ping devices, Determine status, Verify DNS</strong></summary>

  ## Script 2 - Ping devices, Determine status, Verify DNS

  **Learning Objectives**
- Ping network devices to determine reachability
- Establish SSH connections to retrieve configuration data
- Verify DNS settings against an expected value
- Generate a clear status report

**Pseudocode**
1. Import required modules
2. Define the expected DNS server and CSV file path
3. Create a function to ping a device
4. Create a function to SSH into a device and retrieve its current DNS setting
5. Open and read the CSV file
6. For each device:
   - Skip devices with no management access
   - Ping the device to check if it is reachable
   - If reachable, retrieve the current DNS configuration via SSH
   - Compare the DNS setting to the expected value
   - Print the device name, IP address, and status
7. Display a completion message
  
  <details>
  <summary><strong>Actual Script</strong></summary>

    ``` Python
    # b2_check_device_status.py
    # This script pings each device from network_devices.csv and reports its status.
    # B2 - fixed
    
    import os                   # Because I like a clean screen
    import subprocess           # Used to perform the ping
    import paramiko             # Used for SSH
    from pathlib import Path    # Used to get the location of the csv file
    from utility_get_devices import get_devices # My new baby.
    
    EXPECTED_DNS = "10.10.10.10"
    CSV_FILE = Path("/home/student/d522/network_devices.csv")
    
    # defines the ping process with the given ip address
    def ping_device(ip):
        try:
            # Simple and reliable ping
            result = subprocess.run(
                ["ping", "-c", "2", ip],
                stdout=subprocess.DEVNULL,
                stderr=subprocess.DEVNULL
            )
            return result.returncode == 0
        except Exception:
            return False
    
    # defines the ssh connection for each device
    def get_current_dns(ip, username, password):
        try:
            ssh = paramiko.SSHClient()
            ssh.set_missing_host_key_policy(paramiko.AutoAddPolicy())
            ssh.connect(ip, username=username, password=password, timeout=5)
    
            stdin, stdout, stderr = ssh.exec_command("cat /etc/resolv.conf")
            output = stdout.read().decode()
            ssh.close()
    
            for line in output.splitlines():
                if "nameserver" in line:
                    return line.strip().split()[-1]
            return "No DNS found"
        except Exception as e:
            return f"Error: {str(e)}"
    
    # prints the status report
    os.system("clear")
    print("Device Status & DNS Verification Report")
    print("=" * 75)
    
    # open the csv file as read only
    devices = get_devices(CSV_FILE)
    
    for device in devices:
        # Skip only true non-manageable devices
        if device["ip"] in ["None"] or device["username"].lower() == "none":
            print(f"{device["name"]:12} | Skipped (no management access)")
            continue
    
        # ping the device to see if it is reachable
        reachable = ping_device(device["ip"])
    
        # if it is not reachable add to the unreachable list
        if not reachable:
            print(f"{device["name"]:12} | {device["ip"]:15} | UNREACHABLE")
            continue
    
        # grab ssh in
        current_dns = get_current_dns(device["ip"], device["username"], device["password"])
    
        # checks to see if the dns configuration matches what is expected
        if current_dns == EXPECTED_DNS:
            status = "OK - DNS correct"
        else:
            status = f"COMPROMISED - DNS is {current_dns}"
    
        print(f"{device["name"]:12} | {device["ip"]:15} | {status}")
    
    print("=" * 75)
    print("Scan complete.")
    
    # Ends B2 code block

    ```

  </details> <!-- Ends the actual script -->

  ---
      
  </details> <!-- Ends Scrip 2 -->


<details> <!-- Starts Script 3 -->
<summary><strong>Script 3 - Send an Alert Email to Stakeholders</strong></summary>

## Script 3 - Send an Alert Email to Stakeholders

**Learning Objectives**
- Send an automated email using Python
- Include device details (hostname, IP address, and service) in the message
- Use a predefined email template for incident notification

**Pseudocode**
1. Import required modules
2. Define email settings (sender, recipients, subject)
3. Create the email message using the incident alert template
4. Insert the compromised device details (hostname, IP, service)
5. Connect to the email server
6. Send the alert email
7. Confirm the email was sent successfully

<details> <!-- Starts Actual Script -->
<summary><strong>Actual Script</strong></summary>

  ```Python
      # send_incident_alert.py
      # This script sends an incident alert email to stakeholders about compromised devices.
      
      import smtplib
      from email.mime.text import MIMEText
      from datetime import datetime
      
      # Email settings for the lab
      SMTP_SERVER = "smtp.d522.wgu.internal"
      SMTP_PORT = 1025
      SENDER = "network-monitor@d522.wgu.internal"
      RECIPIENT = "stakeholders@d522.wgu.internal"
      
      # Example compromised device 
      device_name = "DNS1"
      ip_address = "10.10.10.10"
      timestamp = datetime.now().strftime("%Y-%m-%d %H:%M:%S")
      
      # Build the email using the required template
      subject = "URGENT: Device Compromise Detected—Immediate Attention Required"
      
      body = f"""Dear Stakeholders,
      
      This is an automated alert to inform you that the following device(s) have been identified as compromised during the recent network scan:
      
      Device Name: {device_name}
      IP Address: {ip_address}
      Last Checked: {timestamp}
      
      Immediate investigation and remediation are recommended to prevent further impact.
      
      If you have any questions or require additional information, please contact the IT support team.
      
      Best regards,
      Network Monitoring System
      """
      
      # Create the email message
      message = MIMEText(body)
      message["Subject"] = subject
      message["From"] = SENDER
      message["To"] = RECIPIENT
      
      # Send the email
      try:
          with smtplib.SMTP(SMTP_SERVER, SMTP_PORT) as server:
              server.send_message(message)
          print("Incident alert email sent successfully!")
      except Exception as e:
          print(f"Failed to send email: {e}")
  
  ```


</details> <!-- Ends Actual Script -->

---
  
</details> <!-- Ends Script 3 -->


<details> <!-- Starts Script 4 -->
<summary><strong>Script 4 - Create Incident Tickets</strong></summary>

## Script 4 - Create Incident Tickets

**Learning Objectives**
- Connect to a web service using Python
- Automatically create ticket entries for affected devices
- Include the issue type and targeted device information in each ticket

**Pseudocode**
1. Import required modules
2. Define the web service connection details
3. Open and read the list of affected devices
4. For each affected device:
   - Create a ticket payload with the issue type and device details
   - Send the request to the web service
   - Confirm the ticket was created successfully
5. Display a summary of created tickets

<details> <!-- Starts Actual Script -->
<summary><strong>Actual Script</strong></summary>

  ```Python
    # create_tickets.py
    # This script creates a helpdesk ticket for each device from the network inventory.
    # C2
    
    import csv
    import requests
    from pathlib import Path
    
    # API details from the lab
    API_URL = "http://api.d522.wgu.internal:5000/api/tickets"
    TOKEN = "<REDACTED - lab only>"
    
    headers = {
        "Authorization": f"Bearer {TOKEN}",
        "Content-Type": "application/json"
    }
    
    csv_file = Path("network_devices.csv")
    
    print("Creating tickets...")
    print("=" * 60)
    
    with open(csv_file, mode="r") as file:
        reader = csv.DictReader(file)
    
        for row in reader:
            device_name = row["Device Name"]
            ip_address = row["Device Address"]
    
            # Skip devices without a usable IP
            if ip_address in ["None", "DHCP"]:
                continue
    
            # Create Ticket Data
            ticket_data = {
                "title": f"DNS Issue - {device_name}",
                "description": f"Device {device_name} ({ip_address}) may have unauthorized DNS settings.",
                "priority": "high",
                "status": "open"
            }
    
            try:
                response = requests.post(API_URL, json=ticket_data, headers=headers, timeout=10)
    
                if response.status_code in [200, 201]:
                    print(f"✓ Ticket created for {device_name}")
                    print(f"  {response.json()}")
                else:
                    print(f"✗ Failed for {device_name} - Status Code: {response.status_code}")
                    print(f"  {response.text}")
    
            except Exception as e:
                print(f"✗ Error creating ticket for {device_name}: {e}")
    
    print("=" * 60)
    print("Ticket creation process complete.")
  
  ```

</details> <!-- Ends Actual Script -->

---
  
</details> <!-- Ends Script 4 -->


<details> <!-- Starts Script 5 -->
<summary><strong>Script 5 - Verify and Restart DNS Service</strong></summary>

## Script 5 - Verify and Restart DNS Service

**Learning Objectives**
- Connect to an internal DNS server
- Check the current status of the DNS service
- Restart the DNS service if it is down
- Confirm the service is running after the restart

**Pseudocode**
1. Import required modules
2. Define connection details for the DNS server
3. Connect to the DNS server
4. Check the current status of the DNS service
5. If the service is down:
   - Restart the DNS service
6. Verify the service is running
7. Print the status before and after the restart

<details> <!-- Starts Actual Script -->
<summary><strong>Actual Script</strong></summary>

  ```python
  # d1_d2_manage_dns_service_total.py
  # This script will complete the tasks of D1 & D2
  
  
  import os                   # Because I like a clean screen
  import paramiko             # Used for SSH
  import time					# Used for time stamping
  from pathlib import Path    # Used to get the location of the csv file
  from utility_get_devices import get_devices # My baby
  
  
  # Path to the csv file
  CSV_FILE = Path("/home/student/d522/network_devices.csv")
  
  
  # defines the ssh function
  def run_cmd(ssh, command):
      stdin, stdout, stderr = ssh.exec_command(command)
      return stdout.read().decode().strip()
  
  
  # ===== Report Header =====
  os.system("clear")
  print("")
  print("=" * 50)
  print("DNS check and restart initiated.")
  print("=" * 50)
  print()
  
  
  # Main
  
  devices = get_devices(CSV_FILE)
  
  for device in devices:
      
      # Check to see if the device is one of the DNS servers
      if device["name"] in ["DNS1", "DNS2"]:
          try:
              
              # If it is, we need to ssh into the device
              ssh = paramiko.SSHClient()
              ssh.set_missing_host_key_policy(paramiko.AutoAddPolicy())
              ssh.connect(device["ip"], username=device["username"], password=device["password"])
              print(f"Connected to {device["name"]}")
  
  
              # Check status
              print(f"Checking status of {device["name"]}...")
              device_status = run_cmd(ssh, "systemctl is-active systemd-resolved || systemctl is-active named || systemctl is-active bind9")
              print(f"{device["name"]}'s status: {device_status}")
  
  
              # If the status is anything other than active
              if device_status != "active":
                  # We need to start the service back up
                  print(f"\n=== Restarting DNS service on {device["name"]} ===")
                  run_cmd(ssh, "sudo systemctl restart systemd-resolved || sudo systemctl restart named || sudo systemctl restart bind9")
                  time.sleep(2)
  
                  
                  # Recheck status
                  device_status = run_cmd(ssh, "systemctl is-active systemd-resolved || systemctl is-active named || systemctl is-active bind9")
                  print(f"{device["name"]}'s status after restart: {device_status}")
  
                  
                  # Options... what do we do?
                  if device_status == "active":
                      print(f"DNS service on {device["name"]} is now UP.")
                  else:
                      print(f"Warning: DNS service may still be down.")
              else:
                  print(f"{device["name"]} is already running.")
  
  
              # Closs SSH connection
              ssh.close()
              print("Connection closed.\n")
  
          except Exception as e:
              print(f"Failed to connect to {device["name"]}: {e}\n")
  
  
  # ===== Report Footer =====
  print("=" * 50)
  print("DNS check and restart completed.")
  print("=" * 50)
  print()
  
  
  
  
  # End of script
  
  
  ```

<img width="462" height="514" alt="D1_D2_manage_dns_service" src="https://github.com/user-attachments/assets/93edffdd-3538-4d05-994a-3d85a61471fd" />


---

</details> <!-- Ends Actual Script -->

---
  
</details> <!-- Ends Script 5 -->

<details> <!-- Starts Script 6 -->
<summary><strong>Script 6 - Correct DNS Settings on Affected Devices</strong></summary>

## Script 6 - Correct DNS Settings on Affected Devices

**Learning Objectives**
- Connect to multiple network devices using SSH
- Update DNS configuration settings on each device
- Confirm the new DNS settings were applied successfully

**Pseudocode**
1. Import required modules
2. Define the correct DNS server address
3. Open and read the list of affected devices
4. For each affected device:
   - Establish an SSH connection
   - Update the DNS configuration
   - Verify the new setting was applied
   - Close the connection
5. Print a status message for each device

<details> <!-- Starts Actual Script -->
<summary><strong>Actual Script</strong></summary>

  ```Python
  # fix_dns_settings.py
  # This script connects to each device and sets the correct DNS server.
  # D4
  
  import os                           # Because I like my clean screen
  import paramiko
  from pathlib import Path
  from utility_get_devices import get_devices     # My new baby
  
  # Correct DNS server for the lab
  CORRECT_DNS = "10.10.10.10"
  
  CSV_FILE = Path("/home/student/d522/network_devices.csv")
  
  # Clean up and set up
  os.system("clear")
  print("Fixing DNS settings on devices...")
  print("=" * 60)
  
  devices = get_devices(CSV_FILE)
  
  for device in devices:
      # Skip devices without a usable IP or credentials
      if device["ip"] in ["None", "DHCP"] or device["username"].lower() == "none":
          print(f"{device['name']:12} | Skipped (no usable IP or credentials)")
          continue
  
      print(f"\nConnecting to {device['name']} ({device['ip']})...")
  
      try:
          ssh = paramiko.SSHClient()
          ssh.set_missing_host_key_policy(paramiko.AutoAddPolicy())
          ssh.connect(
              device["ip"],
              username=device["username"],
              password=device["password"],
              timeout=5
          )
  
          commands = [
              f"echo 'nameserver {CORRECT_DNS}' | sudo tee /etc/resolv.conf",
              "cat /etc/resolv.conf"
          ]
  
          for cmd in commands:
              stdin, stdout, stderr = ssh.exec_command(cmd)
              output = stdout.read().decode().strip()
              if output:
                  print(output)
  
          print(f"{device['name']:12} | DNS settings updated successfully")
          ssh.close()
  
      except Exception as e:
          print(f"{device['name']:12} | Failed - {e}")
  
  print("\n" + "=" * 60)
  print("DNS remediation complete.")
  
  ```

</details> <!-- Ends Actual Script -->

---
  
</details> <!-- Ends Script 6 -->


<details> <!-- Starts Script 7 -->
<summary><strong>Script 7 - Send Resolution Notification Email</strong></summary>

## Script 7 - Send Resolution Notification Email

**Learning Objectives**
- Send a resolution notification email to stakeholders
- Include a list of all affected devices (hostname and IP address)
- Confirm successful delivery of the notification

**Pseudocode**
1. Import required modules
2. Define email settings (sender, recipients, subject)
3. Create the email message using the resolution template
4. Insert the list of affected devices (hostname and IP)
5. Connect to the email server
6. Send the resolution email
7. Confirm the email was sent successfully

<details> <!-- Starts Actual Script -->
<summary><strong>Actual Script</strong></summary>

  ```Python
  # send_resolution_notification.py
  # This script sends a notification of all devices fixed
  # E
  
  import smtplib
  from email.mime.text import MIMEText
  from datetime import datetime
  
  # Email settings for the lab
  SMTP_SERVER = "smtp.d522.wgu.internal"
  SMTP_PORT = 1025
  SENDER = "network-monitor@d522.wgu.internal"
  RECIPIENT = "stakeholders@d522.wgu.internal"
  
  
  # Email Template
  subject = "RESOLVED: DNS Service Issue and Device Compromise-All Issues Remediated"
  
  body = f"""Dear Stakeholders,
  
  This is an automated notification to inform you that the DNS service issue and all related device compromises have been successfully resolved. The following devices were affected and have now been remediated:
  
  API
  DB
  DNS1
  DNS2
  ROUTER1
  SVR1
  SVR2
  
  No further action is required at this time. if you have any questions or concerns, please contact the IT support team.
  
  Thank you for your attention.
  
  Best regards,
  Network Monitoring System
  """
  
  # Create the email message
  message = MIMEText(body)
  message["Subject"] = subject
  message["From"] = SENDER
  message["To"] = RECIPIENT
  
  # Send the email
  try:
      with smtplib.SMTP(SMTP_SERVER, SMTP_PORT) as server: 
          server.send_message(message)
      print("Resolution email sent successfully")
  except Exception as e:
      print(f"Failed to send email: {e}")
  
  
  ```

</details> <!-- Ends Actual Script -->

---

</details> <!-- Ends Script 7 -->

<details>
<summary><strong>Bonus Script (Utility)</strong></summary>

  This script was created as a utility script to handle repetitive functions.

  ```python
  # utility_get_devices.py
  """
  This script will just grab all the devices from the csv
  add the hard coded IP address for PC1-PC4
  Then, return an array of devices.
  """
  
  # ---------------------------------------
  # Imports
  # ---------------------------------------
  import csv                  # Used to read the csv file
  from pathlib import Path    # Used to get the location of the csv file
  
  
  # ---------------------------------------
  # Constants
  # ---------------------------------------
  CSV_FILE = Path("network_devices.csv")
  
  
  # ---------------------------------------
  # Functions
  # ---------------------------------------
  def get_devices(filepath):
      # Known DHCP addresses discovered from the lab
      dhcp_ips = {
          "PC1": "192.168.10.102",
          "PC2": "192.168.20.101",
          "PC3": "192.168.30.101",
          "PC4": "192.168.10.101"
      }    
      
      # Creates the variable to hold all the devices 
      devices = []
      
      # Load device information from the csv file
      with open(filepath, mode="r") as file:
          reader = csv.DictReader(file)
          
          # Iterate through each row grabbing the information needed to ping / ssh
          for row in reader:
              name = row["Device Name"]
              ip = row["Device Address"]
              username = row["Username"]
              password = row["Password"]
              
              # Checks to see if the device is on the list of DHCP addresses
              if ip.upper() == "DHCP" and name in dhcp_ips:
                  ip = dhcp_ips[name]
          
              devices.append({
                  "name": name,
                  "ip": ip,
                  "username": username,
                  "password": password
              })
      return devices  # Sends this array back to the caller.
  
  
  # End of utility_get_devices

  ```
</details>

</details> <!-- Ends Task 1 -->

---
























<!-- 
****************************************************************
******************** This ends Task 1 Notes ********************
****************************************************************
-->




























<details>
<summary><strong>Task 2: Proactive Monitoring and Prevention</strong></summary>
  
## Task 2: Proactive Monitoring and Prevention

### Introduction   
In this task, you build continuous monitoring and automated response for the same lab network. Scripts detect unreachable devices, open tickets, watch for unauthorized DNS changes, remediate configurations, notify stakeholders, and log healthy DNS state.

### Scenario  
After the initial incident response, the organization needs ongoing visibility. Devices may drop offline, DHCP clients may appear, and DNS settings may be altered again. Automation must detect these conditions, notify the right people, create or update tickets, and restore expected DNS configuration where possible.

### Problem Statement  

**Design and implement solutions that:**
- Back up DNS server configuration before changes
- Enumerate devices from inventory (including resolved DHCP addresses)
- Continuously check reachability and notify when devices are unavailable
- Create helpdesk tickets for unavailable devices
- Detect altered DNS, alert, remediate, and mark tickets resolved
- Log when DNS is correct and the DNS-related service is active

---

<details>
<summary><strong>Script 1 - Back up DNS server</strong></summary>

## Script 1 - Backup DNS server

  **Learning Objectives:**
  - Create a directory to save the backup
  - Use SSH to connect to DNS servers and access the config file
  - Save the config file to the backup directory named appropriately

  **Pseudocode**
  1) Create a directory for the backup files to be saved
  2) Obtain DNS IP addresses, usernames, and passwords
  3) Open an SSH connection to the each server in turn
  4) Copy the config file
  5) Save it with a .bak extension
  6) Close SSH connection (repeat steps 2 - 6 for second server)


<details>
<summary><strong>Actual Script</strong></summary>
  
  ```python
  # b1_backup_dns_config.py
  # This script is going to be used to back up the current settings on the two dns servers.
  """ B.  Before making any changes to the devices on your network, write a Python script 
          to create a backup directory of the DNS server device configuration files in the 
          attached "network_devices" CSV file by copying the device configuration and creating 
          a subdirectory for both DNS servers in the network. Within each subdirectory, create a 
          file with the DNS record configuration for the server, including all of the following 
          folders for both DNS servers:
          •   DNS-Backup
          •  Server-1
              record-config.txt
          •  Server-2
              record-config.txt
  """
  
  
  # ---------------------------------------
  # Imports
  # ---------------------------------------
  from pathlib import Path            # Need to get the path to the network_devices.csv file
  from datetime import datetime       # For time stamping entries
  from network_utility import (         # My utilities
      open_ssh,
      get_devices,
      ping_device,
      clear_screen
  )
  
  
  # ---------------------------------------
  # Constants Delcared
  # ---------------------------------------
  CSV_FILE = Path("network_devices.csv")          # To locate and access the csv file containing the information needed. 
  BACKUP_ROOT = Path("DNS-Backup")                # Backup destination
  EXPECTED_DNS_SERVERS = ["DNS1", "DNS2"]         # Expected DNS servers 
  
  
  # ---------------------------------------
  # Functions
  # ---------------------------------------
  
  # Create the backup folder 
  def create_backup_folders():
      """Create the required folder structure"""
      BACKUP_ROOT.mkdir(exist_ok=True)
      (BACKUP_ROOT / "Server-1").mkdir(exist_ok=True)
      (BACKUP_ROOT / "Server-2").mkdir(exist_ok=True)
      print("Backup folders created successfully.")
      print(f"  → {BACKUP_ROOT}/Server-1/")
      print(f"  → {BACKUP_ROOT}/Server-2/\n")
  
  # Grab DNS configuration
  def get_dns_config(ip, username, password):
      """SSH into the device and pull the DNS configuration"""
      try:
          ssh = open_ssh(ip, username, password)
  
          # Try the most common BIND config file first
          stdin, stdout, stderr = ssh.exec_command(
              "cat /etc/bind/named.conf 2>/dev/null || "
              "cat /etc/bind/named.conf.local 2>/dev/null || "
              "cat /etc/resolv.conf")
          config_data = stdout.read().decode().strip()
          ssh.close()
  
          return config_data if config_data else "No configuration data retrieved."
      except Exception as e:
          return f"Error retrieving config: {e}"
  
  # Save the configuration
  def save_config(server_folder, config_data, device_name):
      """Save the configuration to record-config.txt"""
      file_path = BACKUP_ROOT / server_folder / "record-config.txt"
      
      with open(file_path, "w") as f:
          f.write(f"# Backup of {device_name}\n")
          f.write(f"# Created: {datetime.now().strftime('%Y-%m-%d %H:%M:%S')}\n")
          f.write("#" + "="*50 + "\n\n")
          f.write(config_data)
      
      print(f"✓ Configuration backed up → {file_path}")
  
  # Defining the backup process
  def backup_dns_server():
      # Step 1 - Create folder structure
      print("Creating backup folders")
      create_backup_folders()
  
  
      # Step 2 - Read csv file and find the DNS servers information
      devices = get_devices(CSV_FILE)
  
      # ---------------------------------------
      # These steps will be in a loop as there 
      # are more than one DNS server
      # ---------------------------------------
      for device in devices:
      # Step 3 - Test to see if we have a DNS server and if it is reachable 
          if device['name'] not in EXPECTED_DNS_SERVERS:
              # print(f"{device['name']} not a dns server") # Was used for testing
              continue
              
          print(f"Processing {device['name']} ({device['ip']})...")
          
          # Checks if device is reachable
          if not ping_device(device['ip']):
              print(f"{device['name']} is unreachable.")
              continue
          
      # Step 4 - Set save folder location
          if device['name'] == "DNS1":
              server_folder = "Server-1"
          else:
              server_folder = "Server-2"
  
      # Step 5 - Getting configuration file
          print(f"Connecting to {device['name']}: {device['ip']} and retrieving config.")
          config = get_dns_config(device['ip'], device['username'], device['password'])
  
      # Step 6 - Backing up file       
          print(f"Backing up {device['name']} to {server_folder}")
          save_config(server_folder, config, device['name'])
          print()
  
  # Main Script as a function
  def main():
      # A little formatting
      clear_screen()
      print("=" * 60)
      print("DNS Configuration Backup Script")
      print("=" * 60)
      print()
      
      backup_dns_server()
              
      # Step 7 - Goodbye
      print("=" * 60)
      print("Backup process completed. ~ Goodbye")
      print("=" * 60)
  
  
  
  # ---------------------------------------
  # Main Script
  # ---------------------------------------
  if __name__ == "__main__":
      main()
  
  
  
  
  # ---------------------------------------
  # End of Script
  # ---------------------------------------

```

</details> <!-- Ends Actual Script -->

</details> <!-- Ends back up DNS -->

---

<details> 
<summary><strong>Script 2 - Read CSV File</strong></summary>

## Script 2 - Read CSV File

**Learning Objectives:**
- Open and read from a .csv file
- Dynamically add IP addresses for DHCP devices (PC1-PC4)

**Pseudocode**
1) Open and read .csv file
2) SSH into the router
3) Download IP table
4) Parse IP table for PC1-PC4 devices
5) Add IP address to the list

<details>
<summary><strong>Actual Script</strong></summary>

```python
  # c1_device_list.py
  """
  C.  Implement ongoing monitoring and automated response for network device availability and DNS IP configuration by completing the following steps:
  	1.  Write a Python script to read a CSV file containing network device information. 
  """
  
  # ---------------------------------------
  # Imports
  # ---------------------------------------
  from pathlib import Path            # Need to get the path to the network_devices.csv file
  from network_utility import (         # My script
      get_devices,
      clear_screen
      )    
  
  # ---------------------------------------
  # Constants Delcared
  # ---------------------------------------
  CSV_FILE = Path("/home/student/d522/network_devices.csv")          # To locate and access the csv file containing the information needed.  
  
  
  # ---------------------------------------
  # Definitions
  # ---------------------------------------
  # The purpose of this script, print devices
  def print_network_devices():
      network_devices = get_devices(CSV_FILE)
      for device in network_devices:
          print(f"{device['name']:13} {device['ip']}")
  
  
  
  # ---------------------------------------
  # Main script
  # ---------------------------------------
  if __name__ == "__main__":
      clear_screen()
      print("=" * 60)
      print("Network Devices")
      print("=" * 60)
  
      print_network_devices()
  
      print("=" * 60)
      print("\nTask completed. ~ Goodbye")
      print("=" * 60)
  
  
  
  # ---------------------------------------
  # End of script
  # ---------------------------------------
```

</details> <!-- Ends Actual Script -->

</details> <!-- Ends Read CSV scrip -->

---

<details>
<summary><strong>Script 3 - Check Device Availability</strong></summary>

  ## Script 3 - Check Device Abailability

  **Learning Objectives:**
  - Continuously poll devices from inventory
  - Use ICMP to determine reachability
  - Send templated email when a device is unavailable

**Pseudocode**
1) Import modules and load device list
2) Define email settings and check interval
3) Loop until interrupted:
   - For each device with a usable IP, ping
   - If unreachable, send notification email
   - Sleep for the configured interval
4) On Ctrl_C, print how many scans completed

<details>
<summary><strong>Actual Script</strong></summary>
  
  ```python
  # c2_device_availability.py
  
  """
  Write a Python script to verify device availability (e.g., using ping/ICMP, etc.) 
  and automatically send an email notification to stakeholders when a device is unavailable, 
  using the "Device Unavailable Notification" template from the "Task 2 Email Templates" 
  supporting document. 
  
  *** Okay, I had full intent import my c1 script and grab the devices that way. ***
  *** I just wasn't sure if that was allowed. ***
  
  """
  
  # ---------------------------------------
  # Imports
  # ---------------------------------------
  import time                                 # To use sleep
  import smtplib                              # For email purposes
  from email.mime.text import MIMEText        # Building our email
  from pathlib import Path                    # For using the path to the csv file
  from datetime import datetime               # Always want some time stamps
  from network_utility import (                 # My utilities
      get_devices, 
      ping_device, 
      clear_screen
  )
  
  
  # ---------------------------------------
  # Constants Declaired
  # ---------------------------------------
  CSV_FILE = Path("/home/student/d522/network_devices.csv")    # The file we need
  SMTP_SERVER = "smtp.d522.wgu.internal"              # Email server
  SMTP_PORT = 1025                                    # Port number needed
  SENDER = "network-monitor@d522.wgu.internal"        # Email address
  RECIPIENT = "admin@d522.wgu.internal"               # Admin address
  DURATION = 3                                        # How long we are checking
  INTERVAL = 30                                       # How often 
  
  # ---------------------------------------
  # Functions
  # ---------------------------------------
  
  # Build up the email to be sent
  def send_unavailable_email(device_name, ip_address):
      """Send the Device Unavailable Notification email"""
      timestamp = datetime.now().strftime("%Y-%m-%d %H:%M:%S")
  
      subject = f"Network Device Unavailable: {device_name} ({ip_address})"
  
      body = f"""Dear Network Administrator,
  
  This is an automated notification that the following network device is currently unavailable:
  
  Device Name: {device_name}
  IP Address: {ip_address}
  Last Checked: {timestamp}
  
  Please investigate this issue at your earliest convenience.
  
  Best regards,
  Network Monitoring System
  """
  
      message = MIMEText(body)
      message["Subject"] = subject
      message["From"] = SENDER
      message["To"] = RECIPIENT
  
      try:
          with smtplib.SMTP(SMTP_SERVER, SMTP_PORT) as server:
              server.send_message(message)
          print(f" ✓ Email sent for {device_name}")
          return True
      except Exception as e:
          print(f" ✗ Failed to send email for {device_name}: {e}")
          return False
  
  
  def check_device_availability(interval_seconds=INTERVAL):
      """Continuously monitor device availability until stopped (Ctrl+C)"""
      scan_count = 0
  
      try:
          while True:
              scan_count += 1
              print(f"\n=== Scan #{scan_count} at {datetime.now().strftime('%Y-%m-%d %H:%M:%S')} ===")
  
              devices = get_devices(CSV_FILE)
  
              for device in devices:
                  name = device["name"]
                  ip = device["ip"]
  
                  if ip in ["None", "DHCP"]:
                      print(f"{name:12} | Skipped (no usable IP)")
                      continue
  
                  print(f"Checking {name:12} ({ip})...", end=" ")
  
                  if ping_device(ip):
                      print("REACHABLE")
                  else:
                      print("UNAVAILABLE → Sending notification...")
                      send_unavailable_email(name, ip)
  
              print(f"\nSleeping {interval_seconds} seconds... (Ctrl+C to stop)")
              time.sleep(interval_seconds)
  
      except KeyboardInterrupt:
          print(f"\n\nMonitoring stopped by user after {scan_count} scans.")
      
  
  # ---------------------------------------
  # Main
  # ---------------------------------------
  if __name__ == "__main__":
      # Beautification
      clear_screen()
      print("=" * 60)
      print("Device Availability Monitor")
      print("=" * 60)
  
      check_device_availability()
  
      print("=" * 60)
      print("Monitoring complete. ~ Goodbye")
      print("=" * 60)
  
  
  # ---------------------------------------
  # End of script
  # ---------------------------------------

```

</details> <!-- Ends Actual Script -->


</details> <!-- Ends Availability script -->

---

<details> 
<summary><strong>Script 4 - Help Ticket</strong></summary>

## Script 4 - Help Ticket

**Learning Objectives:**
- Generate a help ticket
- API POST
- Skip devices without IPs

**Pseudocode**
1) Import modules and load device list
2) Define help ticket settings and check interval
3) Loop until interrupted:
   - For each device with a usable IP, ping
   - If unreachable, generate help ticket
   - Sleep for the configured interval
4) On Ctrl_C, print how many scans completed


<details>
<summary><strong>Actual Script</strong></summary>

```python
  # c3_ticket_generator.py
  """
  Write a Python script to automatically create a ticket in the web service
  for each unavailable device, indicating the type of issue and the unavailable device.
  """
  
  import time
  import requests
  from datetime import datetime
  from pathlib import Path
  from network_utility import (
      get_devices,
      ping_device,
      clear_screen
  )
  
  # ---------------------------------------
  # Constants
  # ---------------------------------------
  CSV_FILE = Path("/home/student/d522/network_devices.csv")
  API_URL = "http://api.d522.wgu.internal:5000/api/tickets"
  TOKEN = "<REDACTED - lab only>"
  INTERVAL = 30
  
  HEADERS = {
      "Authorization": f"Bearer {TOKEN}",
      "Content-Type": "application/json"
  }
  
  # ---------------------------------------
  # Functions
  # ---------------------------------------
  def create_ticket(device_name, ip_address):
      """Create a helpdesk ticket for an unavailable device"""
      ticket_data = {
          "title": f"Device Unavailable - {device_name}",
          "description": (
              f"Device {device_name} ({ip_address}) is not responding to ping. "
              f"Detected at {datetime.now().strftime('%Y-%m-%d %H:%M:%S')}."
          ),
          "priority": "high",
          "status": "open"
      }
  
      try:
          response = requests.post(API_URL, json=ticket_data, headers=HEADERS, timeout=10)
  
          if response.status_code in [200, 201]:
              print(f"  ✓ Ticket created for {device_name}")
              print(f"    Response: {response.json()}")
              return True
          else:
              print(f"  ✗ Failed for {device_name} - Status {response.status_code}")
              print(f"    {response.text}")
              return False
      except Exception as e:
          print(f"  ✗ Error creating ticket for {device_name}: {e}")
          return False
  
  
  def generate_ticket(interval_seconds=INTERVAL):
      """Continuously monitor devices and create tickets for unavailable ones"""
      scan_count = 0
  
      try:
          while True:
              scan_count += 1
              print(f"\n=== Scan #{scan_count} at {datetime.now().strftime('%Y-%m-%d %H:%M:%S')} ===")
  
              devices = get_devices(CSV_FILE)
  
              for device in devices:
                  name = device["name"]
                  ip = device["ip"]
  
                  if ip in ["None", "DHCP"]:
                      print(f"{name:12} | Skipped (no usable IP)")
                      continue
  
                  print(f"Checking {name:12} ({ip})...", end=" ")
  
                  if ping_device(ip):
                      print("REACHABLE")
                  else:
                      print("UNAVAILABLE → Creating ticket...")
                      create_ticket(name, ip)
  
              print(f"\nSleeping {interval_seconds} seconds... (Ctrl+C to stop)")
              time.sleep(interval_seconds)
  
      except KeyboardInterrupt:
          print(f"\n\nTicket monitoring stopped by user after {scan_count} scans.")
  
  
  # ---------------------------------------
  # Main
  # ---------------------------------------
  if __name__ == "__main__":
      clear_screen()
      print("=" * 60)
      print("Create Tickets for Unavailable Devices")
      print("=" * 60)
      print()
  
      generate_ticket()
  
      print()
      print("=" * 60)
      print("Ticket creation process completed. ~ Goodbye")
      print("=" * 60)
      
      # ---------------------------------------
      # End of Script
      # ---------------------------------------

```

</details> <!-- Ends Actual Script -->

</details> <!-- Ends Ticket Generator -->

---

<details> 
<summary><strong>Script 5 - Email Alert</strong></summary>

## Script 5 - Email Alert

**Learning Objectives:**
- Checking DNS settings
- Checking for unauthorized changes to DNS settings
- Sending an email alert if settings have changed
- Correct DNS settings on the affected devices

**Pseudocode**
1) Import modules and load device list
2) Define email settings and check interval
3) Loop until interrupted:
   - For each device with a usable IP
     - SSH into the device
     - Compare DNS settings to expected settings
     - Send notification email regarding alter DNS devices
     - Set device to correct DNS setting
     - Close SSH connection
4) Sleep for the configured interval
5) On Ctrl_C, print how many scans completed


<details>
<summary><strong>Actual Script</strong></summary>

```python
  # c4_altered_dns_email.py
  """
  4.  Write a Python script to automatically complete the following tasks:
  •  Use the "DNS Setting Altered Notification" template from the attached 
     "Task 2 Email Templates" to send an email notification to stakeholders 
     when a DNS setting is altered.
  •  Correct the DNS setting.
  •  Update the ticket in the web service to show the DNS issue has been resolved.
  """
  
  # ---------------------------------------
  # Imports
  # ---------------------------------------
  import requests                         # Access web services
  import smtplib                          # Email stuff
  import time                             # For setting delay
  from email.mime.text import MIMEText    # For the body
  from pathlib import Path                # Find files 
  from datetime import datetime           # For time stamping things
  from network_utility import (           # My utility script
      get_devices,
      open_ssh,
      clear_screen
  )
  
  
  # ---------------------------------------
  # Constants
  # ---------------------------------------
  CSV_FILE = Path("/home/student/d522/network_devices.csv")
  PRIMARY_DNS = "10.10.10.10"
  SECONDARY_DNS = "10.10.10.20"
  AUTHORIZED_DNS = {PRIMARY_DNS, SECONDARY_DNS}
  INTERVAL = 30
  
  
  SMTP_SERVER = "smtp.d522.wgu.internal"
  SMTP_PORT = 1025
  SENDER = "network-monitor@d522.wgu.internal"
  RECIPIENT = "admin@d522.wgu.internal"
  
  API_URL = "http://api.d522.wgu.internal:5000/api/tickets"
  TOKEN = "<REDACTED - lab only>"
  HEADERS = {
      "Authorization": f"Bearer {TOKEN}",
      "Content-Type": "application/json"
  }
  
  
  
  # ---------------------------------------
  # Functions
  # ---------------------------------------
  # Gets the current DNS settings
  def get_current_dns(ip, username, password):
      """
      SSH in and determine the effective upstream DNS.
      If resolv.conf shows 127.0.0.53 (systemd-resolved), query the real upstream.
      """
      try:
          ssh = open_ssh(ip, username, password)
  
          # 1. Read resolv.conf
          stdin, stdout, stderr = ssh.exec_command("cat /etc/resolv.conf")
          resolv = stdout.read().decode()
  
          nameservers = []
          for line in resolv.splitlines():
              line = line.strip()
              if line.lower().startswith("nameserver"):
                  parts = line.split()
                  if len(parts) >= 2:
                      nameservers.append(parts[1])
  
          if not nameservers:
              ssh.close()
              return "No DNS found"
  
          # 2. If we only see the local stub, ask systemd-resolved for the real upstream
          if all(ns in ("127.0.0.53", "127.0.0.1") for ns in nameservers):
              # Try resolvectl first, then systemd-resolve
              cmd = (
                  "resolvectl status 2>/dev/null || "
                  "systemd-resolve --status 2>/dev/null || true"
              )
              stdin, stdout, stderr = ssh.exec_command(cmd)
              status = stdout.read().decode()
  
              upstream = []
              for line in status.splitlines():
                  line = line.strip()
                  # Common lines: "DNS Servers: 10.10.10.10" or "Current DNS Server: ..."
                  if "DNS Server" in line or "DNS Servers" in line:
                      # grab IPv4-looking tokens
                      for token in line.replace(":", " ").split():
                          if token.count(".") == 3 and not token.startswith("127."):
                              upstream.append(token)
  
              ssh.close()
  
              if upstream:
                  # Return the first real upstream address
                  return upstream[0]
              # Fall back to what resolv.conf said
              return nameservers[0]
  
          # 3. Normal case: return first non-stub nameserver (or first entry)
          for ns in nameservers:
              if ns not in ("127.0.0.53", "127.0.0.1"):
                  ssh.close()
                  return ns
  
          ssh.close()
          return nameservers[0]
  
      except Exception as e:
          return f"Error: {e}"
  
  # Builds and sends an email regarding altered DNS settings found
  def send_dns_altered_email(device_name, ip, current_dns):
      """Send the DNS Setting Altered Notification"""
      timestamp = datetime.now().strftime("%Y-%m-%d %H:%M:%S")
  
      subject = f"DNS Configuration Alert: {device_name} ({ip})"
  
      body = f"""Dear Network Administrator,
  
  This is an automated alert that the DNS configuration for the following device has been altered from the expected settings:
  
  Device Name: {device_name}
  IP Address: {ip}
  Detected DNS Setting: {current_dns}
  Expected DNS Setting: {PRIMARY_DNS}
  Time Detected: {timestamp}
  
  The system will attempt to automatically correct this configuration.
  
  Best regards,
  Network Monitoring System
  """
  
      message = MIMEText(body)
      message["Subject"] = subject
      message["From"] = SENDER
      message["To"] = RECIPIENT
  
      try:
          with smtplib.SMTP(SMTP_SERVER, SMTP_PORT) as server:
              server.send_message(message)
          print(f"  ✓ Alert email sent for {device_name}")
          return True
      except Exception as e:
          print(f"  ✗ Failed to send email: {e}")
          return False
  
  # Fixes altered DNS records
  def correct_dns(ip, username, password):
      """Restore expected DNS resolver configuration and service state"""
      try:
          ssh = open_ssh(ip, username, password)
  
          commands = [
              # Create drop-in dir first
              "sudo mkdir -p /etc/systemd/resolved.conf.d",
              # Point resolved at authorized DNS servers
              f"echo -e '[Resolve]\\nDNS={PRIMARY_DNS} {SECONDARY_DNS}\\nFallbackDNS=\\n' | sudo tee /etc/systemd/resolved.conf.d/99-lab-dns.conf >/dev/null",
              # Restart resolver
              "sudo systemctl restart systemd-resolved 2>/dev/null || true",
              # Fallback resolv.conf for hosts not using resolved
              f"echo -e 'nameserver {PRIMARY_DNS}\\nnameserver {SECONDARY_DNS}' | sudo tee /etc/resolv.conf >/dev/null",
              # Restart common DNS services if present
              "sudo systemctl restart systemd-resolved 2>/dev/null || "
              "sudo systemctl restart named 2>/dev/null || "
              "sudo systemctl restart bind9 2>/dev/null || true",
          ]
  
          for cmd in commands:
              ssh.exec_command(cmd)
  
          time.sleep(1)
          stdin, stdout, stderr = ssh.exec_command(
              "resolvectl status 2>/dev/null | head -20 || cat /etc/resolv.conf"
          )
          stdout.read()  # drain
          ssh.close()
  
          print(f"  ✓ DNS resolver restored")
          return True
      except Exception as e:
          print(f"  ✗ Failed to correct DNS: {e}")
          return False
  
  # Udates the ticked to show the issue has been resolved
  def update_ticket_resolved(device_name):
      """Create/update a ticket showing the DNS issue is resolved"""
      ticket_data = {
          "title": f"DNS Issue Resolved - {device_name}",
          "description": f"DNS setting on {device_name} was corrected back to {PRIMARY_DNS}.",
          "priority": "medium",
          "status": "resolved"
      }
  
      try:
          response = requests.post(API_URL, json=ticket_data, headers=HEADERS, timeout=10)
          if response.status_code in [200, 201]:
              print(f"  ✓ Ticket updated/resolved for {device_name}")
              return True
          else:
              print(f"  ✗ Ticket update failed: {response.status_code}")
              return False
      except Exception as e:
          print(f"  ✗ Ticket error: {e}")
          return False
  
  def generate_alert(interval_seconds=INTERVAL):
      scan_count = 0
      try:
          while True:
              scan_count += 1
              print(f"\n=== Scan #{scan_count} at {datetime.now().strftime('%Y-%m-%d %H:%M:%S')} ===")
  
              devices = get_devices(CSV_FILE)
  
              for device in devices:
                  name = device["name"]
                  ip = device["ip"]
                  username = device["username"]
                  password = device["password"]
  
                  if ip in ["None", "DHCP"] or username.lower() == "none":
                      print(f"{name:12} | Skipped")
                      continue
  
                  print(f"Checking {name:12} ({ip})...", end=" ")
                  current_dns = get_current_dns(ip, username, password)
  
                  if current_dns in AUTHORIZED_DNS:
                      print("DNS OK")
                  else:
                      print(f"ALTERED (found {current_dns})")
                      send_dns_altered_email(name, ip, current_dns)
                      correct_dns(ip, username, password)
                      update_ticket_resolved(name)
  
              print(f"\nSleeping {interval_seconds} seconds... (Ctrl+C to stop)")
              time.sleep(interval_seconds)
  
      except KeyboardInterrupt:
          print(f"\n\nDNS monitoring stopped after {scan_count} scans.")
  
  
  # ---------------------------------------
  # Main
  # ---------------------------------------
  # As usual, let's set up our screen
  if __name__ == "__main__":
      clear_screen()
      print("=" * 60)
      print("DNS Alteration Detection & Remediation")
      print("=" * 60)
      print()
  
      generate_alert()
  
      # Closure
      print("=" * 60)
      print("DNS remediation process completed. ~ Goodbye")
      print("=" * 60)
  
  # ---------------------------------------
  # End of script
  # ---------------------------------------
```

</details> <!-- Ends Actual Script -->

</details> <!-- Ends Email Alert -->

---

<details> 
<summary><strong>Script 6 - Health Log</strong></summary>

## Script 6 - Health Log

**Learning Objectives:**
- How to determine when a device is in a healthy status
- How to create a log to log the status of devices that are healthy
- How to add to that log during future checks

**Pseudocode**
1) Create a log titled dns_health
2) Get a list of devices to check
3) SSH into each device and check the DNS settings
4) Log devices that match the expected DNS settings

<details>
<summary><strong>Actual Script</strong></summary>

```python
  # c5_dns_health_log.py
  """
  C5. Write a Python script to automatically add an entry into a log file,
  including device name, date, and time, indicating when the DNS service
  is functioning correctly and has not been altered.
  """
  
  from pathlib import Path
  from datetime import datetime
  from network_utility import (
      get_devices,
      open_ssh,
      clear_screen
  )
  
  # ---------------------------------------
  # Constants
  # ---------------------------------------
  CSV_FILE = Path("/home/student/d522/network_devices.csv")
  LOG_FILE = Path("dns_health.log")
  PRIMARY_DNS = "10.10.10.10"
  SECONDARY_DNS = "10.10.10.20"
  AUTHORIZED_DNS = {PRIMARY_DNS, SECONDARY_DNS}
  
  # ---------------------------------------
  # Functions
  # ---------------------------------------
  def get_current_dns(ip, username, password):
      """
      SSH in and determine the effective upstream DNS.
      If resolv.conf shows 127.0.0.53 (systemd-resolved), query the real upstream.
      """
      try:
          ssh = open_ssh(ip, username, password)
  
          # 1. Read resolv.conf
          stdin, stdout, stderr = ssh.exec_command("cat /etc/resolv.conf")
          resolv = stdout.read().decode()
  
          nameservers = []
          for line in resolv.splitlines():
              line = line.strip()
              if line.lower().startswith("nameserver"):
                  parts = line.split()
                  if len(parts) >= 2:
                      nameservers.append(parts[1])
  
          if not nameservers:
              ssh.close()
              return "No DNS found"
  
          # 2. If we only see the local stub, ask systemd-resolved for the real upstream
          if all(ns in ("127.0.0.53", "127.0.0.1") for ns in nameservers):
              # Try resolvectl first, then systemd-resolve
              cmd = (
                  "resolvectl status 2>/dev/null || "
                  "systemd-resolve --status 2>/dev/null || true"
              )
              stdin, stdout, stderr = ssh.exec_command(cmd)
              status = stdout.read().decode()
  
              upstream = []
              for line in status.splitlines():
                  line = line.strip()
                  # Common lines: "DNS Servers: 10.10.10.10" or "Current DNS Server: ..."
                  if "DNS Server" in line or "DNS Servers" in line:
                      # grab IPv4-looking tokens
                      for token in line.replace(":", " ").split():
                          if token.count(".") == 3 and not token.startswith("127."):
                              upstream.append(token)
  
              ssh.close()
  
              if upstream:
                  # Return the first real upstream address
                  return upstream[0]
              # Fall back to what resolv.conf said
              return nameservers[0]
  
          # 3. Normal case: return first non-stub nameserver (or first entry)
          for ns in nameservers:
              if ns not in ("127.0.0.53", "127.0.0.1"):
                  ssh.close()
                  return ns
  
          ssh.close()
          return nameservers[0]
  
      except Exception as e:
          return f"Error: {e}"
  
  def is_dns_service_active(ip, username, password):
      """Return True if a DNS-related service is active on the host"""
      try:
          ssh = open_ssh(ip, username, password)
          cmd = (
              "systemctl is-active systemd-resolved 2>/dev/null || "
              "systemctl is-active named 2>/dev/null || "
              "systemctl is-active bind9 2>/dev/null || "
              "systemctl is-active resolvconf 2>/dev/null || "
              "echo inactive"
          )
          stdin, stdout, stderr = ssh.exec_command(cmd)
          status = stdout.read().decode().strip().splitlines()[0]
          ssh.close()
          return status == "active"
      except Exception:
          return False
          
  def log_healthy_dns(device_name):
      """Append a log entry for a healthy DNS device"""
      timestamp = datetime.now().strftime("%Y-%m-%d %H:%M:%S")
      entry = f"{timestamp} | {device_name} | DNS is functioning correctly (not altered)\n"
  
      with open(LOG_FILE, "a") as log:
          log.write(entry)
  
      print(f"  ✓ Logged healthy DNS for {device_name}")
  
  
  def check_and_log_dns():
      devices = get_devices(CSV_FILE)
  
      for device in devices:
          name = device["name"]
          ip = device["ip"]
          username = device["username"]
          password = device["password"]
  
          if ip in ["None", "DHCP"] or username.lower() == "none":
              print(f"{name:12} | Skipped")
              continue
  
          print(f"Checking {name:12} ({ip})...", end=" ")
  
          current_dns = get_current_dns(ip, username, password)
          service_ok = is_dns_service_active(ip, username, password)
  
          if current_dns in AUTHORIZED_DNS and service_ok:
              print("DNS OK + service active → logging")
              log_healthy_dns(name)
          elif current_dns in AUTHORIZED_DNS and not service_ok:
              print(f"DNS setting OK but service NOT active → not logged")
          else:
              print(f"ALTERED ({current_dns}) or service down → not logged")
  
  
  # ---------------------------------------
  # Main
  # ---------------------------------------
  if __name__ == "__main__":
      clear_screen()
      print("=" * 60)
      print("DNS Health Logger")
      print("=" * 60)
      print()
  
      check_and_log_dns()
  
      print()
      print("=" * 60)
      print(f"Log written to: {LOG_FILE}")
      print("DNS health logging completed. ~ Goodbye")
      print("=" * 60)
  
  # ---------------------------------------
  # End of Script
  # ---------------------------------------
```

</details> <!-- Ends Actual Script -->

</details> <!-- Ends Health Log -->

---

<details>
<summary><strong>Utility Script</summary>

  This script handles: Reading the csv file, getting DHCP addresses, pining devices, opening an SSH connection, and clearing the screen.  
  It is a script that grew from Task 1 `utility_get_devices` with DHCP lease lookup and SSH helper.

  Learning Objectives: 
  - How to create my own import script
  - How to better reuse code
  
<details>
<summary><strong>Actual Script</strong></summary> 
  
    ```python
    # network_utility.py
    """
    This script will just grab all the devices from the csv file and
    adds the DHCP IP address for PC1-PC4
    Then, return an array of devices.
    *** This script has grown. It is now a utility script that is called from other scripts to do repetitive tasks. ***
    """
    
    # ---------------------------------------
    # Imports
    # ---------------------------------------
    import os                   # For my love of clear screen
    import csv                  # Used to read the csv file
    import subprocess           # Used for ping
    import paramiko             # Used for SSH
    from pathlib import Path    # Used to get the location of the csv file
    
    
    # ---------------------------------------
    # Constants
    # ---------------------------------------
    CSV_FILE = Path("network_devices.csv")
    
    
    # ---------------------------------------
    # Functions
    # ---------------------------------------
    # Pulls DHCP IP addresses
    def get_dhcp_ips():
        """Query the router for current DHCP leases and return a dict of PC names → IPs"""
        dhcp_ips = {}
    
        try:
            ssh = open_ssh("10.10.10.1", "vyos", "vyos")
    
            # Non-interactive VyOS operational command
            cmd = "/opt/vyatta/bin/vyatta-op-cmd-wrapper show dhcp server leases"
            stdin, stdout, stderr = ssh.exec_command(cmd)
            output = stdout.read().decode()
            err = stderr.read().decode()
            ssh.close()
    
            if err.strip():
                print(f"Warning (stderr): {err.strip()}")
    
            for line in output.splitlines():
                line = line.strip()
                if not line or "IP Address" in line or line.startswith("---"):
                    continue
    
                parts = line.split()
                if len(parts) < 2:
                    continue
    
                ip = parts[0]
                for token in parts:
                    if token.lower() in ["pc1", "pc2", "pc3", "pc4"]:
                        dhcp_ips[token.upper()] = ip
                        break
    
        except Exception as e:
            print(f"Warning: could not get DHCP leases: {e}")
    
        return dhcp_ips
    
    # ---------------------------------------
    
    # Get a list of devices and thier information
    def get_devices(filepath):
        # Known DHCP addresses discovered from the lab
        # print("Reached the correct script.")   # Was added while trying to fix a bug. 
        dhcp_ips = get_dhcp_ips()
        
        # Creates the variable to hold all the devices 
        devices = []
        
        # Load device information from the csv file
        with open(filepath, mode="r") as file:
            reader = csv.DictReader(file)
            
            # Iterate through each row grabbing the information needed to ping / ssh
            for row in reader:
                name = row["Device Name"]
                ip = row["Device Address"]
                username = row["Username"]
                password = row["Password"]
                
                # Checks to see if the device is on the list of DHCP addresses
                if ip.upper() == "DHCP" and name in dhcp_ips:
                    ip = dhcp_ips[name]
            
                devices.append({
                    "name": name,
                    "ip": ip,
                    "username": username,
                    "password": password
                })
        return devices  # Sends this array back to the caller.
    
    
    # ---------------------------------------
    # Ping a device and return the results to the calling script
    def ping_device(ip):
        """Returns True if the device responds to ping"""
        if ip in ["None", "DHCP"]:
            return False
        try:
            result = subprocess.run(
                ["ping", "-c", "2", ip],
                stdout=subprocess.DEVNULL,
                stderr=subprocess.DEVNULL
            )
            return result.returncode == 0
        except:
            return False
    
    # ---------------------------------------
    # Open an SSH tunnel and return that connection to the calling script
    def open_ssh(ip, username, password, timeout=5):
        """Create and return an SSH connection"""
        ssh = paramiko.SSHClient()
        ssh.set_missing_host_key_policy(paramiko.AutoAddPolicy())
        ssh.connect(ip, username=username, password=password, timeout=timeout)
        return ssh
    
    
    
    # ---------------------------------------
    # Clear the screen based on operating system
    def clear_screen():
        """Clear the terminal screen (Works on Windows and Linux/Mac)"""
        if os.name == "nt":
            os.system("cls")
        else:
            os.system("clear")
    
    
    
    # End of utility_get_devices
  
  
    ```
</details> <!-- Ends actual script -->


</details> <!-- Ends the Utility Script -->

---


<details> 
<summary><strong>Menu Script</strong></summary>

  I created this menu script to run all the scripts required for class from one simple location.  
  I did have to modify several scripts after making this one to meet looping requirements,  
  consequently, this one may not work anymore. 

**Learning Objectives:**
- Python's match / case
- Accepting input from use
- Validating choice

<details>
<summary><strong>Actual Script</strong></summary>

  ```python
  # menu.py
  """
  This script will provide the user with a menu of options to be chosen.
  It will then call the script required to perform that task.
  If I get crazy enough, I'll include a FULL run, which, in theory,
      would get the devices, run a status check, send alerts, create tickets,
      backup the DNS servers, correct DNS settings on affected devices, 
      close the tickets, and notify stakeholders of the corrections... in theory anyway.
  """
  
  # ---------------------------------------
  # Imports
  # ---------------------------------------
  from network_utility import clear_screen
  from b1_backup_dns_server import backup_dns_server
  from c1_device_list import print_network_devices
  from c2_device_availability import check_device_availability
  from c3_ticket_generator import generate_ticket
  from c4_altered_dns_email import generate_alert
  from c5_dns_health_log import check_and_log_dns
  # I will addd the rest as I create them. 
  
  
  # ---------------------------------------
  # Constants
  # ---------------------------------------
  
  # ---------------------------------------
  # Functions
  # ---------------------------------------
  
  # Creates the menu
  def show_menu():
      print("=" * 60)
      print(" Network Monitoring & Remediation Menu")
      print("=" * 60)
      print("1. B1 Backup DNS server configurations")
      print("2. C1 List all network devices")
      print("3. C2 Check device availability + send alerts")
      print("4. C3 Create tickets for unavailable devices")
      print("5. C4 Detect & correct altered DNS settings")
      print("6. C5 DNS Health Log")
      print("8. FULL RUN (all of the above)")
      print("0. Exit")
      print("=" * 60)
  
  # Full run options
  def full_run():
      """Run everything in sequence"""
      print("\n=== Starting Full Run ===\n")
      print("=" * 60)
      print()
      
      print("B1. Backing up DNS Servers")
      print("-" * 60)
      backup_dns_server()
      print()
      
      print("C1. Display Network Devices")
      print("-" * 60)
      print_network_devices()
      print()
      
      print("C2. Checking availability")
      print("-" * 60)
      check_device_availability()
      print()
      
      print("C3. Generating Help Tickets")
      print("-" * 60)
      generate_ticket()
      print()
     
      print("C4. Fixing Issues and Sending Alerts")
      print("-" * 60)
      generate_alert()
      print()
  
      print("C5 DNS Health Log")
      print("-" * 60)
      check_and_log_dns()
      print()
  
      print("\n=== Full Run Complete ===")
      
  # The worker bee
  def main():
      while True:
          clear_screen()
          show_menu()
  
          choice = input("\nEnter your choice: ").strip()
  
          match choice:
              case "1":
                  clear_screen()
                  backup_dns_server()
              case "2":
                  clear_screen()
                  print_network_devices()
              case "3":
                  clear_screen()
                  check_device_availability()
              case "4":
                  clear_screen()
                  generate_ticket()
              case "5":
                  clear_screen()
                  generate_alert()
              case "6":
                  clear_screen()
                  check_and_log_dns()
              case "8":
                  clear_screen()
                  full_run()
              case "0":
                  print("\n~ Goodbye.\n")
                  break
              case _:
                  print("\nInvalid choice. Please try again.")
  
          input("\nPress Enter to return to the menu...")
  
  
  
  # ---------------------------------------
  # Main
  # ---------------------------------------
  
  if __name__ == "__main__":
      main()
  
  
  # ---------------------------------------
  # End of script
  # ---------------------------------------

```
</details> <!-- Ends Actual Script -->

</details> <!-- Ends Menu script -->










</details> <!-- Ends Task 2 -->

---



