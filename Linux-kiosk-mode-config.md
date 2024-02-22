Linux kiosk mode (single-application-mode) configuration steps || Fedora 39 Workstation || Wayland Session

1. Install Fedora Workstation 39
    a) set 
        --> timezone Europe/Warsaw
        --> language Polski
    b) reboot to finish installation
        --> disable localization services and reporting problems
        --> enable third-party repositories
    c) create two accounts with passwords
        --> Admin as administrator
        --> kiosk as single user
                ---> open Settings
                ---> scroll down and choose Users
                ---> unclock adding users
                ---> Add Users
                ---> complete creating kiosk user

2. Setting up kiosk mode
    a) login to Admin account
    b) open terminal to set new root password 'sudo passwd root'
    c) re-login to kiosk account
    d) open terminal and switch to root
    e) install:
        yum install gnome-kiosk gnome-kiosk-script-session
            [check is there a package installed]
        rpm -qa | grep kiosk
            [update the system and restart desktop] 
        yum update -y
    f) vi /etc/gdm/custom.conf
        add [comment this for now]:
          #  AutomaticLoginEnable = True
          #  AutomaticLogin = kiosk
        --> save changes 'Shift + :wq!'
    g) see for reference
        vi /usr/bin/gnome-kiosk-script
    h)  --> restart session
        --> login to kiosk
        --> use a tool wheel in a corner - switch to Kiosk Script Session
            --> press Ctrl + Alt + F1 to proceed to the login screen
            --> use Admin account
            --> switch to root and close the kiosk session
                ---> 'who -u'
                ---> 'kill <process-number>'
            --> system automatically switch to the kiosk
            --> check is a file created in a path:
                /home/kiosk/.local/bin/gnome-kiosk-script
            --> replace message inside to:
                [to make any changes the user must be root]
                    #!/bin/sh
                    firefox --kiosk --browser http://webvids.miami-airport.com/webfids/webfids      // this is only an example to see does kiosk mode work as expected
            --> save changes
                (if the file does not exit please create it manually)
    i) logout from kiosk
    j) re-login to kiosk
        --> check the 'Kiosk Script Session (Wayland Display Server)' is selected 
            [the webside opens on fullscreen]
        --> switch to Admin
        --> kill the kiosk process          
        --> return to kiosk account
    l) vi /etc/gdm/custom.conf [root]
        --> uncomment:
          AutomaticLoginEnable = True
          AutomaticLogin = kiosk
        --> save

3. Install VLC
    a) use Software App
        --> search VLC
            --> Install
        [check is there an audio extension installed] | [if it does not then begin install]
        --> search for GStreamer Multimedia Codecs H.264
    [switch to root in the terminal]
    b) yum install vlc
    c) yum install vlc-cli      // command line to vlc

4. Manual VLC stream configuration
    a) log to Admin account
        --> open vlc application
        --> uncheck 'permit metadata to network access'                 
    b) Ctrl + S to open a window
        --> select Network
        --> type an URL address 
                rtsp://<camera-IP>:<port>/<file-name> [optional]              // ask your supervisor for the camera-IP and port or if it needs the username and password
        --> select Stream
            ---> leave as it is
            ---> Next
            ---> new target RTSP - display local
            ---> Next
            ---> Profile: Video-H.264 + MP3(MP4)
            ---> Next
            ---> Stream
        --> if the address is secured
            ---> enter correct data
            ---> select 'remember password'
        --> press OK

4a. Manual OBS Studio configuration
    a) log into Admin account
        --> open obs via terminal
	    ---> during initial settings mark a 'local recording'
        --> press plus sign at Video Source to add new source
        --> in source properties:
            ---> uncheck local file
            ---> in source bracket type rtsp://<login:passwd>@<camera-IP>:<port>/<file-name>       // ask your supervisor for the IP, port and/or the username and password
        --> press OK to save
        --> press RMB 
            ---> point on image distortion
            ---> select 'stretch to the screen option'
        --> copy the video source
            ---> adjust screens to the window
        --> lock sources
	--> on the navigation bar in the 'Panels' chart
            ---> unmark all of panels

5a. Configure the 'gnome-kiosk-script' file - VLC
    a) stay in the Admin account
    b) open terminal
        --> log into the root
        --> vi /home/kiosk/.local/bin/gnome-kiosk-script
        --> replace firefox path to:                                         
                vlc --fullscreen rtsp://<camera-IP>:<port>/<file-name> [optional]
            ---> save   
        --> restart desktop
    c) type kiosk password to authentication keys

5b. Configure the 'gnome-kiosk-script' file - OBS Studio
    a) stay in the Admin account
    b) open terminal
        --> log into the root
        --> vi /home/kiosk/.local/bin/gnome-kiosk-script
        --> replace VLC path to:                                         
                obs --safe-mode --always-on-top --scene <scene-name>
            ---> save   
        --> restart desktop

6. Install GDM Settings
    a) re-log to the Admin
    b) open Software App
        --> search for the app title above
    c) install and open GDM
    d) in the 'login screen' chart proceed to:
        --> disable restart buttons
        --> disable users list
        --> enable logo
        --> save and exit
    e) logout
    f) login to Admin again
    g) restart desktop

7. Finish the configuration
    a) check is the SSH service actived with port 22
        --> ss -plnt | grep 22
    b) active the SSH service
        --> systemctl enable sshd.service
        --> systemctl start sshd.service
        --> systemctl restart sshd.service
        --> systemctl status sshd.service
    [service is enable (green colour), server listening on :: port 22]

8. Troubleshooting - SEAHORSE Software - authentication keys || if you are using the VLC Software
    a) re-log to the Admin account
        --> open temrinal
        --> switch to root
        --> kill the kiosk session process
            ---> return to the Admin session
            ---> change kiosk account password
    b) install seahorse (keys & passwords)
        --> yum install seahorse -y
        --> open new terminal window
        --> type 'seahorse' to open an app
    c) solve the authentication issue during startup
        --> log into kiosk session - make sure there is a Gnome Session active option
        --> open an 'keys & passwords' app
	--> remove exitsing password session
            	[instead of confirm type correct password of account that is in use now]
        --> set new password to account key database
            ---> press right mouse button on the account key database chart
            ---> press 'change password' 
                [instead of confirm type correct password of account that is in use now]
            ---> set new empty password 
                [leave brackets blank]
            ---> contiune
            ---> continue
        --> restart desktop
        --> fill a vlc prompt with correct data
            ---> check 'remember password'
        --> press ok button
        --> restart desktop

