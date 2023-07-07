# SWAT 4 Dedicated Server in Docker
A Docker container running SWAT 4 under Wine, built on Alpine. This container supports mods, see the docker compose or docker cli commands below for information on how to enable them. Your game directory (mounted in the container as `/swat4`) should be structured as follows:

    .
    ├── ...
    ├── Content
    ├── ContentExpansion
    ├── YourMod1
    └── YourMod2

The container is built with the Master Server Patches necessary for listing your game on the community server list, it will check your `Content/` and `ContentExpansion/` folder for their existence at each startup and automatically replace them if are they are not the correct files. You can find the project page for the patches [here](https://github.com/sergeii/swat-patches/tree/master/swat4stats-masterserver/).

### docker compose

```
version: "3.8"
services:
  swat4-server-docker:
    image: ghcr.io/mistercalvin/swat4-server-docker:latest
    container_name: swat4-server-docker
    environment:
      - "CONTENT_VERSION=SWAT4" # Required: Choose SWAT4, TSS, or enter the name of your mod folder (case-sensitive); Default: SWAT4
      - "SERVER_NAME=A SWAT4 Docker Server" # Required: Name of the Server; Default: A SWAT4 Docker Server
      - "SERVER_PASSWORD=" # Optional: Password for the Server (alphanumeric characters only); Default: unset
      - "GAME_TYPE=CO-OP" # Required: Choose Barricaded Suspects, CO-OP, Rapid Deployment, Smash and Grab, or VIP Escort (all case-sensitive); Default: CO-OP
      - "LAN_ONLY=False" # Optional: If True, the server is hosted only over the LAN (not internet); Default: False
      - "SERVER_MAP=SP-FairfaxResidence" # Required: Map name, without extension; Default: SP-FairfaxResidence
      - "MAX_PLAYERS=10" # Optional:  Maximum amount of players for the server; Default: 10
      - "ADMIN_PASSWORD=" # Optional: Admin password for in-game administration (alphanumeric characters only); Default: unset
      - "NUM_ROUNDS=15" # Optional: Number of rounds for each match; Default: 15
      - "ROUND_TIME_LIMIT=900" # Optional: Time limit (in seconds) for each round (0 = No Time Limit); Default: 900
      - "DEATH_LIMIT=50" # Optional: How many deaths are required to lose a round (0 = No Death Limit); Default: 50
      - "MP_MISSION_READY_TIME=90" # Optional: Time (in seconds) for players to ready themselves in between rounds in a MP game; Default: 90
      - "POST_GAME_TIME_LIMIT=15" # Optional: Time (in seconds) between the end of the round and server loading the next level; Default: 15
    volumes:
      - /path/to/your/gamefiles/:/swat4
    ports:
      - 10480-10483:10480-10483/udp
    restart: unless-stopped
```

### docker cli

```
docker run -d \
  --name=swat4-server-docker \
  -e CONTENT_VERSION="SWAT4" \
  -e SERVER_NAME="A SWAT 4 Docker Server" \
  -e SERVER_PASSWORD="" \
  -e GAME_TYPE="CO-OP" \
  -e SERVER_MAP="SP-FairfaxResidence" \
  -e LAN_ONLY="False" \
  -e MAX_PLAYERS="10" \
  -e ADMIN_PASSWORD="" \
  -e NUM_ROUNDS="15" \
  -e ROUND_TIME_LIMIT="900" \
  -e DEATH_LIMIT="50" \
  -e MP_MISSION_READY_TIME="90" \
  -e POST_GAME_TIME_LIMIT="15" \
  -p 10480-10483:10480-10483/udp \
  -v /path/to/gamefiles/:/swat4 \
  --restart unless-stopped \
  ghcr.io/mistercalvin/swat4-server-docker:latest
```
  
## Server Ports
SWAT 4 requires Base Port + 3 (Default port is 10480, so 10480-10483/udp)

| Port      | Default  |
|-----------|----------|
| Join 		| 10480/udp|
| Query     | 10481/udp|
|        	| 10482/udp|
|       	| 10483/udp|

## Notes / Bugs
- I tested this to the best of my abilities, however SWAT 4 is a buggy game and I am sure there are some I missed. If you have any problems feel free to [open a new issue](https://github.com/MisterCalvin/swat4-server-docker/issues).

- Unfortunately, the container is running as the root user. Not only is this poor practice for Docker in general, but it is also highly advised against while using Wine. I spent a couple days trying to figure out why I cannot launch the server as a less restricted user (it is currently due to an issue with the X server), but was not successful. While the probability of something malicious escaping the container and infecting the host is substantially small please be aware of the [risks](https://wiki.winehq.org/FAQ#:~:text=NEVER%20run%20Wine%20as%20root,wine%20folder%20in%20the%20process.).

- Mods are supported, however they will not respect the `ADMIN_PASSWORD` env variable. For setting up in-game administration on a server running a mod you will need to consult with the developers documentation.

- Mods may also have additional features or options you can configure, you will need to manually edit the SwatGUIState.ini file found within your mods `System/` folder if you wish to modify anything. The startup script is designed to only overwrite the env variables listed in the Dockerfile, everything else will be untouched.

- I have tested the container with the following mods:

| Mod       			| Version  																							|
|-----------------------|---------------------------------------------------------------------------------------------------|
| SWAT: Elite Force     | [7](https://www.moddb.com/mods/swat-elite-force/downloads/swat-elite-force-v7)					|
| SEF First Responders 		| [0.67 Stable](https://www.moddb.com/mods/sef-first-responders/downloads/sef-first-responders-v067-stable)	|
| SWAT: Back to LA 		| [1.6](https://www.moddb.com/mods/swat-back-to-los-angeles/downloads/sef-back-to-los-angeles-v16)	|
| Canadian Forces: Direct Action | [4.1](https://www.moddb.com/downloads/canadian-forces-direct-action-41)					|
| 11-99 Enhancement 	| [1.3](https://www.moddb.com/mods/11-99-enhancement-mod/downloads/11-99-enhancement-mod-v13)		|

## Building
If you intend to build the Dockerfile yourself, I have not pinned the packages as Alpine does not keep old packages. At the time of writing (2023/07/05) I have built and tested the container with the following package versions:

| Package   			  | Version  	   |
|-------------------|--------------|
| i386/alpine			  | 3.18.2     	   |
| wine     				  | 8.11-r0	     |
| xvfb-run      		| 1.20.10.3-r1 |
| findutils      		| 4.9.0-r5	   |
| wget					    | 1.21.4-r0	   |
| tini              | 0.19.0-r2    |
