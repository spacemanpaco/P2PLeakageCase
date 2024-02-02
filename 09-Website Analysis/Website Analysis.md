Website Analysis

From the previous step, from the web history we were able to determine a website called Social Upload was visited by the user. The URL for this website was http[:]//dfir-projects.boards[.]net/. We will visit this website and determine if the files of interest were uploaded.

firefox --new-window 'http://dfir-projects.boards.net/' 

Navigate to File Sharing Section --> Look for thread called New music, this was the thread created by the user. It appears to be the most recent post --> Click on the post's title to open post --> Click on the link to download the file, however only registered users will be allowed to download. Username is lab and password is illegal_download_lab --> Download the torrent and move the torrent file to the Web_History folder then perform a MD5 hash check to verify this file is the same one extracted from the user's system. We will also verify when this torrent file was created.

mv ~/Downloads/Sample-1.mp3.torrent .

md5sum Sample-1.mp3.torrent && md5sum ../Torrents/Sample-1.mp3.torrent

wine /home/kali/Forensic_Tools


Now that we have verified that the file that was uploaded to the site has the same creation time as the one found on the user's system, we need to investigate the user on the website and see if it has any linkage to the suspect. The user blowtorch is the one who uploaded the file, the same file that was found on the device, onto the Social Upload website on 21-March 2021 at 20:42(UTC). The timestamps from the Edge Browser History file suggests the suspect was the one who created the forum post and uploaded the file. The linkage between the user blowtorch and the suspect is the file signature is the same for the website and on the system.

To compile all the artifacts we have found, we will create a timeline of these events from the beginning of the incident to the end.