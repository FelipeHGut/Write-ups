# TryHackMe Corridor Write-up

Below is my write-up for the TryHackMe room [Corridor](https://tryhackme.com/room/corridor).

The machine is an easy machine to break.

We first start up the machine and run a `nmap` to see if there is 

```bash

```


```python

import hashlib,string

list_of_hashes = [
"c4ca4238a0b923820dcc509a6f75849b",
"c81e728d9d4c2f636f067f89cc14862c",
"eccbc87e4b5ce2fe28308fd9f2a7baf3",
"a87ff679a2f3e71d9181a67b7542122c",
"e4da3b7fbbce2345d7772b0674a318d5",
"1679091c5a880faf6fb5e6087eb1b2dc",
"8f14e45fceea167a5a36dedd4bea2543",
"c9f0f895fb98ab9159f51fd0297e236d",
"45c48cce2e2d7fbdea1afc51c7c6ad26",
"d3d9446802a44259755d38e6d163e820",
"6512bd43d9caa6e02c990b0a82652dca",
"c20ad4d76fe97759aa27a0c99bff6710",
"c51ce410c124a10e0db5e4b97fc2af39"

]

# creates a string with all with all ascii letter and digits
all_characters = string.ascii_letters+string.digits

## For each character encode as a md5 hash if found on the list of hashes print out the character 
for character in all_characters:
    if hashlib.md5(character.encode()).hexdigest() in list_of_hashes:
        print(character)

```




```python
import hashlib,string


# creates a string with all with all ascii letter and digits
all_characters = string.ascii_letters+string.digits

# for each character
wordlist_file = open("wordlist.txt","w")

# We can create a workd list using all the chararcters to use as a wordlist.
for character in all_characters:
    print(hashlib.md5(character.encode()).hexdigest())
    wordlist_file.write(hashlib.md5(character.encode()).hexdigest()+"\n")
wordlist_file.close()

```



