## spigotmc-init-script

a fork of [superjamie](https://github.com/superjamie)'s [minecraft-init-script](https://github.com/superjamie/minecraft-init-script)

originally by Jamie Bainbridge <<jamie.bainbridge@gmail.com>>

by Edison Chen <<brotheredison.github@yahoo.com>>

This is an initscript to run a SpigotMC server on CentOS, Fedora, and Ubuntu. You are still able to run a vanilla Minecraft server with this script, however the more

### Features

*   Start, stop, restart CraftBukkit as a system service
*   Automatic (via cron) and manual logfile rotation
*   Automatic (via cron) and manual backups
*   Backup compression and rotation (keeps 7 days worth of backups)
*   Allows use of third-party backup solutions
*   Check latest Recommended Build and update to it if required
*   Information display including Java path, current memory usage, current TCP connections
*   Able to run multiple separate instances of the server at once

### Supported Distributions

*   CentOS 5-6, Fedora Core 1-6, Fedora 7-14
*   Ubuntu (Server), Debian 1-7, Linux Mint (untested)

Other distros which use SysV Init or Upstart will probably work.

Distros using systemd (Fedora 15+, Arch Linux, CentOS 7, Debian 8 (future release), etc) will probably not work.

### Dependencies

*   `screen`, `rsync`

    (you may need to install these)

*   `bash`, `chkconfig` or `sysv-rc`, `coreutils`, `cronie`, `curl`, `diffutils`, `grep`, `initscripts`, `net-tools`, `procps`, `tar`

  (these should all be installed by default)

*   Oracle Java 7

*   Enough disk space to save your map twice, plus another ~5 times for a week of compressed backup space.

  ie: If your map is 1GB then you probably need at least 7GB, plus any space your plugins require and any additional backups you'll be making.

### Installation

As the root user:

*   Install Sun Java (CentOS/Fedora)

    Download the RPM from <http://www.java.com/>

    `yum localinstall jre-<version>.rpm`

*   Install Sun Java (Ubuntu)

    ~~~
    sudo add-apt-repository ppa:webupd8team/java
    sudo apt-get update
    sudo apt-get install oracle-java7-installer
    ~~~

*   Confirm your JVM installation

    ~~~
    java -version
    ~~~

*   Create a new user with a home directory

    ~~~
    useradd -m <<username>>
    ~~~
    
*   Save the script as `/etc/init.d/spigotmc` and make it executable

    ~~~
    chmod +x /etc/init.d/spigotmc
    ~~~
    
*   Copy between the `<<COMMENT` and `COMMENT` lines and place the copy at `/etc/default/spigotmc`

    If you need to edit settings, edit the `/etc/default/spigotmc` file, not the initscript

*   Allow the user to run the init script without needing root access

    Type `visudo` and add this line to the bottom:

    ~~~
    <<username>> localhost=NOPASSWD:/etc/init.d/spigotmc*
    ~~~

*   Create an alias so you only have to type `spigotmc` to run the script

    Add the following line to both root and bukkit's `~/.bashrc` file:

    ~~~
    alias spigotmc='/etc/init.d/spigotmc'
    ~~~

*   Start the server on system boot if desired (CentOS/Fedora)
  
    ~~~
    chkconfig --add spigotmc
    chkconfig spigotmc on
    ~~~

*   Start the server on system boot if desired (Ubuntu)

    ~~~
    update-rc.d spigotmc defaults
    ~~~

As the regular user, bukkit:

*   Make the required paths

    ~~~
    mkdir -p ~/spigotmc
    (ignore if you are backing up to a remote host) mkdir -p ~/backups
    ~~~

*   Put your `spigot.jar`, world, plugins, `server.properties`, etc into `~/spigotmc`

### Backups

*   Create cron jobs to do regular backups and rotations around 4am

    Type `crontab -e` to open the cron interface and add the following

    ~~~
     0 4 * * * /etc/init.d/spigotmc backup              # backup world at 4:00am
     5 4 * * * /etc/init.d/spigotmc logrotate           # rotate logs at 4:05am
    15 4 * * * /etc/init.d/spigotmc removeoldbackups    # remove old backups at 4:30am
    ~~~

*   If you have multiple worlds, you can pass the worldname as a parameter to the regular backup

    ~~~
     0 4 * * * /etc/init.d/spigotmc backup world1       # backup world1 at 4:00am
     5 4 * * * /etc/init.d/spigotmc logrotate           # rotate logs at 4:05am
    15 4 * * * /etc/init.d/spigotmc backup world2       # backup world2 at 4:15am
    30 4 * * * /etc/init.d/spigotmc removeoldbackups    # remove old backups at 4:30am
    ~~~

*   If you wish to use a third-party backup solution, just disable world writes

    ~~~
    spigotmc save-off
    ~~~

    Then run your backup tool. Then re-enable world writes

    ~~~
    spigotmc save-on
    ~~~

### Multiple Instances

It is possible to run multiple instances, for example a Creative server and a Survival server, on the same system.

*   Copy the script to two new files

    ~~~
    cp /etc/init.d/spigotmc /etc/init.d/creative
    cp /etc/init.d/spigotmc /etc/init.d/survival
    ~~~

*   Edit the `Provides` section on Line 6 to the same as the new filename

    ~~~
    # Provides: creative
    # Provides: survival
    ~~~

*   Create a settings file for each instance in `/etc/default/` using the same name as the script

    ~~~
    /etc/default/creative
    /etc/default/survival
    ~~~

*   Set an alias for each server in `~/.bashrc`

    ~~~
    alias creative='/etc/init.d/creative'
    alias survival='/etc/init.d/survival'
    ~~~

*   Add the new scripts to `chkconfig` or `update-rc.d`

*   Set the paths of the separate maps in each script

    ~~~
    MCPATH="/home/<<username>>/creative"
    MCPATH="/home/<<username>>/survival"
    ~~~
 
*   Change your screen session names in each script

    ~~~
    SCRNAME="creative"
    SCRNAME="survival"
    ~~~

### Usage

*   Start the server

    ~~~
    spigotmc start
    ~~~

*   Stop the server

    ~~~
    spigotmc stop
    ~~~

*   Restart the server

    ~~~
    spigotmc restart
    ~~~

*   Back up the map and executable

    ~~~
    spigotmc backup
    ~~~

    The final compressed backup is done when the `.md5` file appears in the backup directory.

*   Back up multiple maps

    ~~~
    spigotmc backup world1
    spigotmc backup world2
    ~~~

*   Check the server is running

    ~~~
    spigotmc status
    ~~~

*   Get some more info

    ~~~
    spigotmc info

    (Sample)
    * CraftBukkit (pid 9037) is running...
    - Java Path : /usr/java/jre1.6.0_31/bin/java
    - Start Command : java -Xms512M -Xmx3584M -jar craftbukkit.jar nogui
    - Server Path : /home/bukkit/craftbukkit
    - World Name : world
    - Process ID : 9037
    - Memory Usage : 742796 kb
    - Active Connections :
    Proto Recv-Q Send-Q Local Address Foreign Address State
    tcp 0 0 0.0.0.0:25565 0.0.0.0:* LISTEN
    tcp 0 0 192.168.2.99:25565 192.168.2.69:55507 ESTABLISHED
    ~~~

*   Broadcast a message to the server

    ~~~
    spigotmc say
    ~~~

    Note that some punctuation like 'apostrophes' will not work.

### License

**GNU GPLv3**

This program is free software: you can redistribute it and/or modify
it under the terms of the GNU General Public License as published by
the Free Software Foundation, either version 3 of the License, or
(at your option) any later version.

This program is distributed in the hope that it will be useful,
but WITHOUT ANY WARRANTY; without even the implied warranty of
MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE.  See the
GNU General Public License for more details.

You should have received a copy of the GNU General Public License
along with this program.  If not, see <http://www.gnu.org/licenses/>.

### Contributors

*   Jamie Bainbridge <<jamie.bainbridge@gmail.com>>
*   Polhemic on GitHub
*   Mooash on GitHub
*   Jon Stephens
*   Edison Chen <<brotheredison.github@yahoo.com>>

Initial concept based off

*   http://forums.bukkit.org/threads/tutorial-centos-bukkit-installation.56371/
*   http://www.spigotmcwiki.net/wiki/M3tal_Warrior_Server_Startup_Script
*   http://forums.bukkit.org/threads/admin-linux-init-script-for-bukkit.53235/
