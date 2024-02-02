MFT Timeline

To get a better understanding of the activities surround the user, a timeline needs to be generated so that the investigation has a logical flow according to the artifacts we find. We will need to use the Master File Table (MFT) to find these artifacts. The MFT keeps records of files and folders in the volume, and is kept in a hidden file called $MFT. To be able to pull the MFT records, we require the offset number in sectors and the inode number, instead of the file name. The inode number can be identified in the output of the fls command while grepping for MFT. Then the icat command will extract the MFT with the inode number.

fls -o 104448 Disk_Image_ID-20210327.001| grep MFT

icat -o 104448 Disk_Image_ID-20210327.001 0-128-6 >> ../Exports/MFT/MFT

To analyze the MFT, AnalyzeMFT.py script will be used to parse and display the file data. First install the AnalyzeMFT.py script and than run the script to create the output in a CSV file. To view the csv use LibreOffice application.

pip install analyzeMFT

analyzeMFT.py -f MFT -o mft.csv --bodyfull

libreoffice mft.csv

The record number shows the inode number/entry ID for its respective file or folder. A file or folder is considered active if it is not deleted and inactive if it is deleted. You will also see two sets of timestamps $SI Info and $FN Info. The $SI (Standard Info) are timestamps that the user sees in file explorer and can be manipulated by the user. The $FN (File Name) timestamps are not seen in file explorer and can only be modified by the system kernel. We can compare the two sets of timestamps to see if they were tampered in any way. We will also search for the files of interest in the MFT.

The first command pulls the file information from the MFT relating to the MP3 file in the Torrent-Sources folder that was found. The second command pulls details on the creation, modified, access and changed timestamps.

head -1 mft.csv| cut -d ',' -f 1,3,4,8 && cat mft.csv| grep -E 'Contraband|Sample-|List-|torrent|Torrent' | cut -d ',' -f 1,3,4,8 | more -1

head -1 mft.csv| cut -d ',' -f 8-16 && cat mft.csv| grep -E 'Contraband|Sample-|List-|torrent|Torrent' | cut -d ',' -f 8-16 | more -2

Reviewing the CSV we can see there is alot of activity on this device during 10-March 2021 to 27-March 2021. Most likely, this is when the activity relevant to the investigation began so that will be our date range to create a timeline from.

The command line tool mactime can create timelines from body files, by specifying the dates you want to the tool to create a timeline out of. The analyzeMFT.py script will be used to generate a body file which can be used by mactime tool.

analyzeMFT.py -f MFT -b mft.body --bodyfull

mactime -d -b mft.body -m -z UTC 2021-03-10..2021-03-28 >> mft_timeline_original.csv

The CSV has to become structured with timestamp columns and merged together to be able to read through it easier. Than we can use LibreOffice to view the structured data.

echo -e 'Date and Time\tFile' >> mft_timeline_condensed.csv

cat mft_timeline_original.csv| grep -E 'Contraband|Sameple-|torrent|Torrent|Thunderbird|List-' | cut -d ',' -f 1,8 >> mft_timeline_condensed.csv 

libreoffice mft_timeline_condensed.csv

Next we will analyze the USN Journal and create a timeline based on that data.