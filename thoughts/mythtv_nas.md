### Setting up a MythTV backend on a QNAP NAS

I realize that most people have moved away from watching TV over cable or broadcast these days.  We still use it
at my house, and have become quite used to running it with MythTV as our DVR (skipping commercials, etc). Recently I
decided that it was time to shift the MythTV setup off the dedicated system it had been running on - that system is 
several years old and has crashed a few times recently with hardware-like errors, so it was time to do something else 
before the machine died and I had to do it unplanned. Rather than buy or build a new computer to run it, I decided 
to continue drinking the container/VM coolaid (so tasty), and run the backend as a VM on the QNAP NAS we have to keep 
pictures and other shared media.  

In particular, we're running MythTV with a SiliconDust HDHomeRun Prime network tuner. I like it, but it turns out 
that this makes the setup of MythTV a bit more complicated. In the interest of saving some other poor slob who's trying
to do this also, here's the steps I took to get all this wired together & working. Note: this is just building the
backend. The frontend can be a simple ubuntu desktop with the mythtvfrontend application installed.

## Storage:

To manage pack-rat/digital hoarding tendencies, I decided to make the actual disk storage for mythtv a dedicated volume
on the NAS. This is fairly easily done:
 1.If you already assigned all the space on the drives in the NAS, and need to make space:
    1. Open "Storage and Snapshots", and go to Storage->Storage/Snapshots.
    2. Highlight on the Volume you want to shrink to make space, right-click, and select "Manage"
    3. Click "Resize Volume" and set a new (smaller, assumedly) value for the volume you're shrinking to make space for
     the MythTv volume
 2. In "Storage and Snapshots" create a new volume, and name it whatever you like
 3. in the File Station, make a folder in the new volume to store MythTV's files

If you already have a folder for this purpose and just want to move that folder to a dedicated Volume, you follow
the same procedure to make space as above (step 1), and then right-click on the folder and select "Transfer to Dedicated
Volume"

## Network setup

The communication between the HDHomeRun and the MythTV backend isn't something that I've looked at carefully, but a
number of sites online mention that it tries to call back to the MythTV backend directly, on random ports...which means 
that you can't have a many-to-one NAT between the HDHomeRun and the backend. The way I got around this was to use one 
of the other interfaces on the NAS as a dedicated "containers" interface. I configured the second interface of the NAS 
as  a bridge, and  connected all the containers running on the NAS to that bridge so that they appeared directly on 
my home network, rather than trying to map host ports to VMs.

Here's what I did for networking:
 1. Connect a second interface on qnap to my home network
 2. In the QNAP GUI, go to the Network & Virtual Switch control
 3. in virtual switch, select "add", and select "advanced mode"
 4. select the second adapter you conneted to the network (not the one you talk to the NAS over)
 5. on the next panel select "do not assign an IP address"
 6. on the next panel, leave DHCP and NAT off (you want DHCP from your network, not the NAS)
 7. accept the rest of the defaults

## VM setup
Next we need to make the actual VM, and attach it to the network. In this case, I wanted to use Docker, but it
became very complicated. MythTV assumes a lot about its install environment, especially that it has access to a 
desktop GUI. (we'll get to that a bit below) Docker containers almost never have that, and the pre-built QNAP ones
also minimized Ubuntu itself. I kept running into things it was missing, so I eventually bailed on the Docker images
and used the LXC one. The steps I followed:
 1. open Container Station
 2. Select "Create" on the left, and scroll down to Ubuntu 18.04 LTS. NOTE: there will be several Ubuntu LTS images 
listed here. Select the 18.04 one that is tagged *LXC*, NOT *Docker*
 3. On that image tile, click "Create" or "Install" (depending if it already has the image downloaded).
 4. Give the image a name (like "mythtvbackend"), and set some CPU and memory limits. I used 20% CPU and 8GB RAM.
 5. Click on "Advanced Settings"
     1. In "Network" -> "Network Mode" Select "bridge", and select the bridge you created in the network setup above.
     2. In "Shared Folders" click "add" and select the volume you made for the MythTV storage above. In the mount
       point, I entered "/var/lib/mythtv/" since that's the default Ubuntu place to put mythtv files
 6. Select "Create" and wait for it to build the system.
 7. Go back to the "overview" page, and select the container you just made. you will be able to access the VM's console
    from here.
 8. The default install of Ubuntu like this makes a user "ubuntu" with the password "ubuntu". Change that. Either 
change the password for that user, or make a new user with a better password and make the "ubuntu" user not able to
    log in (I did this).
 9. install openssh-server, so you can connect to it remotely.

Now you're ready to actually install the MythTV backend.


## MythTV setup

To actually configure MythTV, you've going to need to be able to pull the display back to a system somewhere. Basically,
there is no headless way to configure the MythTV backend. That's a shame, but that's the state of things. So, you're
going to need to be able to connect X-Windows somehow back to your system. Personally, I used a Mac to do this, and
ssh'd from the command line to the MythTV VM. 

One note to make before you dive into this: my Mac had dual displays. OSX handles tunneling X-windows, but does not
handle drawing X-Windows applications across both displays. This means when you launch the MythTV configuration 
application, if you have dual displays it will only draw half of the screen. This is not helpful. I ended up having
to unplug one monitor for the time I was configuring MythTV.

My steps:
 1. `ssh -X <user>@<IP of VM>` the -X option tells SSH to tunnel X-Windows also
 2. `sudo apt-get install software-properties-common` you need this to get the `add-apt-repository` command that you're
    going to use in a moment
 3. `sudo add-apt-repository ppa:mythbuntu:31`. (version 31 is the most recent as of the writing of this, you may want
    to check the MythTV installation instructions to see if there's a more recent one when you read this.)
 4. `mythtv-setup`

Some notes about mythtv-setup:
- The HDHomerun Prime has 3 tuners. In previous versions of MythTV these would show up as separate tuners with unique
   identifiers that you would add to the MythTV configuration. MythTV 31 does not do this. If you want to use all 3 
   tuners, you need to add the HDHomeRun Prime unit as a tuner 3 times.
- In the save paths, you will want to double-check that MythTV is really writing files to the default location that
you mounted the NAS share on (/var/lib/mythtv). It probably is, but it's worth making sure.
-  Getting schedulesdirect working is incredibly fiddly. I don't have it quite working yet myself, so I'll leave that 
out here, but if you're stuck, know that you're not alone.
   