File Signature

Although we have found the files on the user's system, we have to validate that these are the exact files that the company has. We will grep for MP3 files from our Reference Files directory. Than use icat command to extract the MP3 files and the MD5sum command to verify we extracted the genuine copies. We will compare the extracted files with the mounted image and the hash reference set we created.

grep -F '.mp3' file_directory_original.csv| cut -f 1,2

mkdir ../Exports/MP3_Files

icat -o 104448 ../Case_materials/Disk_Image_ID-20210327.001 205963-128-5 >> ../Exports/MP3_Files/Sample-1.mp3

icat -o 104448 ../Case_materials/Disk_Image_ID-20210327.001 52901-128-4 >> ../Exports/MP3_Files/Contraband.mp3

md5sum Sample-1.mp3 && md5sum /media/kali/E8DE4350DE4315EA/Users/Kamryn/Downloads/Sample-1.mp3

md5sum Sample-1.mp3 && md5sum /media/kali/E8DE4350DE4315EA/Users/Kamryn/Downloads/Torrent-Sources/Sample-1.mp3

md5sum Contraband.mp3 && md5sum /media/kali/E8DE4350DE4315EA/Users/Kamryn/Downloads/Torrent-Sources/Contraband.mp3

md5sum Contraband.mp3 && md5sum /media/kali/E8DE4350DE4315EA/Users/Kamryn/Documents/Reference/Work/Contraband.mp3

md5deep Sample-1.mp3 -bewM ../../Case_materials/hash_reference.txt

md5deep Contraband.mp3 -bewM ../../Case_materials/hash_reference.txt

Validating the files has allowed us to move onto the next step, which is useing the exiftool command to read the metadata of the files. Although a file may have a certain extension in its name such as .mp3 in this case, we have to make sure it is not being disguised as another file. To validate our metadata findings, we will use the strings command to see the binary signature of the files to search for any potential authors.

exiftool /media/kali/E8DE4350DE4315EA/Users/Kamryn/Downloads/Sample-1.mp3

exiftool /media/kali/E8DE4350DE4315EA/Users/Kamryn/Downloads/Torrent-Sources/Contraband.mp3

strings -t x -f * | grep 'Beat Step'

Now that we have been given the offset in hex of where the strings are located that may relate to the author, we can verify this further with hex editor.

hexedit Sample-1.mp3

hexedit Contraband.mp3

Both files match the names and hash code given to us by the company. Sample.mp3 is showing the author as Beat Step and has no binary signature while Contraband.mp3 does have a binary signature. Contraband.mp3 is the copyrighted file with binary signature.

Next we will look into the email communication of the user.