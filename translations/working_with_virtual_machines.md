# Working with Virtual Machines

## Task 1: Create the VM
### Define a VM using advanced options

1. First, we reserve an external IP address 'mc-server-ip' for the instance.
```bash
gcloud compute addresses create mc-server-ip --region=us-central1
```


2. To get the reserved IP address, run the following command:
```bash
gcloud compute addresses list
```

> Note the address, it will be used referred to later as [MC-SERVER-IP]


3. To create the VM instance, replace [MC-SERVER-IP] with the IP address you noted earlier and run the following command:
```bash
gcloud beta compute instances create mc-server --zone=us-central1-a \
--scopes=https://www.googleapis.com/auth/servicecontrol,\
https://www.googleapis.com/auth/service.management.readonly,\
https://www.googleapis.com/auth/logging.write,\
https://www.googleapis.com/auth/monitoring.write,\
https://www.googleapis.com/auth/trace.append,\
https://www.googleapis.com/auth/devstorage.read_write \
--create-disk=mode=rw,size=50,type=pd-ssd,name=minecraft-disk,device-name=minecraft-disk \
--tags=minecraft-server --address=[MC-SERVER-IP]
```
Notice that a standard SSD disk 'minecraft-disk' of size 50GB was also created and added to the instance on creation.

## Task 2: Prepare the data disk
### Create a directory and format and mount the disk
The disk is attached to the instance, but it is not yet mounted or formatted.

1. SSH into mc-server by running the following command:
```bash
gcloud compute ssh mc-server --zone=us-central1-a
```

2. If prompted, enter Y and press enter three times.

3. To create a directory that serves as the mount point for the data disk, run the following command:
```bash
sudo mkdir -p /home/minecraft
```

4. To format the disk, run the following command:
```bash
sudo mkfs.ext4 -F -E lazy_itable_init=0,\
lazy_journal_init=0,discard \
/dev/disk/by-id/google-minecraft-disk
```

Result (do not copy; this is example output):
```bash
mke2fs 1.42.12 (29-Aug-2014)
Discarding device blocks: done
Creating filesystem with 13107200 4k blocks and 3276800 inodes
Filesystem UUID: 3d5b0563-f29e-4107-ad1a-ba7bf11dcf7c
Superblock backups stored on blocks:
        32768, 98304, 163840, 229376, 294912, 819200, 884736, 1605632, 2654208,
        4096000, 7962624, 11239424
Allocating group tables: done
Writing inode tables: done
Creating journal (32768 blocks): done
Writing superblocks and filesystem accounting information: done
```

5. To mount the disk, run the following command:
```bash
sudo mount -o discard,defaults /dev/disk/by-id/google-minecraft-disk /home/minecraft
```

No output is displayed after the disk is mounted.

## Task 3: Install and run the application
The Minecraft server runs on top of the Java Virtual Machine (JVM), so it requires the Java Runtime Environment (JRE) to run. Because the server doesn't need a graphical user interface, you use the headless version of the JRE. This reduces the JRE's resource usage on the machine, which helps ensure that the Minecraft server has enough room to expand its own resource usage if needed.

### Install the Java Runtime Environment (JRE) and the Minecraft server
1. In the SSH terminal for mc-server, to update the Debian repositories on the VM, run the following command:
```bash
sudo apt-get update
```

2. After the repositories are updated, to install the headless JRE, run the following command:
```bash
sudo apt-get install -y default-jre-headless
```

3. To navigate to the directory where the persistent disk is mounted, run the following command:
```bash
cd /home/minecraft
```

4. To install wget, run the following command:
```bash
sudo apt-get install wget
```

5. If prompted to continue, type Y

6. To download the current Minecraft server JAR file (1.11.2 JAR), run the following command:
```bash
sudo wget https://launcher.mojang.com/v1/objects/d0d0fe2b1dc6ab4c65554cb734270872b72dadd6/server.jar
```

### Initialize the Minecraft server
1. To initialize the Minecraft server, run the following command:
```bash
sudo java -Xmx1024M -Xms1024M -jar server.jar nogui
```

Result (do not copy; this is example output):
```bash
[21:01:54] [main/ERROR]: Failed to load properties from file: server.properties
[21:01:54] [main/WARN]: Failed to load eula.txt
[21:01:54] [main/INFO]: You need to agree to the EULA in order to run the server. Go to eula.txt for more info.
```

The Minecraft server won't run unless you accept the terms of the End User Licensing Agreement (EULA).

2. To see the files that were created in the first initialization of the Minecraft server, run the following command:
```bash
sudo ls -l
```

> You could edit the server.properties file to change the default behavior of the Minecraft server.

3. To edit the EULA, run the following command:
```bash
sudo nano eula.txt
```

4. Change the last line of the file from eula=false to eula=true
5. Press Ctrl+O, ENTER to save the file and then press Ctrl+X to exit nano.

> Don't try to restart the Minecraft server yet. You use a different technique in the next procedure.

### Create a virtual terminal screen to start the Minecraft server
If you start the Minecraft server again now, it is tied to the life of your SSH session: that is, if you close your SSH terminal, the server is also terminated. To avoid this issue, you can use screen, an application that allows you to create a virtual terminal that can be "detached," becoming a background process, or "reattached," becoming a foreground process. When a virtual terminal is detached to the background, it will run whether you are logged in or not.

1. To install screen, run the following command:
```bash
sudo apt-get install -y screen
```

2. To start your Minecraft server in a screen virtual terminal, run the following command: (Use the -S flag to name your terminal mcs)
```bash
sudo screen -S mcs java -Xmx1024M -Xms1024M -jar server.jar nogui
```

Result (do not copy; this is example output):
```bash
...
[21:06:06] [Server-Worker-1/INFO]: Preparing spawn area: 83%
[21:06:07] [Server-Worker-1/INFO]: Preparing spawn area: 85%
[21:06:07] [Server-Worker-1/INFO]: Preparing spawn area: 86%
[21:06:08] [Server-Worker-1/INFO]: Preparing spawn area: 88%
[21:06:08] [Server-Worker-1/INFO]: Preparing spawn area: 89%
[21:06:09] [Server-Worker-1/INFO]: Preparing spawn area: 91%
[21:06:09] [Server-Worker-1/INFO]: Preparing spawn area: 93%
[21:06:10] [Server-Worker-1/INFO]: Preparing spawn area: 95%
[21:06:10] [Server-Worker-1/INFO]: Preparing spawn area: 98%
[21:06:11] [Server-Worker-1/INFO]: Preparing spawn area: 99%
[21:06:11] [Server thread/INFO]: Time elapsed: 55512 ms
[21:06:11] [Server thread/INFO]: Done (102.484s)! For help, type "help"
```

### Detach from the screen and close your SSH session
1. To detach the screen terminal, press Ctrl+A, Ctrl+D. The terminal continues to run in the background. To reattach the terminal, run the following command:
```bash
sudo screen -r mcs
```

2. If necessary, exit the screen terminal by pressing Ctrl+A, Ctrl+D.

3. To exit the SSH terminal, run the following command:
```bash
exit
```

> Congratulations! You set up and customized a VM and installed and configured application software—a Minecraft server!

## Task 4: Allow client traffic
Up to this point, the server has an external static IP address, but it cannot receive traffic because there is no firewall rule in place. Minecraft server uses TCP port 25565 by default. So you need to configure a firewall rule to allow these connections.

### Create a firewall rule

1. To create the firewall rule, run the following command:
```bash
gcloud compute firewall-rules create minecraft-rule \
--direction=INGRESS \
--target-tags=minecraft-server \
--action=ALLOW \
--rules=tcp:25565 \
--target-tags=minecraft-server
```
Users can now access your server from their Minecraft clients.

### Verify server 

1. To get the external IP address of mc-server, run the following command:
```bash
gcloud compute instances list
```

2. Copy the External IP address for the mc-server VM.

3. Use the following website to test your Minecraft server: https://mcsrvstat.us/

> If the above website is not working, you can use a different site or the Chrome extension:

> https://dinnerbone.com/minecraft/tools/status/

## Task 5: Schedule regular backups
Backing up your application data is a common activity. In this case, you configure the system to back up Minecraft world data to Cloud Storage.

### Create a Cloud Storage bucket
1. SSH into mc-server by running the following command:
```bash
gcloud compute ssh mc-server --zone=us-central1-a
```

2. Create a globally unique bucket name, and store it in the environment variable YOUR_BUCKET_NAME. To make it unique, you can use your Project ID. Run the following command:
```bash
export YOUR_BUCKET_NAME=<Enter your bucket name here>
```

> You may make use of your project ID.

3. Verify it with echo:
```bash
echo $YOUR_BUCKET_NAME
```

4. To create the bucket using the gsutil tool, part of the Cloud SDK, run the following command:
```bash
gsutil mb gs://$YOUR_BUCKET_NAME-minecraft-backup
```

> If this command failed, you might not have created a unique bucket name. If so, choose another bucket name, update your environment variable, and try to create the bucket again.


> To make this environment variable permanent, you can add it to the root's .profile by running this command: `echo YOUR_BUCKET_NAME=$YOUR_BUCKET_NAME >> ~/.profile`

### Create a backup script
1. In the mc-server SSH terminal, navigate to your home directory:
```bash
cd /home/minecraft
```

2. To create the script, run the following command:
```bash
sudo nano /home/minecraft/backup.sh
```

3. Copy and paste the following script into the file:
```bash
#!/bin/bash
screen -r mcs -X stuff '/save-all\n/save-off\n'
/usr/bin/gsutil cp -R ${BASH_SOURCE%/*}/world gs://${YOUR_BUCKET_NAME}-minecraft-backup/$(date "+%Y%m%d-%H%M%S")-world
screen -r mcs -X stuff '/save-on\n'
```

4. Press Ctrl+O, ENTER to save the file, and press Ctrl+X to exit nano.

> The script saves the current state of the server's world data and pauses the server's auto-save functionality. Next, it backs up the server's world data directory (world) and places its contents in a timestamped directory (<timestamp>-world) in the Cloud Storage bucket. After the script finishes backing up the data, it resumes auto-saving on the Minecraft server.

5. To make the script executable, run the following command:
```bash
sudo chmod 755 /home/minecraft/backup.sh
```

### Test the backup script and schedule a cron job
1. In the mc-server SSH terminal, run the backup script:
```bash
. /home/minecraft/backup.sh
```

2. To verify that the backup file was written, run the following command:
```bash
gsutil ls gs://${YOUR_BUCKET_NAME}-minecraft-backup
```

3. In the mc-server SSH terminal, open the cron table for editing:
```bash
sudo crontab -e
```

4. When you are prompted to select an editor, type the number corresponding to nano, and press ENTER.

5. At the bottom of the cron table, paste the following line:
```bash
0 */4 * * * /home/minecraft/backup.sh
```

> That line instructs cron to run backups every 4 hours.

6. Press Ctrl+O, ENTER to save the cron table, and press Ctrl+X to exit nano.

> This creates about 300 backups a month in Cloud Storage, so you will want to regularly delete them to avoid charges. Cloud Storage offers the Object Lifecycle Management feature to set a Time to Live (TTL) for objects, archive older versions of objects, or "downgrade" storage classes of objects to help manage costs.

## Task 6: Server maintenance
To perform server maintenance, you need to shut down the server.

### Connect via SSH to the server, stop it and shut down the VM

1. In the mc-server SSH terminal, run the following command:
```bash
sudo screen -r -X stuff '/stop\n'
```

2. Exit the mc-server SSH terminal.
```bash
exit
```

3. To stop mc-server VM instance, run the following command:
```bash
gcloud compute instances stop mc-server --zone=us-central1-a
```

> To start up your instance again, visit the instance page and then click Start. To start the Minecraft server again, you can establish an SSH connection with the instance, remount your persistent disk, and start your Minecraft server in a new screen terminal, just as you did previously.

### Automate server maintenance with startup and shutdown scripts
Instead of following the manual process to mount the persistent disk and launch the server application in a screen, you can use metadata scripts to create a startup script and a shutdown script to do this for you.

1. To add the startup and shutdown scripts to the VM instance, run the following command:
```bash
gcloud compute instances add-metadata mc-server --zone=us-central1-a \
--metadata=startup-script-url=https://storage.googleapis.com/cloud-training/archinfra/mcserver/startup.sh,\
shutdown-script-url=https://storage.googleapis.com/cloud-training/archinfra/mcserver/shutdown.sh
```

2. Restart the VM instance.
```bash
gcloud compute instances start mc-server --zone=us-central1-a
```

> When you restart your instance, the startup script automatically mounts the Minecraft disk to the appropriate directory, starts your Minecraft server in a screen session, and detaches the session. When you stop the instance, the shutdown script shuts down your Minecraft server before the instance shuts down. It's a best practice to store these scripts in Cloud Storage.

## Task 7: Review
In this lab, you created a customized virtual machine instance by installing base software (a headless JRE) and application software (a Minecraft game server). You customized the VM by attaching and preparing a high-speed SSD data disk, and you reserved a static external IP so the address would remain consistent. Then you verified availability of the gaming server online. You set up a backup system to back up the server's data to a Cloud Storage bucket, and you tested the backup system. Then you automated backups using cron. Finally, you set up maintenance scripts using metadata for graceful startup and shutdown of the server.