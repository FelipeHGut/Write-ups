# Tryhackme - Ninja Skills Writeup

The following is my write up for [THM Ninja Skills](https://tryhackme.com/room/ninjaskills). Overall a good little small room to practice unix commands. I took a different approach as I wanted to pratice some bash scripting.

# Task

Let's have some fun with Linux. Deploy the machine and get started.
This machine may take up to 3 minutes to configure.
Answer the questions about the following files:

- 8V2L
- bny0
- c4ZX
- D8B3
- FHl1
- oiMO
- PFbD
- rmfX
- SRSq
- uqyw
- v2Vb
- X1Uy

The aim is to answer the questions as efficiently as possible.

Answer the questions below

Which of the above files are owned by the best-group group(enter the answer separated by spaces in alphabetical order)
>D8B3 v2Vb

Which of these files contain an IP address?
>oiMO

Which file has the SHA1 hash of `9d54da7584015647ba052173b84d45e8007eba94`

>c4ZX

Which file contains 230 lines?

>bny0

Which file's owner has an ID of 502?

>X1Uy

Which file is executable by everyone?

>8V2L

# bash script

I focused on creating a bash that would return all my solutions. The only file not returned was file containing 230 lines. It didn't exist in my instance. 


```bash 


## !/bin/bash

## generate a list
listfiles="8V2L bny0 c4ZX D8B3 FHl1 oiMO PFbD rmfX SRSq uqyw v2Vb X1Uy";
echo "Start of Program"
echo ""


## Variables required for questions
hash='9d54da7584015647ba052173b84d45e8007eba94'
linesans=230
id502=$(cat /etc/passwd | grep '502' | cut -d ':' -f1)
groupcheck='best-group'


## We loop throught the file names in $listfiles
for name_file in ${listfiles};

do
#echo "loop start"
#echo $name_file;
#echo $file_lines;
location=$(find / -type f -name *$name_file* 2>/dev/null);

if [ -z "$location" ];

then 

    echo $name_file "not found";

else
    # For each of those files we get the locations into a variable.
    # This will allow us to run checks to each of the questions.
    file_info=$(ls -la $location);

    # Checks line count and cut provides just the number of lines.
    lines_count=$(wc -l $location|cut -d' ' -f1)
    #echo $lines_count;
    
    ##Checks if the lines matches our #linesans
    if [[ $((lines_count)) = $((linesans)) ]] ;
    then
        echo "$name_file has $lines_count lines";
    fi
    
    ## Checks for the same SHA1 Hash
    shacheck=$(sha1sum $location | cut -d' ' -f1);
    #echo $shacheck;
    if [[ $shacheck = $hash ]];
    then
    echo "$name_file has sham1sum $hash"
    fi

    ##Gathers the for the groupowner of the file
    group_owner=$(echo $file_info | cut -d' ' -f4);

    ##Gathers the for the owner of the file
    owner=$(echo $file_info | cut -d' ' -f3);

    #echo $group_owner;
    if [[ $groupcheck = $group_owner ]];
    then
        echo "$groupcheck Group owns $name_file";
    fi
    #echo $owner;
    if [[ $owner = $id502 ]];
    then
        echo "$id502 owns $name_file";
    fi

    ### checks if the file is executable by all
    executable=$(echo $file_info | cut -c 4,7,10)
    if [[ $executable = "xxx" ]];
    then 
        echo "Excutable by all $name_file";
    fi

    # Checks if the file contains IP addresses
    ip_address=$(cat $location | grep -E -o "([0-9]{1,3}[\.]){3}[0-9]{1,3}");
    if [ -n "$ip_address" ];
    then
        echo "IP addresses in $name_file";
    fi


fi
done
echo ""
echo "End of Program"

```