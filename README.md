# Python-For-IT-Automation - Scripts

This holds the python scripts I wrote for my class assignments. 

---

<details>
<summary><strong>Task1: Incident Response and Remediation</strong></summary>

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
      """
      Read and display network device names from a CSV file.
      """
      import csv                                    # Needed to read csv files
      from pathlib import Path                      # Needed to get to the file
      
      # Path to csv file
      csv_file = Path("network_devices.csv")
      
      # Read the csv file and print device names
      with open(csv_file, mode="r") as file:        # Holds the file open in read only mode (also automatically closes the file when done
          reader = csv.DictReader(file)             # Reads the file
          
          print("Network Devices:")                 # Header for the print display
          print("-" * 30)                           # Gives the header a separation
          
          for row in reader:
              device_name = row["Device Name"]      # "Device Name" is the header used in the csv file 
              print(device_name)                    # Prints to the console
              
              
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
    
    import csv                  # Used to read the csv file
    import subprocess           # Used to perform the ping
    import paramiko             # Used for SSH
    from pathlib import Path    # Used to get the location of the csv file
    
    EXPECTED_DNS = "10.10.10.10"
    CSV_FILE = Path("network_devices.csv")
    
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
    print("Device Status & DNS Verification Report")
    print("=" * 75)
    
    # open the csv file as read only
    with open(CSV_FILE, mode="r") as file:
        reader = csv.DictReader(file)

      # itterate through each row grabbing the information needed to ping / ssh
      for row in reader:
          name = row["Device Name"]
          ip = row["Device Address"]
          username = row["Username"]
          password = row["Password"]
  
          # Skip only true non-manageable devices
          if ip in ["None"] or username.lower() == "none":
              print(f"{name:12} | Skipped (no management access)")
              continue
  
          # Handle DHCP devices – still try if possible
          if ip == "DHCP":
              print(f"{name:12} | DHCP device - limited check")
              continue
  
          # ping the device to see if it is reachable
          reachable = ping_device(ip)
  
          # if it is not reachable add to the unreachable list
          if not reachable:
              print(f"{name:12} | {ip:15} | UNREACHABLE")
              continue
  
          # grab ssh in
          current_dns = get_current_dns(ip, username, password)
  
          # checks to see if the dns configuration matches what is expected
          if current_dns == EXPECTED_DNS:
              status = "OK - DNS correct"
          else:
              status = f"COMPROMISED - DNS is {current_dns}"
  
          print(f"{name:12} | {ip:15} | {status}")

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
































<details> <!-- Starts Actual Script -->
<summary><strong>Actual Script</strong></summary>



</details> <!-- Ends Actual Script -->




---
  
</details> <!-- Ends Script 3 -->


<details> <!-- Starts Script 4 -->
<summary><strong>Script 4 - </strong></summary>

## Script 4 - 




<details> <!-- Starts Actual Script -->
<summary><strong>Actual Script</strong></summary>



</details> <!-- Ends Actual Script -->




---
  
</details> <!-- Ends Script 4 -->


<details> <!-- Starts Script 5 -->
<summary><strong>Script 5 - </strong></summary>

## Script 5 - 




<details> <!-- Starts Actual Script -->
<summary><strong>Actual Script</strong></summary>



</details> <!-- Ends Actual Script -->




---
  
</details> <!-- Ends Script 5 -->

<details> <!-- Starts Script 6 -->
<summary><strong>Script 6 - </strong></summary>

## Script 6 - 




<details> <!-- Starts Actual Script -->
<summary><strong>Actual Script</strong></summary>



</details> <!-- Ends Actual Script -->




---
  
</details> <!-- Ends Script 6 -->


<details> <!-- Starts Script 7 -->
<summary><strong>Script 7 - </strong></summary>

## Script 7 - 




<details> <!-- Starts Actual Script -->
<summary><strong>Actual Script</strong></summary>



</details> <!-- Ends Actual Script -->




---
  
</details> <!-- Ends Script 7 -->





















---

</details> <!-- Ends Task 1 -->


<details>
<summary><strong>Task 2: Proactive Monitoring and Prevention</strong></summary>
  
## Task 2: Proactive Monitoring and Prevention







</details> <!-- Ends Task 2 -->





