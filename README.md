# Google Photos Takeout Scripts
This is a guide on how to get your photos out of Google Photos and reorganize them in a clean structure. This takes a while and this is some manual work, but hopefully the commands below will help you get to your goal faster.
Hope this can help some of you figuring out how to get a clean library out of your exports.

All commands below are used with macOS, any linux/unix terminal will also work.
To perform this, you need to be able to navigate through directories with a terminal, but for macOs users, you can also drag a folder from the finder into the terminal to paste its path.

## Extract pictures from Google Takeout
Go to `https://takeout.google.com/`, select Google Photos and export. The process may take a while and Google says it can take up to a few days.  

When I did my first full extract from Google Takeout, I noticed that I had a lot of issues in the directories, duplicates in a lot of folders, photos in an album from the wrong year, etc... So what I ended up doing was to go to the Google Photos interface (https://photos.google.com/) and create an album for each year, just to have this first structure to work from. In Google Takeout, I then selected only these albums for export.

You end up with one folder for each year.
In each year folder, you get all your media with their associated .json file that contains precious information that we will reintegrate into the media itself as EXIF data.

## Duplicate JSON files for -edited photos
Since whenever you edit a picture with Google Photos, it saves a new file and adds `-edited` to the original name, I duplicated all .json files with the -edited extenseion so that when I run the exiftool scripts, -edited pictures would also find a json file to take data from.
```
$ for file in */*.jpg.json; do cp -v "${file}" "${file/%.jpg.json/-edited.jpg.json}"; done
$ for file in */*.JPG.json; do cp -v "${file}" "${file/%.JPG.json/-edited.JPG.json}"; done
```
You can do it for all needed extensions  

## Some JSON files have a shorter name than their associated picture
Some picture have a .json file with a shorter filename than their associated picture. I think Google has a character limit for the .json filename, so if your picture has a longer name, your .json and your picture filenames won't match and we won't be able to get the data from the json into the picture EXIF data. Sometimes I had to remove just 1 character from the maximum I've found so that they match a maximum of 46 characters.

I used a library called `rename` that you can install through `brew` if you're on macOS: `brew install rename`. For Linux users, this command should be already available.
```
$ rename 's/^(.{46}).*(\..*)$/$1$2/' * -n
```
Add `-n` to the command in a folder to see if any file is gonna be shortened and how.  
WARNING: if your file is a .jpg.json, it's gonna cut into the jpg extension as it won't recognize the double extension. Fix these ones manually before.

## Adjust dates of all files
You will need to install `exiftool`  
For macOs, if you have `brew` installed, just install it using the `brew install exiftool` command.
Otherwise, you can download the package here and install it manually: `https://exiftool.org/install.html`  

This command will take the `photoTakenTime { timestamp: '' }` out of the .json associated to a picture and integrate it as EXIF data in the picture as `DateTimeOriginal`
```
$ exiftool -r -d %s -tagsfromfile "%d/%F.json" "-DateTimeOriginal<PhotoTakenTimeTimestamp" --ext json -overwrite_original -progress .
```
Sometimes the file will end with .json instead of .jpg.json, so I also ran this command
```
$ exiftool -r -d %s -tagsfromfile "%d/%f.json" "-DateTimeOriginal<PhotoTakenTimeTimestamp" --ext json -overwrite_original -progress .
```
Also, some files have their .json filename cropped or changed, you will have to check when sorting photos (or run the script below beforehand).

## Fix iPhone Live Photos exported as .JPG
In my specific case, Live photos would be extracted as a .JPG and a (1).JPG instead of a .JPG and a .MOV.
So I ran exiftool to correct this for files that have a filetype MOV.  
The script checks the filetype in the EXIF data of the file. If it's a .MOV, it will change the extension of the file to .MOV
```
$ exiftool -r -ext jpg -overwrite_original -filename=%f.MOV -if '$filetype eq "MOV"' -progress .
```

## Sort pictures by month
Using this lib: `https://github.com/andrewning/sortphotos` (clone it through `git` or just download the code, and then run the installation instructions on the page from a terminal)  

I ran it into every year folder separately so that I could see the result and modify quickly any problem (problems with wrong dates would usually be sent to the last folder created)
```
$ sortphotos --sort %Y/%m --recursive source destination
```
You can use `-copy` if you want to copy instead of moving  
You can use `--use-only-tags EXIF:DateTimeOriginal` if you see some pictures that are not sent to the right year/folder (sortphotos by default uses the earliest date found in the EXIF data file)

## Some useful scripts to use for edge cases
### View EXIF data for individual photos
If you need to adjust EXIF for a specific picture or folder:
```
$ exiftool <FILE/DIRECTORY>
```
### Adjust EXIF for individual photos
If you need to adjust EXIF for a specific picture or folder:
```
$ exiftool -overwrite_original -DateTimeOriginal="1988-02-05 00:00:00" <FILE/DIRECTORY>
```
### Remove all .json files
When you don't need the JSON files anymore 
```
$ find . -name "*.json" -type f -delete
```
### Remove -edited files
On some of the directories, Google used to adjust contrast/colors for every image. I chose to remove that for some folders.
```
$ find . -name "*-edited.jpg" -type f -delete
```