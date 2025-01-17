> /*
>
> PVA is coded by Marius Schwarz in 2021-2022
>
> This software is free. You can copy it, use it or modify it, as long as the result is also published on this condition.
> You only need to refer to this original version in your own readme / license file. 
>
> */


Hi,

This project depends heavily on the AlphaCephei Software VOSK, which can be found here:

https://alphacephei.com/vosk/Why is there 

Language models for 18 different languages can be found here: 

https://alphacephei.com/vosk/models

To run it, you need at least:

```
Python3
pip3
portaudio
mbrola             (found on github)
espeak
vosk               (found on github)
sox                (needed in case you wanne make use of GTTS)
openssl 1.x+       (used to communicate with the PVA TLS Server)
```

# JAVA deps have been removed to prepare for distro packaging

### How to install:

Via RPM REPO:

create /etc/yum.repos.de/pva with this content:

```
[pva]
name=PVA $releasever - $basearch
baseurl=http://repo.linux-am-dienstag.de:80/$basearch/fedora/$releasever/
enabled=1
metadata_expire=1d
#repo_gpgcheck=1
type=rpm
gpgcheck=1
gpgkey=http://repo.linux-am-dienstag.de:80/RPM-GPG-KEY-fedora-$releasever-$basearch
```

Execute the following commands as root:

```
dnf makecache --repo=pva
dnf install pva-base pva-vosk-model-de-small
```

for german speaking persons: that's it. 
All non german users need to a) install a different language model b) translate the configs to theire language first, otherwise it won't work.

Manual installation via Github:

We don't need fancy frameworks or complicated makefiles, so relax, it's just done in about 5 minutes :) The author uses Fedora as OS,
so if you run Ubuntu or Arch, your install commands will vary a bit.

## if you want to use GTTS instead of espeak/mbrola:

You need to checkout gsay, change your config to use gsay as "say" app and install gtts for your distro:

i.E. Fedora: `dnf install gtts`

## WARNING: 
### Your personal privacy is at risk if you use gtts, as the text of the what PVA answers ( which includes repeats of what you said ) is transferred to a Google server, for which you need an active internet connection too. 

## Espeak:

```sudo dnf -y install espeak```

## Mbrola: ( OPTIONAL, but than you have to live with espeak ;) ) 

The say script now checks if mbrola is available. if it's not, it uses native espeak which sounds terrible ;)

Download:

https://github.com/numediart/MBROLA
https://github.com/numediart/MBROLA-voices

Compile: ( needs gcc )

```
cd MBROLA-master
make
cp Bin/mbrola /usr/local/sbin/“
```

Experiment a bit with the best voice for you. You will find the `say` Bash script in this repo, but it will default to de5, as it's setuped for german.
Just replace de5 with your prefered language and don't forget to translate the responses in PVA.java ;)

If you need help with mbrola: https://marius.bloggt-in-braunschweig.de/2021/03/24/mbrola-etwas-bessere-sprachsynthese/

## Python3 + Pip3 + Portaudio:

```pip3 install sounddevice```

ATTN: do not install it as user, if you wanne install it systemwide ( see below on "Installation" )

```sudo dnf install python3-pyaudio espeak```

## Vosk:

On your desktop you can directly install vosk :

```pip3 install vosk```

ATTN: do not install it as user, if you wanna install it systewide ( see below on "Installation" )

If you wanna run this on a Pinephone, as recently demonstrated, you need atleast this BETA version:

```pip3 install https://github.com/alphacep/vosk-api/releases/download/v0.3.30/vosk-0.3.30-py3-none-linux_aarch64.whl```

check if there is a newer version, before you run this command.

Demonstration: http://static.bloggt-in-braunschweig.de/Pinephone-PVA-TEST1.mp4

## Getting the right language model:

Download them here:

https://alphacephei.com/vosk/models

There are small models, with a smaller stock of words, but it may already be enough for your tasks, or you can use the big ones ( ~1.9 GB ), which have a lot more words, but tend to have false positives for exactly this reason :) i.e. in german "füge" and "flüge" sound very similar and this gives us a relativly high error rate. You should use the small model on the Pinephone, due to it's size advantage.

####
ATTN: previouse installation procedures focused on a private, one user only installation. We now focus on a MULTI-USER systemwide installation.
####

PVA is now fully aware of `systemd --user` actions and freedesktop directorystructures

## Installation: 

(replace the modelfile with your filename! )

Become root, sudo alone is not enough!

use `sudo -i` or `sudo su` or just `su` if you still have a root password.

```pip3 install sounddevice vosk```

# NOTE: If you wanna do development work on this, your repo should be located somewhere else, as you will move files out of the repo directories and git won't like that ;)

# Example if you i.e. used a zip archive of pva

```
mkdir -p /usr/share/pva
cd /usr/share/pva
copy the relevant zip content here (means, do not include "master" or something similar in the path)
mv etc/pva /etc/
```

# NOTE: You don't need to "move" files into place, you can of course copy them i.e. with `cp` or `tar c etc | tar x -C /`

Example on the modelfile:

```
unzip vosk-model-small-de-0.21.zip
ln -s vosk-model-small-de-0.21 model

mkdir -p /usr/local/sbin
cp usr/local/sbin/*say /usr/local/sbin 
```

# NOTE: alternative: `tar c usr |tar x -C /`

now compile the JAVA Classes :

```
cd /usr/share/pva
javac PVA.java
```

if you do not have other java apps running with 8 you can use "javac PVA.java" and compile for the latest version. If you see error messages, use:

```
javac --release 8 PVA.java
```

move desktop/* to /usr/share/applications/

You should now find two PVA entries in your desktop app menu.

You can now use your desktop specific autostarttool ( Startprogramme ) to add PVA to your desktop autostart.

You should hear the default ( german ) startup greeting, next time your start your desktop session.
Of course, you can start PVA the first time manually from the app menu ;)

Check the processlist if a process "python3" appeared for pva.py. If not, the startup did not work, something is wrong.

In this case:

switch to your desktop userid

```
cd /usr/share/pva
./pva 
```

If you did not install the german model file, you need to change the /etc/pva/conf.d/01-default.conf to your model name, by overwriting the config in a user.conf file:

the directory "~/.config/pva/conf.d/" should already exists, as you had started PVA via your app menu. The executed "pva" bash script will create all necessary directory and basic config files for you.

In case it does not exist, the PVA start failed much earlier in the chain and you need to check the desktop files in /usr/share/applications/ for more.

To overwrite the config you create this:

```
mkdir -p ~/.config/pva/conf.d/
echo 'vosk:"model","vosk-model-de-0.21"' > ~/.config/pva/conf.d/00-model-overwrite.conf
```

Of course you have to use the model name you need, not this example ;) 

Don't forget to update it everytime you update the model version. 

## Why is it in the config file and not defaulting to the symlink "model" from above?

Because PVA can switch the model on a voice command :) It will be changed on the next app restart.

PVA will search for a matching language model from the list of models located in /usr/share/pva.
To add a model, just download and unpack it there.

Example:

```
$ ls -l /usr/share/pva/
insgesamt 244
-rw-r--r--. 1 root root   478 26. Jan 10:50  AppResult.class
drwxrwxr-x. 3 root root  4096 30. Dez 2020   com
-rw-rw-r--. 1 root root   480 26. Jan 10:50  Command.class
drwxr-xr-x. 2 root root  4096 17. Jul 2021   data
drwxrwxr-x. 2 root root  4096 16. Dez 18:58  hash
drwxrwxr-x. 2 root root  4096  4. Dez 17:41  io
lrwxrwxrwx. 1 root root    18 20. Dez 17:44  model -> vosk-model-de-0.21
-rwxr-xr-x. 1 root root  2246 26. Jan 11:18  pva
-rw-r--r--. 1 root root  1977 26. Jan 10:50 'PVA$AnalyseMP3.class'
-rw-rw-r--. 1 root root 55337 26. Jan 10:50  PVA.class
-rw-r--r--. 1 root root 14214 20. Dez 17:27  pva.conf.default
-rw-r--r--. 1 root root 99253 26. Jan 10:50  PVA.java
-rwxrwxr-x. 1 root root  2512 20. Dez 21:55  pva.py
-rwxr-xr-x. 1 root root   970 20. Jan 18:42  pvatrayicon.py
-rw-rw-r--. 1 root root   421 26. Jan 10:50  Reaction.class
-rw-rw-r--. 1 root root  4963 23. Jul 2021   README.txt
-rwxr-xr-x. 1 root root   461 10. Jan 11:43  shutdown.sh
drwxr-xr-x. 2 root root  4096 26. Jan 11:17  systemd
-rwxr-xr-x. 1 root root    41 25. Jan 20:03  timer.sh
drwxr-xr-x. 8 root root  4096 15. Sep 00:21  vosk-model-de-0.21
drwxr-xr-x. 6 root root  4096  8. Dez 2020   vosk-model-small-en-us-0.15
```

it does detect small and large model files, but don't use both per language.


<== Carola, a PVA ==>


Carola is the keyword I use to address the pva. You can change this in an overwrite file in `~/.config/pva/conf.d/00-keyword-overwrite.conf` you need to create.

HINT:

In case your desired app does not have the same functionality as the default apps, which are the default apps for a reason, create bash wrapper scripts and use those.

SETUP:

You noticed the `pva.conf.default` file? It's now completely useless, but may give some impression on how to configure it.

To start, visit `/etc/pva/conf.d/`  and check the german numbers file. It's the equivilant to a `locale` file. It container numbers, monthnames and weekdaynames etc. Clone it as `10-<yourlanguage>-numbers.conf` and change the content accordingly. Most languages of the western world will match the format used. Numbers from 0-19 differently but from 20+ calculateable, 7 days a week etc. If you need to use a none romanic based language, PVA may needs to be rewritten and for this we surely need to your help with examples, rulesets etc. 

There are two ways to handle changes: 

You can do them for YOUR useraccount only by placing the configfiles into `~/.config/pva/conf.d/`  OR
You can write new configfiles with higher startnumbers in `/etc/pva/conf.d/` like `10-defaults-english.conf` and just rewrite and slightly adapt the texts used in the german default file. 

The higher number ( 10 ) means, this configfile is read later, which means, it's content is overwriting stuff the is unique. 

```
text:"de","KEYWORD","meine text version in deutsch"
command:"erzeuge|metadata","MAKEMETACACHE",""
```

Of course you do not overwrite the german "text" base, instead you change it like this:

```
conf:"lang_short","en"
conf:"lang","en_EN"

text:"en","KEYWORD","your version in english"
command:"create|metadata","MAKEMETACACHE",""
```

The "command" line does not overwrite the german command, it adds it to the command hashtable. You ask why: because an english model won't recognize german words anyway and vice versa, and so we don't need to overwrite those texts. You can just add as may lines in as many languages as you like and give them the ACTIONKEYWORD it needs.

For the `text` section, you can't do this simple way, we need those texts distinguished per as tripple-pairs of Language+Keyword+Text. 

The `conf` entry overwrites the german default setting, as you can only have one language per user. 

If you don't have the apps  in there, just install them. 

You can find qmmp & celluloid in the fedora repos, but to fully make use of qmmp, you may wanne install some plugins from rpmfusion.org. 
Jitsi ( Desktop version ) can be found on github and leave a +1 to Ingo Bauersachs, the main dev of Jitsi, you will LOVE calling people just with your voice ;)
Jitsi will be the bridge to your SIP provider, which can be your dsl or cable modem or something like SIPgate, Asterix etc.

A online CardDAV addressbook (with content) is required for some functions like sending email or calling someone. If you find a way to get those infos from the desktop contact app, please send a pull request. CardDAV has the advantage of also working in Thunderbird, which gives you a nice UI to enter your data. Also Thunderbird is the main MailAPP.

The `pva.conf` file is self explaning.

## Since commit #2c74e1d (16.12.2021) you can have multiply config files. 

There are two places configs are found:

```
/etc/pva/conf.d/00-test.conf
~/.config/pva/conf.d/00-test.conf
```

The actual config `./pva.conf` is always loaded as the last config file, to overwrite existing, alternative settings . I.E. if you changed the searchengine via "use duckduckgo as searchengine" and you had google before, you need to save this up. The file is found in $HOME/.config/pva/ .

COMMANDS:

You will find here the commands, PVA can undertand atm. You will see, they are in german, but explained in english.
Thats because, it has been developed in german. Now any language can be supported, but someone still needs to write those commands and responses in theire native language and post them. If you wannte help out, get the /etc/pva/conf.d/ files and move them from 0x-.. to 1x-... then translate them. It may be easier for you to do that at $HOME/.config/pva/conf.d/ , because you don't need to be root to do it.

Any time you see a "|" it seperates AND-connected keywords:

"you|succeed|will" can be any combination of those words, but they ALL need to be in the spoken sentence to have a positive match.
In case the cmd has an optional part like this:

command:"i|want|to|listen","PLAYAUDIO","add|it"

it matches: "I want to listen to queen "

it also matches: "I want to listen to queen add it" AND it removes "add" and "it" from the sentence, so those words do not end up in the searchterm.

it also matches: "I want to listen to queen it" and removes the "it", because it tries to still remove "add" and "it". 

You need this to remove words, that shall not part of the search term.

## More advanced commands:

There are two different ways to start playing music:  `PLAYAUDIO` and `PLAYMUSIC`

They do the same thing, but sligtly different:

`PLAYAUDIO` is a cmd to search for a song or artistname and adds all matches to a new playlist, afterwards the playback starts.
If you use the `PLAYAUDIO` keywords and ADD the `PLAYMUSICRADD` keywords, the playlist is not cleared and all matches will be added to the list.

`PLAYMUSIC` means, "start playback" whatever is in the list, from the actual position in the list. You can use this to restart a stopped playback. 
But PLAYMUSIC can do more, it can randomly search the database and play a song. If you use PLAYMUSIC + PLAYMUSICRANDOM + PLAYMUSICRADD together, 
it adds those randomly loaded songs to the end of the current playlist. 

You find those keywords defined in the textsection:

`text:"de","PLAYMUSICRANDOM",".*(zufällig).*"`
`text:"de","PLAYMUSICRADD",".*(noch|dazu).*"`

in english:

`text:"en","PLAYMUSICRANDOM",".*(random).*"`
`text:"en","PLAYMUSICRADD",".*(add).*"`

As you see, it's a Regular Expression, which is a mighty tool to match patterns in texts. If you know how to use them, you can have a lot of fun here,
BUT REMEMBER, all additional keywords need to be removed in the cmd section to they end up as parts of the searchterm!

There are some commands that have optional parameters. The easiest way to spot them is to check the source OR to search for the keyword in conjunction with the "text:" definition ( i.e. `PLAYMUSIC` -> `PLAYMUSICRANDOM` )

A third cmd, `ADDTITLES` adds a number of randomly selected songs to the playlist.


{TERM} mandatory infos needed to perform task
[TYPE] i.e. optional info i.e. number type like cellphone, work or home

Keyword-Sentences in Human Response Simulation:

"ha ha ha"
"danke"
"sehr witzig"
"das funtioniert"
"das funtioniert nicht"
"wie ist dein name"


Keywords in Bot Reaction: " Keyword " + content

```
"autorisierung code {TERM}"		-> give secret code to ack previous command
"autorisierung neuer code {TERM} "	-> set new secret code 

"nochmal"                               -> repeat last cmdline

"wie war der letzte fehler" 		-> readout last error message of selfcompile try
"wie war die fehlermeldung" 		-> readout last error message of selfcompile try

"schalte dich ab" 			-> request Alpha code and exit
"compiliere dich neu" 			-> compile the PVA.java file new  ( This obviously can only work, if it's installed for your userid )

"liste telefonbuch auf" 		-> print out entries in carddav database (it's for debug) 

"rufe {TERM} [TYPE]  an"		-> search for TERM in carddav database and execute jitsi/calls to make the call 
"{TERM} [TYPE] anrufen" 		-> 
"ich möchte {TERM} [TYPE] sprechen"	-> 
"ich möchte mit {TERM} [TYPE] reden"	-> 

"mache einen screenshot"		-> make screenshot
"mache screenshot"

"wie spät ist es"			-> tells the actual time
"sag mir die Uhrzeit"

"wie hoch ist die last"			-> tells the actual /proc/loadavg
"wie hoch ist die load"

"wie ist das wetter"			-> gives out actual weather report for {LOCATION}
"wie wird das wetter"			-> gives out actual weather report for the next hours
"wie wird das wetter morgen"		-> gives out actual weather report for tomorrow

"beende {TERM}"				-> stops apps  {TERM} =>  firefox, wetter, karte, karten, kartenapp, film, video, musik, audio
"stoppe {TERM}"				   "film" "video" "musik" "audio" refer to teh configured players

"starte {TERM}				-> starts apps  {TERM} => "blender" "wetter" "karte" "karten" "kartenapp" "firefox" "netflix" "windy" "openoffice" "emailprogramm"
"öffne  {TERM}				# ALIAS of starte 
"öffne {TYPE} mit {APP}"		-> takes the last search result for {TYPE} and opens it with {TERM} . This implies, that you did a search before.
	
"ich möchte {TERM} hören"		-> search for musik files matching TERM, where TERM can have more than one word. on Success the audioplayer gets informed

"was ist da zu hören"			-> ask audioplay about trackinfos
"was ist gerazu zu hören"
"was spielst Du gerade"
"was höre ich"
      
"ich möchte {TERM} sehen" 		-> search for video files matching TERM, where TERM kann have more than one word. on Success the videoplayer is started

"schreibe email an {TERM} [betreff [subject text]] [inhalt [body content]]" -> opens mailapp (thunderbird) IF a emailaddress for TERM has been found. Optional: use keyword "inhalt" or "betreff" to add infos at the right positions.

"suche email {TERM}" 			-> searches for Emailaddress of TERM in carddav database and reads it.
"suche (telefon|telefonnummer) {TERM}" 	-> searches for phonenumber of TERM in carddav database and reads it, according to it's type.

"suche dokument {TERM}"			-> search on ~/Dokuments aka. ~/Documents  for TERM as PDF or TXT or ODT/S/P
"suche dokument {TERM} pdf"		-> search on ~/Dokuments aka. ~/Documents  for TERM as PDF

"suche bild {TERM}"			-> searches in configures pics paths for: JPEG,JPG,GIF,PNG,SVG

"lies pdf {TERM}" 			-> **EXPERIMENTAL** .. find and read out pdf file
"lies text {TERM}" 			-> find and read out txt files

"spiele musik" 				-> starts audioplayer in playback mode ( whatever it had in its queue )
"spiele musik zufällig" 		-> search the music database and randomly chooses one file
					   ** the entire audio database is saved to a cache file for fasten up requests ** 
"stoppe musik"				-> stops audioplayer playback
"halte musik an"
"musik anhalten"
"musik stop"
"stop musik"

"mach leiser"				-> decrease volume on audioplayer
"mache leiser"	

"mach lauter"				-> increase volume on audioplayer
"mache llauter"	

"musik leiser"				-> decreases musik volume by 1 steps
"musik viel leiser"			-> decreases musik volume by 5 steps

"musik lauter"				-> increases musik volume by 1 steps
"musik viel lauter"			-> increases musik volume by 5 steps

"nächster titel"			-> skip audio track by one
"nächstes stück"			-> skip audio track by one
"nächstes lied"				-> skip audio track by one

"übernächster titel"			-> skip audio track by two
"übernächstes stück"			-> skip audio track by two
"übernächstes lied"			-> skip audio track by two

"{NUMBER} lied weiter"			-> skip audio track by NUMBER : can be anything from 1 to 99
"{NUMBER} lieder weiter"		-> skip audio track by NUMBER : can be anything from 1 to 99

"{NUMBER} lied zurück"			-> skip audio track backwards by NUMBER : can be anything from 1 to 99
"{NUMBER} lieder zurück"		-> skip audio track backwards by NUMBER : can be anything from 1 to 99    

"springe {NUMBER} vorwärts"		-> advance by NUMBER of seconds : can be anything from 1 to 99
"springe {NUMBER} vor"			-> advance by NUMBER of seconds : can be anything from 1 to 99

"springe {NUMBER} zurück"		-> go back in track by NUMBER of seconds : can be anything from 1 to 99
"{NUMBER} lieder zurück"		-> go back in track by NUMBER of seconds : can be anything from 1 to 99    

"ton an"				-> unmute audioplayer
"ton aus" 				->   mute audioplayer

"benutze {TERM}"                        -> switches to named alternative for a command
"websuche {TERM}"                       -> starts a websearch based on configured searchengine and prefered browser.
"bildschirm sperren"                    -> locks the desktop
"bildschirm entsperren"                 -> locks the desktop

"video fullscreen"			-> tell media/videoplayer to go to fullscreen 
"video vollbild"			

"video fullscreen aus"			-> tell media/videoplayer to go back to windowed mode
"video vollbild aus"

"{NUMBER} videos weiter"		-> skip N videos in the playback queue forward
"{NUMBER} videos zurück"		-> skip N videos in the playback queue backwards
"ein video weiter"			-> skip 1 video forward
"nächstes video"

"ein video zurück"			-> skip 1 video backwards
"letztes|video"				

"video weiter"				-> UNPAUSE or start videoplayback, which in most mpris implementations does the same.
"video start"
"video|fortsetzen
"wiedergabe|starten
"wiedergabe|fortsetzen

"video stop"				-> stop videoplayback FULLY <- this is an important difference regarding PAUSE!
"wiedergabe|stop

"pausiere video"			-> pause videoplayback
"video pause"        
"wiedergabe|pausiere

"erzeuge metadata"
"erzeuge mp3 metadata"

"erinnere mich {DAY} um {TIME} an {TEXT}" 
					-> add a timed reminder to the database at {TIME} on {DAY} with the reminder {TEXT}
					DAY parameter is optional and defaults to "today"
					Example: "erinnere mich um neun uhr zwölf an Frühstück einnehmen mit Carola"
						 which result in an entry for TODAY 09:12 (AM) with the text "Frühstück einnehmen mit Carola"
					Example: "erinnere mich morgen um neun uhr zwölf an Frühstück einnehmen mit Carola"
						 which result in an entry for the next morning 09:12 (AM) with the text "Frühstück einnehmen mit Carola"
					Example: "erinnere mich freitag um neunzehn uhr fünfundfünfzig an romatisches Abendessen im Donbass mit Carola"
						 which result in an entry for next Friday 19:55 (PM) with the text "romatisches Abendessen im Donbass mit Carola"
						 
"ließ termine vor"			-> lists verbally all reminders
"meine termine"

"benenne dich in {NAME} um"		-> rename yourself to {NAME}
"dein neuer name ist {NAME}"
"dein neuer name lautet {NAME}"

! This part works only, if you add the twinkle scripts & configs to your main or user config directory. 
! The config and scripts need manual adjustment to your system to function proper! 
! But, you can archive cool stuff with the underlying command mechanics!

"schalte X auf Y um" 	-> switches pulseaudio output device for process X to sink Y
```
