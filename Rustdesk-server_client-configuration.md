Rustdesk software configuration - Server [Ubuntu 22.04 or Server version] | Client [Windows 10]

| You have to have installed and configured an apache server first on your operating system. |
| It may be an Ubuntu 20.04/22.04 LTS or Ubuntu Server. |
| Than proceed to the rustdesk installation and configuration. |

| If you are using a Virtual Machine pay attention is there a Bridged Network Adapter setting selected, if doesn't please switch it. |

| Look at the video for more info about the config https://www.youtube.com/watch?v=t7UobpjDsRY |

1. Server configuration:

    a) enter to the Ubuntu
    b) open a terminal
        -> switch to root user or use root privileges by sudo
    c) open ports by ufw 
        -> ufw allow proto tcp from <server_ip> to any port 22 (you may open selected ports to connect them from anywhere or just your IP)
            or 
            --> ufw allow ssh 
        -> ufw allow 21115:21119/tcp
        -> ufw allow 8000/tcp
        -> ufw allow 21116/udp
            --> ufw enable
        -> ss -plnt (to look do all above ports allowed)
    d) download and install 
        -> cd /opt
        -> wget https://raw.githubusercontent.com/dinger1986/rustdeskinstall/master/install.sh
            --> chmod +x install.sh
        -> ./install.sh (during proccess choose options below)
            --> Your IP - type 1 and hit enter
            --> install HTTP - type 1 and hit enter
                ---> save your info, it looks like this one:

                        Your IP/DNS Address is ...
                        Your public key is ...
                        Install Rustdesk on your machines and change your public key and IP/DNS name to the above
                        You can access your install scripts for clients by going to http://...:8000
                        Username is admin and password is ...

                    (the public key is the most important in further steps)
            --> cd /opt/rustdesk
                ---> cat/nano id_ed25519.pub file to see the public key

2. Client configuration:
    | you will need a github account and vscode (using Git) to edit files, and do update a repository |

    a) fork a rustdesk repo to your repo
        -> master/main branch 
    b) clone the repository to vscode
        -> create or select a folder in your desktop where you want to save the repository
        -> git clone https://<repo-path>
        -> code . -r (to open repo folder)
        -> git branch (check are you on the correct branch - main/master)
    c) edit files

        + config.rs (lib/hbb_common/src/)
    
            pub const RENDEZVOUS_SERVERS: &[&str] = &["rs-ny.rustdesk.com"];
            pub const PUBLIC_RS_PUB_KEY: &str = "OeVuKk5nlHiXp+APNn0Y3pC1Iwpwn44JGqrQCsWqmBw=";

            to:

            pub const RENDEZVOUS_SERVERS: &[&str] = &[""];
            pub const PUBLIC_RS_PUB_KEY: &str = "";

        + flutter-nightly.yml (.github/workflows/) - comment these lines

            schedule:
                schedule build every night
                - cron: "0 0 * * *"
        
            to:

            # schedule:
            #   schedule build every night
            #   - cron: "0 0 * * *"

        + flutter-build.yml (.github/workflows/) - double comment (##) everything since

            The fallback for the flutter version, we use Sciter for 32bit Windows.
            build-for-windows-sciter:
            name: ${{ matrix.job.target }} (${{ matrix.job.os }})
            ...

        including these three lines above so it should looks like this:

            ## The fallback for the flutter version, we use Sciter for 32bit Windows.
            ## build-for-windows-sciter:
            ##    name: ${{ matrix.job.target }} (${{ matrix.job.os }})
            ...

        -> git commit -m "<your-comment>" to commit the changes
        -> git push to push changes to the repository
    
    d) add new repository secrets - github.com/<your-nickname>/<your-repo>/settings/secrets/actions
    
        +   Name*
                RENDEZVOUS_SERVER
            Secret*
                <server-ip>

        +   Name*
                RS_PUB_KEY
            Secret*
                <public-key>

        +   Name*
                API_SERVER
            Secret*
                http://<server-ip>:21116

3. Building client application:
    | you will use a Github Actions to build a package | 

    a) in your Github repository go to Actions
    b) select Flutter Nightly Build from All workflows
    c) press Run workflow - it should takes less then one hour
    d) when its completed jump to a Code chart and click on Tags --> nightly
    e) download a rustdesk-1.2.4-x86_64.exe
    f) run the client (you don't have to install the application to use it)

| Always remember to start server before client |

That's all!