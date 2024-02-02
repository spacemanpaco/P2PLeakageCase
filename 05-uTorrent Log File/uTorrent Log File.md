uTorrent Log File

To torrent a file, a user needs a Torrent Client/Application, Torrent file, the original file to create the torrent from and a method of download/transferring the file either through email, download link on a website, cloud etc. 

Currently the suspected user has a Torrent Client - uTorrent, Torrent file - Sample-1.mp3.torrent, Contraband.mp3.torrent and the original file to create the torrent from - Sample-1.mp3, Contraband.mp3. 

When user is downloading a torrent, or torrenting, a tracker server determines who wants to download and/or who is seeding the file the user wants to download, these are called 'peers'. Seeding occurs after the torrenting, when the user makes the file available for other users to download. A peer is a computer that is downloading or seeding a torrent file.

When torrenting, the torrent client will process the torrent by looking at the extension of the file being torrented. If a file is called 'example.mp3.torrent', then the torrent client will download 'example.mp3'. To download a file, the torrent client will take pieces of 'example.mp3' from peers that are downloading or seeding. Once the download is completed, the user will begin seeding the file to other users who may be downloading 'example.mp3'

We will utilize RegRipper to see what torrenting application exists on the device. Than we will use grep command to find the relevant file paths for the torrenting application in our Reference Files directory. 

rip.pl -r NTUSER.DAT -p uninstall

cd ../../Reference_Files/

grep -F 'uTorrent.exe' file_directory_original.csv|cut -f 1,2

We see that an uninstall key was created for uTorrent application on 21-March 2021 at 20:39(UTC). A uTorrent.exe is on the system's Deskstop directory and in the Users/Kamryn/AppData/Roaming/uTorrent. Now that we have identified the torrent applications on the system, we need to search for torrent files. 

grep -F '.torrent' file_directory_original.csv|cut -f 1,2

From here we can see two torrent files, four in total. Two reside in the User's Downloads/Torrents directory and the other in the User's AppData/Roaming/uTorrent directory. With these files being identified, we need to determine the original file, what trackers are on the torrent file, what application created it and when was it created. We will need to create a Torrents folder to store our findings. icat command will extract the torrent files from the image.

mkdir ../Exports/Torrents

icat -o 104448 ../Case_materials/Disk_Image_ID-20210327.001 55634-128-4 >> ../Exports/Torrents/Contraband.mp3.torrent

icat -o 104448 ../Case_materials/Disk_Image_ID-20210327.001 206280-128-4 >> ../Exports/Torrents/Sample-1.mp3.torrent

We need to verify the files we extracted have been extracted properly so a hash match must be conducted on both files. Both MD5 and SHA1 checks are recommended.

md5sum Sample-1.mp3.torrent && md5sum /media/kali/E8DE4350DE4315EA/Users/Kamryn/Downloads/Torrents/Sample-1.mp3.torrent

md5sum Contraband.mp3.torrent && md5sum /media/kali/E8DE4350DE4315EA/Users/Kamryn/Downloads/Torrents/Contraband.mp3.torrent

sha1sum Sample-1.mp3.torrent && sha1sum /media/kali/E8DE4350DE4315EA/Users/Kamryn/Downloads/Torrents/Sample-1.mp3.torrent

sha1sum Contraband.mp3.torrent && sha1sum /media/kali/E8DE4350DE4315EA/Users/Kamryn/Downloads/Torrents/Contraband.mp3.torrent

The Torrent File Editor exists in the Forensics Tools directory as part of the lab setup. However you may still run into some errors when attempting to launch the file editor. Along with the File editor launnch command, there is configuration steps needed before you can launch the GUI. Once the GUI is launched, you will need to open the torrent files, Sample-1.mp3.torrent and Contraband.mp3.torrent, that were extracted.

sudo dpkg --add-architecture i386

sudo apt update

sudo apt install wine32

sudo apt upgrade (optional)

wine /home/kali/Forensic_Tools

We can see the application that created these Torrent files and when they were created. We see that Sample-1.mp3.torrent was created by uTorrent application found on the system. However, Contraband.mp3.torrent was created by another Torrent application not found on the device, which could mean the file was put on the device or sent to the user. The application also has a log file named resume.dat and we need to find the file path for this file using grep command.

grep -F 'resume.dat' file_directory_original.csv| cut -f 1,2

The log file will tell us the IP address and port numbers for each peer and any torrents that were added to the application. First we need to extract the log file to be able to analyze it. As with any extraction, we need to validate it is a genuine copy with a hash match. The Tree tab in the Torrent File Editor will give you the details on the torrents added to the uTorrent application. We will need to create a text file to put the peer HEX data into. The hex column must be checked, than the data can be copied and pasted for both torrent files.

icat -o 104448 ../Case_materials/Disk_Image_ID-20210327.001 85683-128-4 >> ../Exports/Torrents/resume.dat

md5sum resume.dat && md5sum /media/kali/E8DE4350DE4315EA/Users/Kamryn/AppData/Roaming/uTorrent/resume.dat

nano peers_hex_format.txt

For each line the first 8 characters are split four groups of two characters. So 0a00020f would be 0a 00 02 0f, this would be the IP in HEX. The port number would be the last four characters but the third and fourth character would come before the first and second i.e.  ba3c would be 3cba.

Peers for Contraband.mp3.torrent
00000000000000000000ffff0a00020fba3c --> 0a 00 01 0f 3cba
00000000000000000000ffff7f000001ba3c --> 7f 00 00 01 3cba
00000000000000000000ffff49d468e951c7 --> 49 d4 68 e9 c751
00000000000000000000ffff458f197eba3c --> 45 8f 19 7e 3cba


Peers for Sample-1.mp3.torrent
00000000000000000000ffff0a00020fba3c --> 0a 00 02 0f 3cba
00000000000000000000ffff7f000001ba3c --> 7f 00 00 01 3cba

HEX to decimal conversion is possible through Python, to achieve this a For Loop will be created. An an alternative, you can use an online conversion calculator as well such as RapidTables. The pair bytes will need to be put in one at a time for this site.

#Manual Python input
>>> print("this result is for line 1 under contraband.mp3.torrent.")
this result is for line 1 under contraband.mp3.torrent.
>>> peer1 = ["0a", "00", "02", "0f", "3cba"]
>>> for x in peer1:
...     result = int(x,16)
...     print(result)
... 
10
0
2
15
15546 

>>> print("this result is for line 2 under contraband.mp3.torrent.")
this result is for line 2 under contraband.mp3.torrent.
>>> peer2 = ["7f", "00", "00", "01", "3cba"]
>>> for x in peer2:
...     result = int(x,16)
...     print(result)
... 
127
0
0
1
15546

>>> print("this result is for line 3 under contraband.mp3.torrent.")
this result is for line 3 under contraband.mp3.torrent.
>>> peer3 = ["49", "d4", "68", "e9", "c751"]
>>> for x in peer3:
...     result = int(x,16)
...     print(result)
... 
73
212
104
233
51025

>>> print("this result is for line 4 under contraband.mp3.torrent.")
this result is for line 4 under contraband.mp3.torrent.
>>> peer4 = ["45", "8f", "19", "7e", "3cba"]
>>> for x in peer4:
...     result = int(x,16)
...     print(result)
... 
69
143
25
126
15546

#Output of python script
this result is for line 1 under Sample-1.mp3.torrent.
10
0
2
15
15546

this result is for line 2 under Sample-1.mp3.torrent.
127
0
0
1
15546



Above are two ways you can run the script, either manually enter each peer entry and edit after the output is given or put the entries all into one script to be able to get a faster output.  Now that we have these conversions, we can compile the IPs into a text file for easier reference. We will also need to find the IP address of the device's user to identify activity related to the user. RegRipper tool will be able to extract that information from the Registry Files directory.

rip.pl -r SYSTEM -p ips

From the information gathered, we can see that the user's IP address is 10.0.2.15, and the log file shows the port number being used is 15546. The IP 127.0.0.1 appears to be a loopback address and IP 69.143.25.126 may also belong to the user. The only IP left is 73.212.104.233, which belongs to a user who also has the Contraband.mp3 file and was the peer for the file. 

To summarize our findings, the user may have been the sole seeder for Sample-1.mp3.torrent as there was no peers recorded for this file, so the user was the creator of the file. Contraband.mp3.torrent was downloaded from somewhere else as the torrent client that made it was qBittorrent while the use was using uTorrent. Furthermore, this means that more than one person has the copyrighted Contraband.mp3, as the peer information shows an external IP of 73.212.104.233. 

Next we will look at the binary signatures of these files to validate their origin and file type.