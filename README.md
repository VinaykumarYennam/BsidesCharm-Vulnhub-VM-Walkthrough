# BSides Vancouver 2018-Vulnhub-VM-Walkthrough
This walk-through will detail how the BSides Vancouver 2018 workshop VM was exploited to get root.
The following walkthrough contains spoilers for those who haven't tried it. Give it a try by grabbing the VM from https://www.vulnhub.com/entry/bsides-vancouver-2018-workshop,231/

## Let's Start! 
Download and import the VM into your favorite Virtualization software. In this walkthrough, I used VMware Workstation.
Let's get started!!
I will be using Kali Linux as attacking VM and BSides VM as a target.<br />
1. We will scan the subnet created by VMware to find the IP address of the target VM.<br />
  The command is **_nmap -sP [subnet IP range]_**
  The screenshot is attached for the above Ping sweep by nmap.<br />
  ![capture1](https://user-images.githubusercontent.com/35183615/40511085-a8f59a8e-5f6c-11e8-94bc-0708d639c3fa.JPG)<br />
The highlighted IP address is the target IP address.<br />
2. After getting the IP address, we will version scan the target by probing all ports so that we don't miss out any running service.
  The command for the same is **_nmap -sV -p- [target-IP]_**
  The image below shows the services running on the target machine along with the version number.
  ![capture2](https://user-images.githubusercontent.com/35183615/40511112-bc38855c-5f6c-11e8-9c42-1097e2fae031.JPG)
  There are three services running that are exposed to the outside world. This is our attack surface. Any vulnerability in these three
  services might give us access to the system. Let's explore the services.
3. #### Failed<br />
  As soon as Web server was spotted, Web browser was pointed to the target-IP to find any web related vulnerabilities. Unfortunately, the
  webpage only displayed a welcome message saying it works.<br />
  SSH service was exposed but we cannot exploit/brute force ssh credentials unless we know the users on the system. This attempt failed.<br />
4. Only one service remains which we haven't probed yet and that is FTP. Let's scan the FTP using **metasploit** to find more details.
  The command used at msfconsole is _**use auxiliary/scanner/ftp/anonymous**_<br />
  Set the other options accordingly and issue _**exploit**_. The Screenshots below shows the same.
  ![capture3](https://user-images.githubusercontent.com/35183615/40514135-1a58d732-5f76-11e8-8e0a-ce559101cd72.JPG)
  ![capture4](https://user-images.githubusercontent.com/35183615/40514155-2c11e388-5f76-11e8-8e75-053653057f60.JPG)<br />
  The scan shows that FTP is configured for anonymous read only. So dropping a backdoor will not work.<br />
  Let's connect to the FTP server and check what are the files hosted on it. WinSCP was used to connect to the target machine with username as **anonymous** and password as **anonymous** for FTP server. 
  ## Breakthrough
  On the FTP server, there was a file named users in the public directory. The file listed the users. At this point, I had no clue about
  the information in the file. Then, I realized that these users may be users of the system. It can be used to bruteforce the credentials via **SSH**. The contents of the file is shown below<br />
  ![capture5](https://user-images.githubusercontent.com/35183615/40514816-9834d410-5f78-11e8-9ad2-c7eaa6a95131.JPG)
  ## Using Hydra to bruteforce credentials via SSH.<br />
  The SSH was configured to use Public Key for authentication. This was a setback. I still gave it a try for all users and luckily the user anne was configured to use password for authentication. The image below shows password denial for few users.
  ![capture6](https://user-images.githubusercontent.com/35183615/40515180-d17f00aa-5f79-11e8-8240-01317c1044b7.JPG)
Then Hydra was used to go through dictionary rockyou.txt in kali for user anne. It successfully cracked the password as **princess**
![capture7](https://user-images.githubusercontent.com/35183615/40515566-5185e54c-5f7b-11e8-9ae2-a42c2500d33b.JPG)

## Success
I SSHed into the target VM with the credentials of anne and it gave me a shell. The image below shows successful login and id of anne and later used sudo to elevate privileges. It gave root privileges.
![capture8](https://user-images.githubusercontent.com/35183615/40515681-b67959b6-5f7b-11e8-87cb-1804b1f7eecd.JPG)
![capture9](https://user-images.githubusercontent.com/35183615/40515699-c80008d8-5f7b-11e8-9645-2361550cd33c.JPG)
