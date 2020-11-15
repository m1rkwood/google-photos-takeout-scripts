# Google Photos Takeout Scripts
List of scripts & commands I used to get out of Google Photos.  
Hope this can help some of you figuring out how to get a clean library out of your exports.

## Extract pictures from Google Takeout
When I did my first extract from Google Takeout, I noticed a lot of issues, duplicates in a lot of albums, etc... So the first thing I did was to go to the Google Photos interface (https://photos.google.com/) and created an album for each year. Google Takeout would then send me one folder per year that was easier to manipulate.

## Duplicate JSON files for -edited photos
Since whenever you edit a picture with Google Photos, it saves a new file and adds `-edited` to the original name, I duplicated all json files with the -edited so that when I run the exiftool scripts, -edited pictures would also find a json file.
```
$ for file in */*.jpg.json; do cp -v "${file}" "${file/%.jpg.json/-edited.jpg.json}"; done
$ for file in */*.JPG.json; do cp -v "${file}" "${file/%.JPG.json/-edited.JPG.json}"; done
```
You can do it for all needed extensions  

## Adjust dates of all files
You will need to install `exiftool` first  
```
$ exiftool -r -d %s -tagsfromfile "%d/%F.json" "-DateTimeOriginal<PhotoTakenTimeTimestamp" --ext json -overwrite_original -progress .
```
Sometimes the file will end with .json instead of .jpg.json, so I also ran this command
```
$ exiftool -r -d %s -tagsfromfile "%d/%f.json" "-DateTimeOriginal<PhotoTakenTimeTimestamp" --ext json -overwrite_original -progress .
```
Also, some file have their JSON name cropped or changed, you will have to check when sorting photos (or run the script below beforehand)

## Fix iPhone Live Photos exported as .JPG
In my specific case, Live photos would be extracted as a .JPG and a (1).JPG instead of a MOV.
So I ran exiftool to correct this for files that have a filetype MOV.
```
$ exiftool -r -ext jpg -overwrite_original -filename=%f.MOV -if '$filetype eq "MOV"' -progress .
```

## Sort pictures by month
Using this lib: `https://github.com/andrewning/sortphotos`
I ran it into every year folder separately so that I could see the result and modify quickly any problem (problems with wrong dates would usually be sent to the last folder created)
```
$ sortphotos --sort %Y/%m --recursive source destination
```
You can use `-copy` if you want to copy instead of moving  
You can use `--use-only-tags EXIF:DateTimeOriginal` if you see some pictures that are not sent to the right year/folder (sortphotos by default uses the earliest date found in the file)

## Some JSON have a shorter name than the picture
Some picture have a JSON file with a shorter name than the picture. Sometimes I had to remove just 1 character from the maximum I've found so that they match (46). Re-run exiftool scripts above afterwards.
```
$ rename 's/^(.{46}).*(\..*)$/$1$2/' * -n
```
Add `-n` to the command in a folder to see if any file is gonna be shortened and how.  
WARNING: if your file is a .jpeg.json, it's gonna cut into the jpeg extension. Fix these ones manually before.

## Adjust EXIF for individual photos
If you need to adjust EXIF for a specific picture or folder:
```
$ exiftool -overwrite_original -DateTimeOriginal="1988-02-05 00:00:00" <FILE/DIRECTORY>
```

## Remove all .json files
When you don't need the JSON files anymore 
```
$ find . -name "*.json" -type f -delete
```

## Remove -edited files
On some of the directories, Google used to adjust contrast/colors for every image. I chose to remove that for some folders.
```
$ find . -name "*-edited.jpg" -type f -delete
```