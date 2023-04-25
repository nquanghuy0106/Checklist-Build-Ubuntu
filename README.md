# Checklist-Build-Ubuntu
check list build ubuntu template
# Update & Upgrade OS
> sudo apt-get update && sudo apt-get upgrade
# Set TimeZone Asia/Ho_Chi_Minh
> sudo timedatectl set-timezone Asia/Ho_Chi_Minh
# Install DKMS package for Nvidia Services
> sudo apt-get install dkms -y
# Install Nvidia Driver
> sudo dpkg -i /tmp/Nvidia.x.x..xxx
# Config cloud-init boot auto generate token license from server & restart nvidia services
> sudo vi /etc/cloud/cloud.cfg \
bootcmd: \
 – curl --insecure -L -X GET https://103.140.249.10/-/client-token -o /etc/nvidia/ClientConfigToken/client_configuration_token_$(date '+%d-%m-%Y-%H-%M-%S').tok \
 – systemctl restart nvidia-gridd.service
# Check license assign
> nvidia-smi -q | grep "Licensed"
# Create new bash script for auto renew token license & restart services apply new lic
> cd /etc/nvidia \
> sudo vi renew-lic.sh \
#!/bin/sh \
wget --no-check-certificate -O /etc/nvidia/ClientConfigToken/client_configuration_token_$(date '+%d-%m-%Y-%H-%M-%S').tok https://103.140.249.10/-/client-token && systemctl restart nvidia-gridd.service \
sudo chmod +x renew-lic.sh
# Set crontab auto generate new lic from server & restart services after 00:00 AM day 1 after 2 month
> sudo crontab -e \
0 0 1 */2 * /etc/nvidia/renew-lic.sh 
# remove old machine-id for every vm after create generate new machine-id 
> sudo rm /etc/machine-id && sudo touch /etc/machine-id
# remove every bash history for package template
> sudo cat /dev/null > ~/.bash_history && history -c

# Bug Ubuntu 22.04 can't boot OS when restart nvidia-gridd service from cloud-init.
# Use Systemd for get license from server && restart nvidia gridd service
> sudo vi /etc/systemd/system/renew-lic.sh 

> [Unit] \
> Description=Renew Lic.

> [Service] \
> Type=simple \
> ExecStart=/bin/bash /etc/nvidia/renew-lic.sh 

> [Install] \
> WantedBy=multi-user.target

#Next, we’ll need to set the file permissions to 644, and enable our service by using systemctl:
> sudo chmod 644 /etc/systemd/system/renew-lic.service \
> sudo systemctl enable renew-lic.service

# Windows Task Schedule
![image](https://user-images.githubusercontent.com/106576870/234170668-7ca835f3-f2d1-457a-9a79-220bebb12646.png)
![image](https://user-images.githubusercontent.com/106576870/234170721-4da54922-5a45-49b2-896b-345d7d640486.png)

arguments \
-executionpolicy bypass -noninteractive -file "C:\Program Files\Cloudbase Solutions\Cloudbase-Init\LocalScripts\renew-lic.ps1"


