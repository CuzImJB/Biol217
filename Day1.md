# Day1

## Simple Commands
pwd "Path working directory" = shows current working directory

mkdir = creates new directory

cd = changes current directory

ls = list 
<br> ls -l = longer list

mv = moves files
* mv file.txt dir/ = move to another directory and keep the name
* mv file.txt file2.txt = renames file
* mv file.txt file2.txt dir/ = rename and move

cp = copy

 XX*= XX1,XX2,XX3 = placeholder to select everything that fits

 rm = remove files

 cat = print the contents of a file
 * cat> = overwrite content of a file
 * cat>> = add content to a file

 wget = download files from websites

 gunzip = unpack zipped files
 <br> zip = zip uncompressed files


 ## CAUCluster

 Connect to the supercomputer of the CAU for more complex codes and more data 

 ssh -X sunam227@caucluster.rz.uni-kiel.de
<br> alwayls change working directory to $WORK <br> cd$WORK

## Micromamba
Smaller Environments or Rooms for different codes
<br> Better for codes

## Bash Scripts
Documentation and easier writing of code --> these are published in papers

## For Loops
for x in [ITEMS]
<br> do
<br>   [COMMAND]
done


