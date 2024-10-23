# Welcome to the "Roll Your Own EDR/XDR" Workshop

This guide will provide you with a step-by-step of all the commands we will use throughout this workshop. Please reference it as we move forward. If you have questions, feel free to ask your group moderator.

## Lab 1: Building the Lab

In this lab we will actually be building the lab environment we will be using Virtual Box and Windows. 

### Tools we will use:

- [VirtualBox](https://www.virtualbox.org/wiki/Downloads)
- [Microsoft Windows 10 Enterprise Evaluation Copy](https://www.microsoft.com/en-us/evalcenter/download-windows-10-enterprise) (download 64 bit)

1. Download and install [VirtualBox](https://www.virtualbox.org/wiki/Downloads) on you laptop if you do not already have it, install the appropriate version for your operating system.
 ![Download VirtualBox](/img/1_virtual_box.png)

2. Download the [Microsoft Windows 10 Enterprise Evaluation Copy ISO](https://www.microsoft.com/en-us/evalcenter/download-windows-10-enterprise) (64 bit) and save it in a folder on your system called "EDR_Workshop"
 ![Download Windows](/img/2_windows_download.png)

3. Open VirtualBox and click on "New" to create a new VM

 ![Open VirtualBox](/img/3_add_vm.png)

4. Configure your new VM.
   - For "Folder" navigate to the "EDR_Workshop" folder you created (where the Windows ISO was saved).
   - For "ISO" navigate to the actual Windows ISO you downloaded.
   - The "Edition" should automatically populate.
   - Select "Skip Unattended Installation"
   - Click on "Next"
  
  ![VM Setup](/img/4_vm_setup.png)

5. On the "Hardware" screen bump the memory up to around 8GB if you system has the RAM available (2GB will still work...things will just be slow)

![VM Setup](/img/5_vm_hardware_setup.png) 

6. Leave the "Virtual Hard disk" screen with the defaults.

  ![VM Setup](/img/6_vm_storage_setup.png)

7. On the next screen review and click "Finish" and then click "Start"

8. Go through the install process for Windows

9. When asked to "Sign in with Microsoft" click on "Domain join intead" in the lower left hand corner and create a local username and password. 
![VM Setup](/img/7_windows_setup1.png)

10. In VirtualBox go to the top menu and go to "Devices -> Insert Guest Additions CD image"
![VM Setup](/img/7-2_windowsvmsetup.png)

11. Navigate via explorer to the now mounted CD drive and access the "VirtualBox Guest Additions" and double click on "VBoxWindowsAdditions-amd64" and follow the wizard to install and let the Windows system reboot. 

![VM Setup](/img/7-3_windowsvmsetup.png) 

12. Go to the top menu in VirtualBox and go to "Machine -> Take Snapshot" and name it "Base Install"

## Lab 2: Configuring EDR Telemetry in LimaCharlie

We will use the free tier of LimaCharlie for our lab, this will allow us to easily deploy an agent to start gathering telemetry from our Windows system, and then start writing and deploying detection and response rules. 

1. [Create a free LimaCharlie account](https://free.limacharlie.io) (please select BSidesPDX for the first option, feel free to use a burner email). 

![VM Setup](/img/8_lc_setup1.png)

![VM Setup](/img/9_lc_setup2.png)



## Lab 3: YARA Scanning

## Lab 4: Atomic Red Team

## Lab 5: Velociraptor
