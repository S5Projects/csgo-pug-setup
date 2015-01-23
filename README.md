csgo-pug-setup
===========================

[![Build Status](https://travis-ci.org/splewis/csgo-pug-setup.svg)](https://travis-ci.org/splewis/csgo-pug-setup)

This is a useful plugin for managing pug games, especially **10 mans**/gathers. It allows a player to type .setup into chat and select (from a menu):
- how to choose the teams (players do it manually, random teams, captains select teams)
- how to choose the map (use the current map, do a map vote, veto maps)

The goal is to allow a **lightweight, easy-to-use setup system** that automates as much as possible with as few dependencies as possible. However,
the goal isn't fully automated - it assumes the players know each other or there is an admin. There is no mechanism for kicking players or anything similar.

Part of being lightweight is doing nothing that can interfere with the server's performance. **When the game is actually live, the plugin is doing extremely little work** - the only thing it does is read chat commands (e.g. pausing when a captain types .pause). Otherwise, there is 0 effect on gameplay and the rounds progress the same as without sourcemod on the server. This is in contrast to the WarMod plugin or what ESEA servers use, where they are tracking the overall score, overriding the in-game warmup period, etc.

The goal is to make setup easier for people, not provide a comprehensive pug platform. Please keep this and the principle of **keep it simple, stupid** in mind when making any suggestions.

Also see the [AlliedModders thread](https://forums.alliedmods.net/showthread.php?t=244114).

#### Some alternatives
- [Goonpug](https://github.com/goonpug/goonpug) (very similar to this plugin, but with more features - and more dependencies)
- [PUG Mod](https://forums.alliedmods.net/showthread.php?p=1742753) (relatively plain - but no captain selection)
- [WarMod](https://forums.alliedmods.net/showthread.php?t=225474) (does pretty much everything - can be difficult to configure, heavyweight)


## Download
You should be able to get the most recent download from https://github.com/splewis/csgo-pug-setup/releases.

This plugin optionally supports the [Updater](https://forums.alliedmods.net/showthread.php?t=169095) plugin, which will automatically update to backwards compatible releases.

You may also download the [latest development build](http://ci.splewis.net/job/csgo-pug-setup/lastSuccessfulBuild/) if you wish. If you report any bugs from these, make sure to include the build number (when typing ``sm plugins list`` into the server console, the build number will be displayed with the plugin version).


#### Usage
There is a notion of the the pug/game "leader". This is the player that writes .setup first and goes through the setup menu. The leader has elevated permissions and can use some extra commands (e.g. pause). To prevent some abuse there is also an admin command sm_leader to manually change the leader.

Generally, here is what happens:
- A player joins and types ``.setup`` and goes through the menu to select how the teams and map will be chosen
- Other players join and all type ``.ready``
- If the leader setup for a map vote, the map vote will occur and the map will change, then all players will type ``.ready`` on the new map
- If the leader setup for a captain-style team selection, the game will wait for when 2 captains are selected, then the captains will be given menus to chose players
- Then the game will initiate a live-on-3 restart and go


## Installation
Since this is a sourcemod plugin, you must have sourcemod installed. You can download it at http://www.sourcemod.net/downloads.php.

Note that sourcemod also requires MetaMod:Source to be on the server. You can download it at http://www.sourcemm.net/downloads.

Installing these simply means placing their files on the game server. Uploading them over FTP (for example, by FileZilla) is all you need to do.

**As of 1.3.0, sourcemod 1.7 is required.**

Download pugsetup.zip and extract the files to the game server. You only need to merge the ``cfg`` and ``addons`` folders from the downloaded zip archive.

 From the download, you should have installed the following (to the ``csgo`` directory):
- ``addons/sourcemod/plugins/pugsetup.smx``
- ``addons/sourcemod/translations/`` (the entire directory)
- ``addons/sourcemod/data/pugsetup/`` (the entire directory)
- ``addons/sourcemod/configs/pugsetup/`` (the entire directory)
- ``cfg/sourcemod/pugsetup`` (the entire directory)

**Once all of these are on the server, it should just work.** If you want to tweak the configs, maplists, or use the addon-plugins, read on.

Sometimes it's easier to add features in a separate plugin than the core plugin. So, there are a few addon (**optional**) plugins included in the download:
- ``pugsetup_autokicker``: kicks players that join when the game is already live, and players not selected by captains when using captain-player selection
- ``pugsetup_teamnames``: sets team names/flag according to the players on the team, see more detail at the end of the readme
- ``pugsetup_teamlocker``: blocks players from joining full teams when a game is live
- ``pugsetup_hostname``: adds some tags to the server hostname depending on the pug status, examples: "[LIVE]" and "[NEED 3]"
- ``pugsetup_rwsbalancer``: implements a simple rws calculation (stored via clientprefs) and balances team accordingly when using manual/random teams
- ``pugsetup_chatmoney``: prints out the team members' money to chat on round starts
- ``pugsetup_practicemode``: adds an option to the .setup menu to launch a practice mode with cheats/infinite ammo/respawning/etc.

All of these are in the ``plugins/disabled`` directory and they are all independent of each other. To enable one, move it from the `plugins/disabled` directory to the `plugins` directory.


## Configuration

After installing the plugin, start the server and check ``cfg/sourcemod/pugsetup``. There will be a file called ``pugsetup.cfg`` that you can edit to change the cvars the plugin uses.

You can also add more chat alias commands in [addons/sourcemod/configs/pugsetup/chataliases.cfg](configs/pugsetup/chataliases.cfg) if you wish. If players are not comfortable with english, I'd
strongly recommend adding chat aliases, since those will be read by the plugin and used in chat messages when referencing commands.

Below is a list of commands you may want to alias:
```
sm_ready
sm_unready
sm_setup
sm_rand
sm_pause
sm_unpause
sm_endgame
sm_forceend
sm_leader
sm_captain
sm_stay
sm_swap
```


## For developers
There is some extension support in the form of some natives and forwards. See [pugsetup.inc](scripting/include/pugsetup.inc).

The optional addon plugins generally make good use of these. Check the [scripting](scripting) directory and look at some for examples.


## Enabling GOTV
You need to enable gotv to use the demo-recording feature. Adding the following to your ``server.cfg`` will work:

    tv_enable 1
    tv_delaymapchange 1
    tv_delay 45
    tv_deltacache 2
    tv_dispatchmode 1
    tv_maxclients 10
    tv_maxrate 0
    tv_overridemaster 0
    tv_relayvoice 1
    tv_snapshotrate 20
    tv_timeout 60
    tv_transmitall 1

Of course, you can tweak the values.


## Commands

Some commands that are important are:
- **!setup**, begins the setup phase and sets the pug leader
- **!ready**
- **!unready**
- **!pause**
- **!unpause**
- **!capt** gives the pug leader a menu to select captains
- **!rand** selects random captains
- **!leader** gives a menu to change the game leader
- **!endgame** force ends the game safely (only the leader can do this, note that this **resets the leader** to nobody)
- **!forceend** force ends the game without a confirmation menu

You can also type .ready instead of !ready, or .capt instead of !capt, etc.

These are some helper commands for automation purposes the bypass requiring a player to press any menus:
- sm_10man (sets up a 5v5 game using captains and a map vote)
- sm_forceend (force ends the game with no confirmation menu)


## ConVars
These are put in an autogenerated file at ``cfg/sourcemod/pugsetup.cfg``, once you start the plugin go edit that file if you wish. These are just some important ones, check the file for more.
- **sm_pugsetup_warmup_cfg** should store where the warmup config goes, defaults to the included file ``sourcemod/pugsetup/warmup.cfg``)
- **sm_pugsetup_autorecord** controls if the plugin should autorecord a gotv demo (you may need to add some extra cvars to your cfgs, such as tv_enable 1)
- **sm_pugsetup_requireadmin** controls if an admin flag is needed to use the setup command

You may also want to change ``sm_vote_progress_chat`` in ``cfg/sourcemod/sourcemod.cfg`` to print players' votes when map voting is being used.

## Fun with team names/flags
(Using the included ``pugsetup_teamnames.smx`` plugin)

Just for fun, I added support to automatically set mp_teamname_1 and mp_teamflag_1 (and 2). Here's how it works
- you run the **sm_name** command in console to associate a player with a team name and flag (example: ``sm_name splewis "splewises terrible team" "US"``)
- when the game goes live, the plugin picks a name/flag randomly from the players on each team
- when running the sm_name command, the syntax is: ``sm_name <player> <teamname> <teamflag>``
- note that the team flags are the [2-letter country codes](http://countrycode.org/)
- the team names/flags are stored using the clientprefs API, so a database for clientprefs must be set (it is by default)
