Registry and File Directory

Now that we have verified the collection has not been changed during the process, we need to view the disk image. To view the disk image, first it needs to be mounted. A script will be created that will mount the disk image for us. 

#!bin/bin/bash
#Mount Disk_Image_ID-20210327.001

#Echo cmd will tell us what disk image is being mounted
echo "Disk_Image_ID-20210327.001"

#losetup cmd will automatically mount disk image
sudo losetup --partscan --find --show --read-only ~/Illegal_download_case/Case_materials/Disk_Image_ID-20210327.001

After running this script, the volume will appear on the desktop. Double clicking on the volume will prompt for a password, this will be your user password set for the Kali Linux account. After authenticating, the partition will appear and the serial number can be matched as the second partition for the device.

We will need to export files for evidence so export folders will be created to be used for storage.

mkdir Exports && mkdir Exports/Registry_Files

Next we need to copy the necessary Registry hives that are critical for this investigation such as SAM, SECURITY, SOFTWARE, SYSTEM and NTUSER.DAT. Using the copy command, the path for the volume's registry is given and the export folder path that will be copied into.

cp /media/kali/E8DE4350DE4315EA/Windows/System32/config/SAM Exports/Registry_Files

cp /media/kali/E8DE4350DE4315EA/Windows/System32/config/SECURITY Exports/Registry_Files

cp /media/kali/E8DE4350DE4315EA/Windows/System32/config/SOFTWARE Exports/Registry_Files

cp /media/kali/E8DE4350DE4315EA/Windows/System32/config/SYSTEM Exports/Registry_Files

cp /media/kali/E8DE4350DE4315EA/Users/Kamryn/NTUSER.DAT Exports/Registry_Files

Time is important in investigations because we need to be able to create a timeline of events from beginning to end, so we need to identify what the system's timezone is set to. This gives a better understanding if we need to account for timezone differences or if the system is configured for Coordinated Universal Time (UTC). RegRipper is a tool that can parse registry hives to obtain relevant information from the selected hive. The SYSTEM hive contains the time information for the device so we will use RegRipper on that.

rip.pl -r SYSTEM -p timezone

The output shows the Timezone name as well as the offset during Daylight Savings time. The Timezone for this system shows Pacific Standard Time (PST) which is 8 hours behind UTC and Pacific Daylight Time (PDT) which is 7 hours behind UTC. This device was collected from Maryland, USA, while this timezone is used in western USA. Since Daylight Savings was active when this image was acquired, we have to assume the timestamps from the system could be in PDT.

Next the system's OS version, name and user accounts will be identified.

rip.pl -r SOFTWARE -p winver

rip.pl -r SYSTEM -p compname

rip.pl -r SAM -p samparse | grep -E 'Username|Created|Date' --color=none 

Below is the relevant information about the suspect that has been pulled from registries by RegRipper.

 
User Account​                       Kamryn​
User Account Creation Date​         03/10/2021 00:13:56​
User Account Last Login Date​       03/21/2021 20:04:35​
Operating System (OS)​              Windows 10 Home​
OS Build​                           19041​
System Name​                        DESKTOP-E4SUNT2​
System Owner​                       Kamryn​
System Timezone​                    Pacific Standard Time (PST)​
System Install Date & Time​         03/10/2021 00:04:29​


Although RegRipper has been successful in getting us the relevant information, it is recommended to validate your findings with other tools. Hivexsh is a command line tool to manually analyze Registry files. 


Now we need to identify the last account to logon. This gives us an idea of the types of activity the last user was engaged in and it allows us to tie the activity to the suspect if it applies. 

rip.pl -r SOFTWARE -p lastloggedon

We were able to identify this information from looking up user accounts on the system but this gives a more succint output for the exact user that was last logged in.

Identifying when the computer was last shut down is crucial because there could have been alot of data loss in the time the system was shutdown and when the image could be collected. Using the last logged in user information and last shutdown, we can see if it was deliberately shutdown or left alone to go into sleep or hibernation mode. Windows Events logs would help verify this information on user login activity.

rip.pl -r SYSTEM -p shutdown

Identifying the recent documents/files that the suspected user opened can give us files of interest that may need to be investigated further. 

rip.pl -r NTUSER.DAT -p recentdocs

From here we can see the files that the user interacted with the most. We need to create a directory listing of the partitions to be able to investigate the recent documents easier. The fls command will generate a directory listing based on the partition offset. We need to specify the starting offset for this partition to be able to get a directory listing and direct the output into a readable CSV file.

fls -o 104448 Case_materials/Disk_Image_ID-20210327.001 -r -l -p -z UTC >> Reference_Files/file_directory_original.csv

Once completed, we can use grep command to pull the relevant document extensions to search for those files. Using the recentdocs plugin for RegRipper, we were ablt to find files the user recently accessed. We found torrent application, torrent files, MP3 files with similar names to the torrents, an email client and web browsers. These artifacts also have timestamps of when the user last accessed and changed the files.

grep -F ".exe" file_directory_original.csv| grep -i "torrent" | cut -f 1,2,4,5

grep -F ".mp3" file_directory_original.csv| grep -i "torrent" | cut -f 1,2,4,5

grep -F ".mp3" file_directory_original.csv| grep -iE "contraband|sample-" | cut -f 1,2,4,5

grep -F ".txt" file_directory_original.csv| grep -F "List-" | cut -f 1,2,4,5


grep -F ".exe" file_directory_original.csv| grep -iE "outlook|thunderbird|mail" | cut -f 1,2,4,5

grep -F ".exe" file_directory_original.csv| grep -iE "msedge|firefox|chrome|iexplore" | cut -f 1,2,4,5

To make the output for fls more structured, we need to use the echo command with a number of different flags to be able to make it more ledgible.

echo -e 'File Type and MetaData\tName\tLast Modified\tLast Accessed\tLast Changed\tCreated' >> file_directory_modified.csv

cat file_directory_original.csv >> file_directory_modified.csv 

head -2 file_directory_modified.csv

Next we will analyze the MFT record table and create a timeline from that file.