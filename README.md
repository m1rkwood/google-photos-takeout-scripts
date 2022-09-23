# Google Photos Takeout Scripts
This is a guide on how to get your photos out of Google Photos and reorganize them in a clean structure. This takes a while and this is some manual work, but hopefully the commands below will help you get to your goal faster.
Hope this can help some of you figuring out how to get a clean library out of your exports.

All commands below are used with macOS, any linux/unix terminal will also work.
To perform this, you need to be able to navigate through directories with a terminal.  
For macOs users, you can also drag a folder from the finder into the terminal to paste its path faster.
If you're not comfortable with navigating your folders through a terminal, I encourage you to follow a quick tutorial online, it comes fairly easily with a bit of practice.

## Extract pictures from Google Takeout
Go to `https://takeout.google.com/`, select Google Photos and export. The process may take a while and Google says it can take up to a few days.  

When I did my first full extract from Google Takeout, I noticed that I had a lot of issues in the directories, duplicates in a lot of folders, photos in an album from the wrong year, etc... So what I ended up doing was to go to the Google Photos interface (https://photos.google.com/) and create an album for each year, just to have this first structure to work from. In Google Takeout, I then selected only these albums for export.

You end up with one folder for each year.
In each year folder, you get all your media with their associated .json file that contains precious information that we will reintegrate into the media itself as EXIF data.

## Duplicate JSON files for -edited photos
Since whenever you edit a picture with Google Photos, it saves a new file and adds `-edited` to the original name, I duplicated all .json files with the -edited extenseion so that when I run the exiftool scripts, -edited pictures would also find a json file to take data from.

Navigate to the main Google Photos folder and run these commands:
```
for file in */*-edited.jpg; do cp -v "${file/%-edited.jpg/.jpg.json}" "${file/%-edited.jpg/-edited.jpg.json}"; done
for file in */*-edited.JPG; do cp -v "${file/%-edited.JPG/.JPG.json}" "${file/%-edited.JPG/-edited.JPG.json}"; done
```
You can do it for all needed extensions  

## Some JSON files have a shorter name than their associated picture
Some picture have a .json file with a shorter filename than their associated picture. I think Google has a character limit for the .json filename, so if your picture has a longer name, your .json and your picture filenames won't match and we won't be able to get the data from the json into the picture EXIF data. Sometimes I had to remove just 1 character from the maximum I've found so that they match a maximum of 46 characters.

I used a library called `rename` that you can install through `brew` if you're on macOS: `brew install rename`. For Linux users, this command should be already available.

Navigate to the folder of your choice and run this command:
```
rename 's/^(.{46}).*(\..*)$/$1$2/' * -n
```
Add `-n` to the command in a folder to see if any file is gonna be shortened and how.  
WARNING: if your file is a .jpg.json, it's gonna cut into the jpg extension as it won't recognize the double extension. Fix these ones manually before.

## Adjust dates of all files
You will need to install `exiftool`.  
For macOs, if you have `brew` installed, just install it using the `brew install exiftool` command.
Otherwise, you can download the package here and install it manually: `https://exiftool.org/install.html`  

This command will take the `photoTakenTime { timestamp: '' }` out of the .json associated to a picture and integrate it as EXIF data in the picture as `DateTimeOriginal`. See the "useful scripts" section below to find additional tags that you can add to this command to get more data back into your pictures.
```
exiftool -r -d %s -tagsfromfile "%d/%F.json" "-DateTimeOriginal<PhotoTakenTimeTimestamp" --ext json -overwrite_original -progress <directory_name>
```
Sometimes the file will end with .json instead of .jpg.json, so I also ran this command
```
exiftool -r -d %s -tagsfromfile "%d/%f.json" "-DateTimeOriginal<PhotoTakenTimeTimestamp" --ext json -overwrite_original -progress <directory_name>
```
Also, some files have their .json filename cropped or changed, you will have to check when sorting photos (or run the script below beforehand).

## Fix iPhone Live Photos exported as .JPG
In my specific case, Live photos would be extracted as a .JPG and a (1).JPG instead of a .JPG and a .MOV.
So I ran exiftool to correct this for files that have a filetype MOV. The script checks the filetype in the EXIF data of the file. If it's a .MOV, it will change the extension of the file to .MOV
```
exiftool -r -ext jpg -overwrite_original -filename=%f.MOV -if '$filetype eq "MOV"' -progress <directory_name>
```
If you want to remove the `(1)` in the new .MOV files, you can run this in each directory:
```
rename 's/^(.*)\(1\)(\.MOV)$/$1$2/' * -n
```
`-n` doesn't actually rename but shows you what would happen. If the result of the test shown is good for you, replace `-n` by `-v`.

## Sort pictures by month
Using this lib: `https://github.com/andrewning/sortphotos` (clone it through `git` or just download the code, and then run the installation instructions on the page from a terminal)  

I ran it into every year folder separately so that I could see the result and modify quickly any problem (problems with wrong dates would usually be sent to the last folder created)
```
sortphotos --sort %Y/%m --recursive <source_directory> <destination_directory>
```
You can use `-copy` if you want to copy instead of moving  
You can use `--use-only-tags EXIF:DateTimeOriginal` if you see some pictures that are not sent to the right year/folder (sortphotos by default uses the earliest date found in the EXIF data file)

## Some useful scripts to use for edge cases
### View EXIF data for individual photos
If you need to adjust EXIF for a specific picture or folder:
```
exiftool <filename>
```
### Adjust EXIF for individual photos
If you need to adjust EXIF for a specific picture or folder:
```
exiftool -overwrite_original -DateTimeOriginal="1988-02-05 00:00:00" <file or directory>
```
### If you want to integrate more data from the JSON file into the EXIF data
The command below shows you different tags that you can extract from .json files to add to your pictures EXIF data.
```
exiftool -r -d %s -tagsfromfile "%d/%F.json" "-GPSAltitude<GeoDataAltitude" "-GPSLatitude<GeoDataLatitude" "-GPSLatitudeRef<GeoDataLatitude" "-GPSLongitude<GeoDataLongitude" "-GPSLongitudeRef<GeoDataLongitude" "-Keywords<Tags" "-Subject<Tags" "-Caption-Abstract<Description" "-ImageDescription<Description" "-DateTimeOriginal<PhotoTakenTimeTimestamp" -ext "*" -overwrite_original -progress --ext json <directory_name>
```
### Remove all .json files
When you don't need the JSON files anymore 
Navigate to a directory and run (reminder: `.` is the current directory you're in)
```
find . -name "*.json" -type f -delete
```
### Remove -edited files
On some of the directories, Google used to adjust contrast/colors for every image. I chose to remove that for some folders.
Navigate to a directory and run (reminder: `.` is the current directory you're in)
```
find . -name "*-edited.jpg" -type f -delete
```
