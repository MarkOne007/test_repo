Linux kiosk mode (single-application-mode) configuration steps

1. Install Fedora Workstation 39
    a) set 
        --> timezone Europe/Warsaw
        --> language Polski
    b) reboot to finish installation
    c) create two accounts with passwords
        --> Admin as administrator
        --> kiosk as single user

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
        --> check the 'Kiosk Script Session (Wayland Display Server)' is selected ----
        --> the webside opens on fullscreen then switch to Admin account and kill the kiosk process          
        --> return to kiosk account
    l) vi /etc/gdm/custom.conf
        uncomment :
          AutomaticLoginEnable = True
          AutomaticLogin = kiosk
    
3. Install VLC
    [log into Admin and switch to root in the terminal]
    a) use Software App
        --> search VLC
            --> Install
        [check is there an audio extension installed] | [if it does not then begin install]
        --> search for GStreamer Multimedia Codecs H.264
    b) yum install vlc-cli      // command line to vlc

4. Configure the 'gnome-kiosk-script' file
    a) select Admin account
    b) open terminal
        --> log to root
        --> vi /home/kiosk/.local/bin/gnome-kiosk-script
        --> replace firefox path to:                                          // ask your supervisor for the camera-IP and port or if it needs the username and password
                vlc --fullscreen rtsp://camera-IP:port/file-name [optional]
            ---> save
        --> restart desktop

5. Install GDM Settings
    a) search for the app title above
    b) install and open GDM
    c) in the 'login screen' chart proceed:
        --> disable restart buttons
        --> disable users list
        --> enable logo
        --> save and exit

6. Manual VLC stream configuration
    a) open vlc application
    b) Ctrl + S to open a window
        --> select Network
        --> type an URL address 
                rtsp://camera-IP:port/file-name [optional]
        --> select Stream
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
    c) logout from Admin and switch to kiosk
    d) the stream is now working
