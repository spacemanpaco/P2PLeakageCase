Web History

We have to identify the browser artifacts for this case because it is a crucial element in determining if there were any browser history that may be related to the case such as when the Torrent application could have been downloaded and were any of the files uploaded to any websites. This helps establish motive for the suspect and create linkage between the evidence and suspect. We identified the browser earlier before when we ran the ReegRipper tool to look for anything that may have been uninstalled. Microsoft Edge was shown in here as the only browser present on the system.

Now we need to identify the location of these Microsoft Edge files in the system and extract the history for analysis.

cd ../../Reference_Files 

cat file_directory_original.csv| grep -iE 'iexplore.exe|msedge.exe'| cut -f 1,2

cat file_directory_original.csv| grep -i 'edge' | grep -F 'History' | cut -f 1,2

cat file_directory_original.csv| grep -F 'WebCache' | cut -f 1,2

After running the above commands, we have confirmed there is only Microsoft Edge and Internet Explorer on the user's system at the time of imaging. Now we must export the Internet Explorer files and use the strings tool to analyze them.

mkdir ../Exports/Web_History

icat -o 104448 ../Case_materials/Disk_Image_ID-20210327.001 94798-128-3 >> ../Exports/Web_History/History

icat -o 104448 ../Case_materials/Disk_Image_ID-20210327.001 94619-128-4 >> ../Exports/Web_History/WebCacheV01.dat

cd ../Exports/Web_History

strings WebCacheV01.dat|grep -i 'http' | more -10

Analysis of Internet Explorer web cache shows nothing of significant value so now Microsoft Edge web history must be analysed. This file is compiled as a sqlite database and will be viewed with the sqlite3 terminal based tool that lets us create and edit sqlite databases. We have to ensure we put the read only flag as to not alter the data. Once the tool is launched, we will have to navigate the tables that are the schemas in which sqlite compiles its data.

sqlite3 History -readonly

.tables

.schema urls

From investigating the URLs table, we see website hyperlinks stored that the user visited. It shows the hyperlink column, title of the webpage if applicable and last time visit for date and time when each visit took place. Next we have to export this table to a CSV file.

sqlite> .header on
sqlite> .mode csv
sqlite> .output edge_history_original.csv
sqlite> select
   ...> title, url, datetime(last_visit_time / 1000000 + (strftime('%s', '1601-01-01')), 'unixepoch')
   ...> from urls;
sqlite> .quit

Having extracted the CSV, we can view it through the LibreOffice GUI or through the terminal by grepping for any references to torrent. 

libreoffice edge_history_original.csv

cat edge_history_original.csv| grep -i 'torrent'

Timeline of Web Activity

On 3/10/2021 –​

- DuckDuckGo search for ‘mozilla thunderbird’ (00:27:42)​
    - DuckDuckGo is a search engine like Google.​
- Web page opens that shows Thunderbird’s download (00:27:54)​
- DuckDuckGo search for ‘utorrent’ (00:28:15)​
- uTorrent website visited (00:28:55)​
- Web page opens that says uTorrent’s download is complete (00:29:50)​
- Suspect visits outlook.com (00:30:25)​
- Suspect is signing into Outlook email (00:30:29)​
- Suspect is signed into the ‘kamalle123@outlook.com’ email (00:34:56)
- DuckDuckGo search for ‘bittorrent tracker list’ (20:22:45)​
- Suspect opens ngosang’s Github repository titled ‘trackerslist: Updated list of public BitTorrent trackers’ (20:22:51)​
- Suspect accesses text files from ngosang’s Github repository:​
    - ‘trackers_best.txt’ (20:24:11)​
    - ‘trackers_all_ip.txt’ (20:24:32)​
    - ‘trackers_best_ip.txt’ (20:25:52)​
- Suspect visits ‘Social Upload’ (20:46:42)​
- Suspect logs into this website (20:48:37)​
- Suspect visits a section of the website called ‘Files’ (20:50:39)​
- Suspect is creating a thread (20:50:42)​
- Suspect is in a thread titled ‘New Music’ (20:53:19)

From this timeline it shows that what email was access on Outlook, the suspect downloaded Thunderbird and uTorrent, suspect accessed a trackerlist on GitHub and they also acessed a site called Social Upload creating a thread named New Music. Although we know these downloads took place, we need to know when the downloads took place. We can also save the GitHub page for our records. 

firefox --new-window 'https://github.com/ngosang/trackerslist'

Right Click --> Save Page As --> in Web_History folder location

Now we must use sqlite3 again to extract the specific downloads table to analyze the date and times these events took place. The downloads table shows us the columns where each entry is recorded, there is target path where the downloaded item is saved to, start time and end time refers the start and end time for the downloads, received bytes shows the bytes downloaded, total size is total bytes downloaded, tab url shows the URL where the download took place and last access time shows when the downloaded file was opened from the browser. We will extract the downloads table into a CSV.

sqlite> .header on
sqlite> .mode csv
sqlite> .output edge_downloads.csv
sqlite> select
   ...> tab_url, target_path, received_bytes, total_bytes,
   ...> datetime(start_time / 1000000 + (strftime('%s', '1601-01-01')), 'unixepoch'),
   ...> datetime(end_time / 1000000 + (strftime('%s', '1601-01-01')), 'unixepoch'),
   ...> datetime(last_access_time / 1000000 + (strftime('%s', '1601-01-01')), 'unixepoch')
   ...> from downloads;
sqlite> .quit

cat edge_downloads.csv| grep -i 'torrent'

With this information we can update the timeline with download history data.

Updated Timeline of Web Activity

On 3/10/2021 –​

- DuckDuckGo search for ‘mozilla thunderbird’ (00:27:42)​
    - DuckDuckGo is a search engine like Google.​
- Web page opens that shows Thunderbird’s download (00:27:54)​
- Thunderbird downloaded​
    - Download started (00:27:58)​
    - Download ended (00:28:08)
- DuckDuckGo search for ‘utorrent’ (00:28:15)​
- uTorrent website visited (00:28:55)​
- Web page opens that says uTorrent’s download is complete (00:29:50)​
- uTorrent downloaded​
    - Download started (00:29:51)​
    - Download ended (00:30:02)
- Suspect visits outlook.com (00:30:25)​
- Suspect is signing into Outlook email (00:30:29)​
- Suspect is signed into the ‘kamalle123@outlook.com’ email (00:34:56)
- DuckDuckGo search for ‘bittorrent tracker list’ (20:22:45)​
- Suspect opens ngosang’s Github repository titled ‘trackerslist: Updated list of public BitTorrent trackers’ (20:22:51)​
- Suspect accesses text files from ngosang’s Github repository:​
    - ‘trackers_best.txt’ (20:24:11)​
    - ‘trackers_all_ip.txt’ (20:24:32)​
    - ‘trackers_best_ip.txt’ (20:25:52)​
- Suspect visits ‘Social Upload’ (20:46:42)​
- Suspect logs into this website (20:48:37)​
- Suspect visits a section of the website called ‘Files’ (20:50:39)​
- Suspect is creating a thread (20:50:42)​
    - A ‘thread’ is a post on a forum that other people can look at and/or respond to
- Suspect is in a thread titled ‘New Music’ (20:53:19)

We see that the timestamps for the downloads are consistent with the entries found in the MFT record. Earlier we identified text files on the system but did not see any download history entries for them. We also saw the suspect was in contact with alleged leakers, and there is a website the suspect visited to create a thread on New Music. This website must be investigated to see if there is evidence of any uploaded files we have recovered.

Torrent trackers are important in finding other peers to help seed the torrent file. Trackers are special type of server that assist in communication between peers in. The process begins when a peer sends a message to the tracker to register its interest in a torrent, the tracker responds with a list of other peers in a TXT file who have previously expressed interest, then the peer connects directly to each of the peers it received from the tracker. The tracker CAN NOT act as a proxy between peers, relay data between peers, see what pieces of the torrent each peer has or see the torrent file or its metadata. It is time to investigate the TXT files and see if they hold any useful information.

We will grep the list name from the MFT then extract the text files.

cd ../MFT

grep -F 'List-' mft_timeline_condensed.csv

mkdir ../Torrent_Tracker_lists

icat -o 104448 ../../Case_materials/Disk_Image_ID-20210327.001 215259-128-1 >> ../Torrent_Tracker_lists/List-1.txt

icat -o 104448 ../../Case_materials/Disk_Image_ID-20210327.001 217188-128-3 >> ../Torrent_Tracker_lists/List-2.txt

icat -o 104448 ../../Case_materials/Disk_Image_ID-20210327.001 218931-128-3 >> ../Torrent_Tracker_lists/List-3.txt

cd ../Torrent_Tracker_lists

head -10 List-1.txt

head -10 List-2.txt

head -10 List-3.txt

Looking at these txt files shows they contain trackers, which were used for the torrents that are on the system. The creation timestamps from the text files are close to when the suspect or torrent client looked at the text files within the browser. It is also possible these trackers came from the GitHub page. From investigation the Web History, we can confirm when the uTorrent client was downloaded by the user, that the trackers may have come from the GitHub page which can also be validated by checking clipboard history from the memory image. We also have the hyperlink to Social Upload, which could mean that if the suspect uploaded any MP3 files, they may have done it from this website.

Next we will analyze the Social Upload website to see if there were any uploads of the music files in this case.