# Networking and Security - Sandboxed network 

### **Introduction**

A sandboxed network is an isolated virtual environment designed for secure experimentation, learning, and testing. This project involves building a sandboxed network using VirtualBox and configuring multiple virtual machines to simulate a real-world networking setup. The network consists of two private subnets connected through a gateway router, enabling controlled communication while maintaining isolation from external networks.

---

### **Objective of Creating a Sandboxed Network** 

- **Practical Learning:** 
To gain hands-on experience in setting up and managing a private network.

- **Networking Concepts:** 
To understand key networking principles such as IP addressing, subnetting, and network interface configuration.

- **Safe Environment:** 
To provide a secure environment for testing configurations without affecting production networks.

- **Inter-Subnet Communication:**
To enable seamless communication between devices in separate subnets using a gateway.

---

### **Skills Learned**

- **Networking Fundamentals** 
Understanding IP addressing, subnetting, and network architecture. 
Configuring and managing routers, switches, and gateways. 

- **Network Configuration and Deployment** 
Setting up subnets and VLANs for segmented network environments. 
Assigning static and dynamic IPs and configuring DHCP servers. 

- **Infrastructure Management**
Installing and managing network hardware such as desktops, servers, and routers. 
Ensuring proper connectivity and communication between network nodes. 

- **Documentation and Reporting**
Documenting configurations, changes, and findings for reference and future use. 
Preparing security evaluation reports and recommendations. 

- **Monitoring and Troubleshooting** 
Using diagnostic tools like ping, traceroute, and packet analyzers. 
Identifying and resolving network connectivity issues. 

---

### **Tools Used**
- **VirtualBox**: Used to create and manage virtual machines (VMs) for the sandboxed network.
- **Ubuntu Server**: Acts as the gateway router with multi-interface configurations for NAT and subnet communications.
- **Ubuntu Desktop**: Functions as a desktop VM within one of the subnets.
- **Bitnami WordPress Server**: Serves as the application server providing a web-based service in the other subnet.

---

### **Prerequisites**
1. **Software**: Install VirtualBox and required virtual machine images (Ubuntu Server, Ubuntu Desktop, and Bitnami WordPress).
2. **Basic Networking Knowledge**: Understanding of IP addressing, subnetting, and configuring network adapters.
3. **Hardware Requirements**: Ensure sufficient CPU, RAM, and storage for hosting multiple VMs.
4. **Access to the Internet**: For downloading software and testing NAT configurations.

---

### **Steps to Create a Sandboxed Network**
1. **Design the Network**:
   - Create two subnets connected via a gateway router.
   - Assign unique private IP addresses to devices in each subnet.

2. **Configure Virtual Machines**:
   - Install and configure the gateway router, desktop VM, and application server.
   - Set up necessary interfaces and adapters.

3. **Verify Connectivity**:
   - Use ping commands to confirm communication within and between subnets.
   - Validate NAT configuration for internet access.

---

### **VirtualBox Setup**
1. Install VirtualBox.
2. Create new VMs with allocated resources (CPU, RAM, disk size) for Ubuntu Server, Ubuntu Desktop, and Bitnami WordPress.
3. Configure network settings for each VM:
   - Gateway VM: Multiple network adapters.
   - Other VMs: Internal Network and Bridge as per requirements.

---

### **Desktop VM**
- **Role**: Functions within Subnet 2 with IP address 192.168.13.4.
- **Configuration**:
  - Adapter connected to "Internal Network".
  - Assigned subnet mask 255.255.255.0.
    ![image](https://github.com/user-attachments/assets/47aa2566-0e5a-4072-ab63-b103a7526e9e)
    ![image](https://github.com/user-attachments/assets/750016c3-abfe-464c-86ec-ab713d8505e0)


- **Functionality**:
  - Acts as a client machine to test communication with the application server and gateway.

---

### **Gateway VM**
- **Role**: Central routing device enabling communication between subnets.
- **Network Interfaces**:
  - enp0s3: Configured for NAT (internet access).
  - enp0s8: Connects to Subnet 1 (IP 192.168.113.1).
  - enp0s9: Connects to Subnet 2 (IP 192.168.13.1).
    ![image](https://github.com/user-attachments/assets/5ddd2be3-932f-499d-a33c-070f89067ac9)
 
  - ***Edit the network configuration file:*** sudo nano /etc/netplan/00-installer-config.yaml
  -  ***Configure the network interfaces in netplan file:***
    ![image](https://github.com/user-attachments/assets/0fab93ac-7570-4ab8-ac69-5c5266451eb3)

  - ***Save Netplan Changes:*** sudo netplan apply
    
    ![image](https://github.com/user-attachments/assets/e5c6377e-fc5d-4551-9b7e-88f483c708a2)

- **Configuration**:
  - ***Enable IP forwarding.***
     - Open the sysctl configuration file: sudo nano /etc/sysctl.conf
     - Enable forwording: net.ipv4.ip_forward=1
     - Save changes: sudo sysctl -p

  - ***Set up routing rules for inter-subnet communication.***
      - ****Configure iptables:****
       - Allow forwarding between enp0s8 and enp0s9
          - sudo iptables -A FORWARD -i enp0s8 -o enp0s9 -j ACCEPT
          - sudo iptables -A FORWARD -i enp0s9 -o enp0s8 -j ACCEPT
       - Allow forwarding between enp0s3 and the internal interfaces
          - sudo iptables -A FORWARD -i enp0s3 -o enp0s8 -j ACCEPT
          - sudo iptables -A FORWARD -i enp0s8 -o enp0s3 -j ACCEPT
          - sudo iptables -A FORWARD -i enp0s3 -o enp0s9 -j ACCEPT
          - sudo iptables -A FORWARD -i enp0s9 -o enp0s3 -j ACCEPT
      - Enable NAT on enp0s3 for internet access
          - sudo iptables -t nat -A POSTROUTING -o enp0s3 -j MASQUERADE

      - ****Save IPtable Changes:****
        - sudo apt install iptables-persistent
        - sudo netfilter-persistent save
        - sudo netfilter-persistent reload 

---

### **Application Server VM**
- **Role**: Provides web services in Subnet 1 with IP 192.168.113.4.
  ![image](https://github.com/user-attachments/assets/4a43d23d-88c7-473a-a874-b504be98361b)
  ![image](https://github.com/user-attachments/assets/c1db4d09-2aee-4d5a-9291-627c9c9c37c2)

- **Configuration**:
  - Connected to Subnet 1 via an internal network adapter.
  - Accessible via HTTP/HTTPS to provide WordPress-based services.

---

### **Configuration of Network Adapters**
- **Gateway VM**:
  - NAT (Adapter 1): For internet access.
  - Internal Network 1 (Adapter 2): Interface enp0s8.
  - Internal Network 2 (Adapter 3): Interface enp0s9.
- **Desktop VM**:
  - Internal Network 2: Connects to Gateway via Subnet 2.
- **Application Server VM**:
  - Internal Network 1: Connects to Gateway via Subnet 1.

---

### **IP Configuration**
| **Device**       | **IP Address**   | **Subnet Mask**     | **Purpose**                         |
|-------------------|------------------|---------------------|-------------------------------------|
| Desktop VM        | 192.168.13.4    | 255.255.255.0       | Subnet 2 device for testing.        |
| Application Server| 192.168.113.4   | 255.255.255.0       | Subnet 1 web service provider.      |
| Gateway NAT       | 10.0.2.15       | 255.255.255.0       | Internet access.                    |
| Gateway Subnet 1  | 192.168.113.1   | 255.255.255.0       | Connects to Subnet 1.               |
| Gateway Subnet 2  | 192.168.13.1    | 255.255.255.0       | Connects to Subnet 2.               |

--- 


 

 
