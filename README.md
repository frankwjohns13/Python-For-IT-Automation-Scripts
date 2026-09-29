# Python-For-IT-Automation - Scripts

This holds the python scripts I wrote for my class assignments. 

One big thing to remember about Python, indents matter... **a lot**!

The way this is setup:
- Tasks #
  - Task Overview
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

### Overview
In this task, I continued in the role of a network administrator following a recent DNS service outage caused by unauthorized configuration changes. The goal was to design and implement a proactive monitoring solution that continuously checks DNS configurations and device status across the network.

### Scenario
After restoring normal DNS service, the focus shifted to preventing similar incidents. The objective was to detect unauthorized DNS changes or service disruptions early, before they could impact business operations.

### Problem Statement
Design and implement a solution that:
- Regularly monitors DNS configurations and device status
- Automatically detects anomalies or unauthorized changes
- Generates real-time alerts for stakeholders
- Creates tickets when issues are found

The overall goal is to improve network security and reliability through early detection and rapid response.

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
    TOKEN = "vGkbXkGLqQSo7YLflp9DutuG8st4xdPPF7wnTcwB0FE"
    
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
  
  This is an automated notification to inform you that the DNS service issue and all related device compromises have been successfully resolved. The following devices were affected and have now be remediated:
  
  API
  DB
  DNS1
  DNS2
  ROUTER1
  SVR1
  SVR2
  
  No further action is required at thhis time. if you have any questions or concerns, please contact the IT support team.
  
  Thank you for your attention.
  
  Best regards,
  Network Monitoring System
  """
  
  # Create the email message
  message = MIMEText(body)
  message["Subject"] = subject
  message["from"] = SENDER
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







</details> <!-- Ends Task 2 -->





