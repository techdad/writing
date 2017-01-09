# SANS Holiday Hack Challenge 2016

Solutions/answers by Daniel Shaw ("techdad") <danielshaw@protonmail.com>:


## Part 1

#### 1) What is the secret message in Santa's tweets?

`BUG BOUNTY`


#### 2) What is inside the ZIP file distributed by Santa's team?

An Android app: `SantaGram_4.2.apk`


## Part 2

#### 3) What username and password are embedded in the APK file?

```
username: guest
password: busyreindeer78
```


#### 4) What is the name of the audible component (audio file) in the SantaGram APK file?

`discombobulatedaudio1.mp3`


## Part 3

#### 5) What is the password for the "cranpi" account on the Cranberry Pi system?

`yummycookies`


####  6) How did you open each terminal door and where had the villain imprisoned Santa?

#### Terminals

 1. *the "itchy and scratchy" terminal*

    * Using `sudo -l` shows that the commands `tcpdump` and `strings` can be run on the file as the user ID with the permission to do so.
    
    * Then, just using `strings out.pcap |more` gives the 1st half of the password easily, and the second half can be guessed from the first.


2. *the "deep directories" terminal*

    * I just used `find / >/tmp/find.out` to enumerate the entire filesystem by full paths.
    
    * It was then possible to copy and pasting the right paths with quotes  and/or escape characters to get to the `key_for_the_door.txt` file. Running `bash` instead of the login version, also helped with tab command complettion.


3. *the "WOPR" terminal*

    * Just follow the War Games script to the letter... :-)

4. *the "wumpus" terminal*

    * Online searching fund various source versions of the wumpus game that document command-line swithes to set numbers of rooms, bats and pits. Using `strings` on the `wumpus` binary confirmed it suports them too.
    
    * So, then, starting the game with minimum rooms and no bats or pits made it quick and simple to "win".


5. *the "train time machine" terminal*

    * The clue was in the HELP that the pager was `less`.

    * As `less` allows for shell commands, this allows running `bash`. From there, it was posible to analyse the files and extract the password, but even simpler was to just run `./ActivateTrain` each time.

#### Santa

Santa was imprisoned in the past, in 1978, in the Dungeon For Errant Reindeer!

    
## Part 4

####  7) REMOTE TARGETS

1. *analytics.northpolewonderland.com (part 1)*

    * I actually found the /.git with `nmap` *before* finding the username and password in the APK file. And so I just used generated cookies set with the browser's JavaScript console to authenticate as either "guest" or "administrator" right away.

    * The 2nd audio file is of course accessible as soon as you're logged in or authenticated as "guest" from the "MP3" menu link.


2. *dungeon.northpolewonderland.com*

    * For this one, I had to search a lot online for info about the Dungeon/Zork games. I eventually found out about the Dugeon/Zork games havig a debug mode (GDT). From there, I experimented with that mode and it's commands; the useful one being `DT- Display text`.

    * To get all the text parts of the game, I use a quick expect script:
    
    ```
    #!/usr/bin/env expect
    spawn nc 35.184.47.139 11111
    expect ">"
    send "gdt\r"
    
    for {set i 1} {$i < 1027} {incr i 1} {
        expect "GDT>"
        send "dt\r"
        expect "Entry:"
        send "$i\n"
    }
    ```

    Then the instructons to email for the 3rd audio where easily found with `grep` in the ouput captured from above.


3. *dev.northpolewonderland.com*

    * Following instructions/hints from the Elves was the key to this one...
    
    * First follow the hint to use Apktool, modify the smali file and recompile and re-sign (exactly as in the Youtube video); so as to enable debugging in the APK.

    * Next the tweaked app can be run in an Emulator, and a debug JSON post captured for re-posting.

    * Finally noting verbose: false in the output, adding `"verbose":true`, and reposting again leads to a file list including the 4th audio.

    
4. *ads.northpolewonderland.com*

    * This was again a matter of following instructions: Installing 'Tampermonkey' and 'Meteor Miner', and reading the related blog.

    * Browsing all the Routes shown by Meteor Miner, led to a "HomeQuote" Collection under /admin/quotes

    * Then exactly as in the blog, running `HomeQuotes.find().fetch()` in the JavaScript console revealed the 5th audio file link.


5. *ex.northpolewonderland.com*

    * This app used the `php://` filter tequinque, also from the Elves hints and the linked blog article but in a less obvious way.

    * The process was to craft a WriteCrashdump JSON from the APK source or proxying the app, and then to do a ReadCrashdump with `php://filter/convert.base64-encode/resource=` in the `"data":` section.

    * Once that reveals how the crashdump files are read back, you can see how to do further WriteCrashdump posts with embedded PHP code that runs. Which in turn can use php's `system()`, to run any command available to the nginx/www uid, leading to various ways to list the 6th audio file.

6. *analytics.northpolewonderland.com (part 2)*

    * As above, in part 1, generate a cookie to authenticate, but as "administrator". Which gives access to the  /edit.php page to rename queries.

    * However, it doesn't do checking on what parameters are passed, so in addition to updating "name" and "description in the report table, if you append `&query=some SQL query;` in a browser the query is saved for that report.

    * Then re-viewing the report runs the query. Thus the MySQL database can be explored, including the "audio" table, revealing the 7th audio file.

    * As the queries are displayed via `htmlenties`, and the audio is a binary blob, it needs to be retrieved by something like `select to_base64()`, and then the resutling text copied and decoded.

    
#### 8) What are the names of the audio files you discovered from each system above?

1. discombobulatedaudio1.mp3
2. discombobulatedaudio2.mp3
3. discombobulatedaudio3.mp3
4. debug-20161224235959-0.mp3
5. discombobulatedaudio5.mp3
6. discombobulated-audio-6-XyzE3N9YqKNH.mp3
7. discombobulatedaudio7.mp3


## Part 5

#### 9) Who is the villain behind the nefarious plot.

Doctor Who

#### 10) Why had the villain abducted Santa?

In a (failed) attempt to prevent the airing of the 1978 Star Wars Holiday Special

