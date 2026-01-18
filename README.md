# xim-docker
A docker container for hosting the game Xim by Aamace.

this is set up to run on ubuntu currently, though if you know how to user docker it should be pretty straight forward to modify it for use on your system.

on ubuntu it's striaght forward, you just run run.sh /path/to/ffxi/files. That's it as long as  you've also run the ffxiSoundConvert to make sure that you have a .ogg copy of your audio files in your sound folder.

ffxiSoundConvert is located in my profile.

After run.sh finishes simply navigate to your directory and run "docker compose up -d" and the server will be running at localhost:8082. You can change this in your docker_compose.yml file. 


This version does not support the ability to act as a server, so it's only able to run on your local system. There is another version i'll publish that will allow you to set it up as a server for your friends on a local network via a self signed certificate. If that's what you're interested in, please wait for me to upload it.
