USN Journal Timeline

The Update Sequence Number (USN) Journal keeps a record of changes made to files and folders in a volume. The USN Journal is hidden and located in the $Extend directory. The changes that are recorded are when a file/folder is opened, closed, renamed, created, deleted etc. 

The location of the USN Journal can be found the MFT.csv that was created in the previous step. You will see that the $UsnJrnl is an empty file with also an alternate date stream (ADS)named $J. 

grep -F '$Extend' mft.csv| grep -i 'jrnl' | cut -d ',' -f 1,3,4,8

An ADS is hidden data inside of a file on an NTFS volume. Users may be able to leverage ADS to hide compromising information and/or malicious executables.

Now that we have the location of the USN Journal, we need to obtain the data through a USN Record Carver python script (usncarve.py). We will create a directory to store the USN Journal data into. We will also show how to import the python script to use. The USN Journal parser (usn.py) will than be able to parse the USN entries to view in a CSV. Then LibreOffice can help view the CSV.

mkdir USNJournal

cd USNJournal

pip install usncarve

usncarve.py -f ../../Case_materials/Disk_Image_ID-20210327.001 -o usnjournal

pip install usnparser

usn.py -f usnjournal -c -o usn.csv

libreoffice usn.csv

The reason columns, showing why the entry was recorded, corresponds with the specific timestamps. Now we can use the previous utilities of usn.py to create a body file which mactime can use to create a timeline from. We will than format the CSV into relevant columns.

usn.py -f usnjournal -b -o usn.body

mactime -d -b usn.body -m -z UTC 2021-03-10..2021-03-28 >> usn_timeline_original.csv

echo -e 'Date and Time\tRecord/Inode\tFile Name' >> usn_timeline_condensed.csv

strings usn_timeline_original.csv | grep -E 'Contraband|Sample-|torrent|Thunderbird|List-' | cut -d ',' -f 1,7-8 >> usn_timeline_condensed.csv

libreoffice usn_timeline_condensed.csv

Now we can analyze the USN timeline for files of interest through the command line. Searching for 'Contraband' related terms, shows a number of files that have been created on 27-March 2021. Seeing these instances of files related to 'Contrabnd' allows us to see the changes that have been made to them and we are able to reference them to verify what changes could have been made to them. Compared to the MFT timeline, we can see specific entries for file activity in the USN timeline. MFT provides only  the time certain files were created, while USN includes deletion or renaming of files. However, the USN timeline does not include the file path, therefore the MFT timeline should be referenced with USN timeline to confirm locations of files of interest.

grep -F 'Contraband' usn_timeline_condensed.csv|more -5

Next we will look at Torrent log files and create readable data from those files.