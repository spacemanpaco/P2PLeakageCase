Emails

Investigating emails is crucial in forensic investigations as it can reveal information on  files of interest that may have infiltrated or exfiltrated the device or network. We have to investigate the email application that was used by the user and to analyze the emails that were sent. We identified the location of Thunderbird, which is an email client for Mozilla, like Outlook is to Microsoft. Thunderbird can be used to read and/or send email. The file path for the application was 'Program Files/Mozilla Thunderbird/thunderbird.exe' however the emails are stored in a seperate location on the system. 

Change directories into the Registry Files. From there we will use RegRipper command to see when Thunderbird could have been installed. Grepping for uninstall will give us results, as most executables are paired with a uninstall executable as well.

cd ../Registry_Files

rip.pl -r SOFTWARE -p uninstall

From the command above, we can see that 'Mozilla Thunderbird 78.8.1' has an entry in the Registry for 10-March 2021. This suggest it was installed on the system at this date. We will need to analyze link files, shortcut files that point to an application or file, to validate information on how this application was used, if the user or OS created the link, timestamp and any other information that may be of use. The file directory we extracted will contain information on the link file. Once the thunderbird link files are identified, we will use icat to extract them.

cat file_directory_original.csv|grep -F 'lnk' | grep -i 'thunderbird' | cut -f 1,2

mkdir ../Exports/Email

icat -o 104448 ../Case_materials/Disk_Image_ID-20210327.001 93410-128-4 >> ../Exports/Email/'Mozilla Thunderbird (1).lnk'

icat -o 104448 ../Case_materials/Disk_Image_ID-20210327.001 93411-128-4 >> ../Exports/Email/'Mozilla Thunderbird (2).lnk'

The link files related to Thunderbird have been extracted so now we have to analyze them with the lnkinfo command. The information for the link files will show the same except for the access time, even their paths are the same. The creation time also matches up with the MFT file entry. This validates the creation information data we have been given. 

lnkinfo Mozilla\ Thunderbird\ \(1\).lnk

lnkinfo Mozilla\ Thunderbird\ \(2\).lnk

Although we have validated information for the Mozilla Thunderbird email client, we should also investigate any data relating to Microsoft Outlook, as this is a popular email client used widely in organizations. Identifying the emails in inbox, sent and draft folders will help us further in the user's communication if any.

cd ../../Reference_Files

cat file_directory_original.csv| grep -i 'outlook' | grep -iE 'inbox|sent|draft' | cut -f 1,2

icat -o 104448 ../Case_materials/Disk_Image_ID-20210327.001 93677-128-3 >> ../Exports/Email/INBOX

icat -o 104448 ../Case_materials/Disk_Image_ID-20210327.001 93847-128-3 >> ../Exports/Email/Sent-1

icat -o 104448 ../Case_materials/Disk_Image_ID-20210327.001 85034-128-3 >> ../Exports/Email/Drafts-1

These files will be viewed through the Mutt terminal based email application. Please ensure the mailbox file format is in a format supported by Mutt which are mbox, mmdf, mh, and maildir. 

mutt -f INBOX -R 

mutt -f Sent-1 -R

mutt -f Drafts-1 -R

From here we can see there are three important emails in the inbox that pertain to the files of interest in this case. Try this one! contains an attachment for Sample-1.mp3, Re: Try this one! and Here's Another One! contains an attachment for Contraband.mp3.torrent. These emails were sent by a user named Willis Gibbs. According to these emails Gibbs sent the user ‘Sample-1.mp3’ and ‘Contraband.mp3.torrent’ making Gibbs a possible suspect in creating Contraband torrent file. The torrent application that created the Contraband torrent was different than the one found on the user's device.

Important emails in the sent folder are related to the ones found in the inbox. These emails were responses to Gibbs and a message sent to multiple people. Re: Try this one!, Re: Try this one!, Re: Here’s Another One! and Check these files out!​ contain the attachments Contraband.mp3 and Sample-1.mp3. Accordin to these emails, the user acknowledges recieving files from Gibbs and sends ‘Contraband.mp3’ and ‘Sample-1.mp3’ to user IDs realltrain and ram91284.

To summarize, the two employees that the company named are involved in the transmission of the two files that were named. Gibbs emailed the user Sample-1.mp3 and later Contraband.mp3.torrent, and Gibbs verifies Sample-1.mp3 will be presented in a meeting. The user then emails the two files to two people and insinuates that these two users are leakers who could gain reputation in the leaker community, suggesting these files may be posted online or by anyone else they sent the files to.

Next we will look into the web history for the user.