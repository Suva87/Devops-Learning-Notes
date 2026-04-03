# EC2 Stands for Elastic Compute Cloud

                       AWS Connection with system...

1. Sign in to aws.amazon.com/console
2. in the search type EC2
3. click on Launch instance
4. write a name (eg. Linux-for-devops)
5. in application -- for free reasons choose ubuntu
6. instance type - again for free reasons choose t2.micro
7. in key pair -- create a new key pair (for eg.. linux-for-devops-key)
8. click on "launch an instance"
9. click on the new generated instance-id
10. check the state - it should not be pending..
11. select the instance and click on connect..

SSH Connection: 12. on windows.. give permission to the key downloaded...
in powershell.. do this inside the folder where key exists..

        icacls docker_in_linux.pem /inheritance:r

        icacls docker_in_linux.pem /grant:r "$($env:USERNAME):(R)"

        icacls docker_in_linux.pem /remove "BUILTIN\Users"

13. then do ssh...

# Ubuntu Shell Commands:

1.  ls - shows list of files in the directory
    .) ls -l ... gives you details of the directory that was created
    .) ls -ltr ... gives you more details and shows links as well
    .) ls -a ... this shows all the files and folders present in the directory including the hidden ones..

2.  date - shows date and time

3.  mkdir - make a directory
    eg. mkdir devops - this will create a directory named as devops

4.  pwd - present working directory

5.  touch - create a file inside a directory
    eg. touch newfile.txt - this will create a new file named newfile.txt in the present working directory.
    .) when you do ls -l ... when it shows a list.. a directory detail will always start with d...
    eg.. drwzxrr

6.  clear - clears the screen

7.  cd - change directory ... used to get inside a directory or out of a directory...
    eg.. cd devops, cd home/ubuntu/devops/
    .) cd.. to go outside the directory

8.  rm - remove..
    .) rm newfile.txt ... this will delete the file but this cannot be used to remove directory
    .) rm -r directory .... this will remove the directory and all the files inside it..
    .) rmdir ... rmdir removes only empty directories. If files exist, the command fails.
9.  echo - to print anything on screen...
    eg.. echo "hello world"
    .) to write something inside a file...
    echo "hello world" > demofile.txt
    ... ">" this chanellize the output to someother file..
    ... this also creates the file demofile.txt if there is no such file in the directory..

10. cat - to see what is inside a file...
    eg... cat demofile.txt .... output will be hello world..
    .) zcat - to see what is inside a zipped file..

11. head - this prints the top no. of lines which is there in the file..
    .) syntax.. head -n (number) myfile.txt
    eg.. head -n 5 myfile.txt ... will show first 5 lines

12. tail - this prints the bottom no. of lines which is there in the file..
    syntax.. tail -n (number) myfile.txt
    .) tail -f - this keeps monitoring the file and shows new lines added in the file..
    eg.. tail -f myfile.txt
    press ctrl + c to come out
13. less - this shows the file in a paginated way..

14. cp - copy
    .) copy a file inside a folder..
    eg.. cp newfile.txt devops/ , syntax is cp "source/file" "destination/"
    .) you can use this even if you are not inside the folder.. you can copy a file from any folder and place it inside any other folder..
    eg.. cp devops/newfile.txt cloud/ , syntax is the same.. cp "source/file" "destination/"
    .) you can also copy the whole directory to another directory.. use -r
    eg.. cp -r devops cloud/

15. mv - move
    .) move syntx is -- mv "filename" "path of the destination from the source" when you are inside the destination folder
    .) you can also use move for renaming...
    eg.. if you want to rename a directory named devops to linux-for-devops... mv devops/ linux-for-devops

16. wc - tells the word count
    .) wc myfile.txt... if this had content as hello word, then wc will say 1 2 2 which means 1->line, 2-> words, 2-> bytes

17. hardlink - A hard link is another name for the same file (same inode). Both files point to the same data on disk. If one file is deleted, the data still exists as long as another hard link exists.
    .) syntax eg... ln original.txt hardlink.txt
    eg... lets say the path to original file is cd linux-for-devops/cloud/.. and it contains content.. "hello world"...
    to make a harlink file ... ln "file path" "filename" ... here it will be..
    ln linux-for-devops/cloud/original.txt hardlink-file-of-original.txt
    .) In hard link, changing content in either file updates both files because they point to the same inode.

18. softlink - Acts like a shortcut. Points to the file path, not the inode. If the original file is deleted, the soft link breaks. If the original file was changed then the softlink also shows the new content..
    .) syntax . ln -s "file path" "filename"

19. cut - it gives you a new output by cutting that much byte from the file..
    eg.. if myfile.txt has the content "hello world", and i say - cut -b 1 myfile.txt.. it will give output by give me 1 byte out of "hello world".. may be "h" as the output
    eg... cut -b 1-4 myfile.txt ... this will give 4 bytes as output.. may be "hell"

20. tee - Reads input from stdin. Writes output to both screen AND file
    .) syntax - command | tee file.txt
    eg.. Save output & show on screen - ls | tee list.txt
    Append instead of overwrite - ls | tee -a list.txt - echo "hello" | sudo tee file.txt

21. diff - shows difference between 2 files...
    .) diff hello.txt devops.txt .. output.. if there is a difference between these two then both outputs will be displayed..

22. VI Editor - this is to edit any file..
    eg.. vi demofile.txt ... after it opens.. press i for insert.. press on "esc" to come out of insert mode ... to come out of vi mode.. ":wq" stands for write and quit..
    in browser while using aws ubuntu.. this might mal function sometimes.. so reset it to use..
    vim demofile.txt

23. ssh - stands for secure shell.. used for remote login.. it gets connected via a port..
    .) ssh access is on port 22 in general
    .) the remote server should have public key and user(local) should have a private key..
    .) so whenever key is created.. in background there is a command that runs.. "ssh-key-gen"... this creates 2 keys.. public key and private key..

24. SSH - aws ssh client with windows terminal steps:

        1. Sign in to aws.amazon.com/console
        2. in the search type EC2
        3. click on Launch instance
        4. write a name (eg. Linux-for-devops)
        5. in application -- for free reasons choose ubuntu
        6. instance type -  again for free reasons choose t2.micro
        7. in key pair -- create a new key pair (for eg.. linux-for-devops-key)
        8. While creating EC2 instance, AWS downloads a file like: linux-for-devops-key.pem
        9. Open Windows Terminal / PowerShell - Go to the folder where key is downloaded - cd Downloads
        10. Check if file exists: ls
        11. Fix permission for key file (IMPORTANT) - Windows is strict about SSH key permissions.
            .) Run this command:
                icacls linux-for-devops-key.pem /inheritance:r
                icacls linux-for-devops-key.pem /grant:r "$($env:USERNAME):R"
        12. Copy EC2 Public IPv4 address From AWS EC2 console:
        13. Connect to EC2 using SSH: Run ssh -i linux-for-devops-key.pem ubuntu@<PUBLIC-IP>
            .) eg.. ssh -i linux-for-devops-key.pem ubuntu@13.234.56.78
        14. First-time connection message: Are you sure you want to continue connecting (yes/no)? yes
        15. Successful Login Output: If connected successfully, you will see: ubuntu@ip-172-31-xx-xx:~$

25. Disk Usage - df - tells us more about storage of the disk
    .) df -h : this gives more detailed information
    .) du : this gives information about the disk used by a particular folder
    ..eg - du devops
    .) free : this shows in total disk space
    .) free -h : this gives more detail data

26. nohup : this creates a file called nohup.out and stores the output of any command in that file.
    eg.. nohup free-h &
    .) used case scenario is that it does not delete the first output and then takes the second output in it. So it can be used to collect logs.. so for eg.. it already stores the output of free-h , now when we run nohup df -h , it now stores both the output.. it just keeps appending

# SYSTEM LEVEL COMMANDS

27. uname - this tells us which platform we are running the command

28. uptime - this tells us the time since which the system is running

29. who - this tells us about which user looged in at what time..

30. whoami - current user USERNAME

31. which - tells us the location of the file..

32. id - to find id of user and groups

33. sudo - to do anything bypassing root user permission ... superuser do .. sudo ... its a group with adminintrator rights.. its gives you superuser permission..

34. shutdown - sudo shutdown..

35. reboot - sudo reboot..

36. apt - its a package manager..install anythird party software.. but for that system should be updated.. so first do sudo apt-get update... once updated.. then -
    .) if you have it in the system, then just say sudo apt install docker.io
    .) if you do not have it in the system, the we should get it from the internet...
    sudo apt-get install docker.io
    .) to uninstall .. sudo apt remove docker.io
    .) similar commands of package managers for other linux models - yum, dnf, pacman, portage..etc.

# USER & GROUP MANAGEMENT COMMANDS

35. useradd -
    command is : sudo useradd -m : this -m means add the user and make a directory..

    .) multiple user add -
    for user in user1 user2 user3; do
    sudo useradd -m $user
    done

36. passwd - this will add a password for the new user..
    eg.. sudo passwd user-2 : this will ask for a new password.. but you will not able to see what you type for security reasons.. but you can create the password and hit enter..
    .) to change the password for any user... sudo passwd user-2

37. su - switch user  
     eg.. su user-2
    .) to switch back to primary user just type "exit"

38. cat/etc/passwd - this will show all the list of users..

39. userdel - to delete userdel
    eg. sudo userdel user-2

40. groupadd -

        suppose there is a company abc ltd, where there are 2 devops eng (def, ghi ) and 3 testers (jkl, mno, pqr).. so instaed of defining roles and permissions of each user seperately, we can create groups in the name of devops and tester and add them accordingly to these groups..

        first add these users.. then create the groups ... then add these users to the required groups..

        eg..

        step 1: add the users and the groups

            for user in def ghi jkl mno pqr; do
                sudo useradd -m $user
            done

            AND THEN.....

            for group in devops tester; do
                sudo groupadd $group
            done

            .) to check the group list, you can check by .. cat/etc/group

        step 2: add users to a group..

        command - sudo gpasswd -a (user) (group)
                eg.. sudo gpasswd -a def devops

        to set multiple user in a group in a single instance..
        command - sudo gpasswd -M (user),(user),(user).. (group)
                eg.. sudo gpasswd -M jkl,mno,pqr tester
            note: this would set the users in the group and if there was any previous users, it would get removed..

        to add multiple user in a group in a single instance..
            for user in ubuntu user-2 rocky tiger lion; do
                sudo gpasswd -a $user devops
            done
        note: this will not replace the existing users in the group..

41. groupdel - to delete a group ..
    eg.. sudo groupdel tester

# FILE PERMISSION COMMANDS

42. ls-l - this command was described in earlier notes as well, but now it is time to mention that this command helps to see the permission of the file or directory.. this also tells the owner and group the file can be accessed in currently..

        eg.. drwxrwxr-x : here d stands for directory as we have studied earlier..
                            now read the rest in triplets: rwx (user), rwx (group), r-x (other users) ... so :
                                r -> read, w -> write, x -> execute

        .) File permission chart in Linux:

             no need to learn this by heart.. just remember the follwoing pattern..
             1. first write 0-7 vertically
             2. now remember the last line of the column will be altrenative 1-1 .. i.e. - x - x ...
             3. now the middle column pattern is 2-2 i.e - - w w - - w w
             4. now the 1st one pattern will be 4-4 i.e - - - - r r r r

                    so in column it will look like:

                    0   -     -        -
                    1   -     -        x
                    2   -     w        -
                    3   -     w        x
                    4   r     -        -
                    5   r     -        x
                    6   r     w        -
                    7   r     w        x

            5. Now moving back to the ls -l eg. of above.. so you can know the permission of the file as below:
             so drwxrwxr-x means directory775

43. chmod - this we will use to change the permission of a file..
    eg.. when we do ls-l, lets say we get.. drwxrwxr-x which we want to cheange to all rwx, so we will command:
    sudo chmod 777 (folder/file name)
    eg.. sudo chmod 777 cloud

44. umask - it stands for user-file-creation-mode-mask. when any file or directory is created, then a four digit octal number is used by Linux to determine the by default file permission
    this by default value table is as below:

        umask user group  others
        0000  rwx  rwx    rwx
        0002  rwx  rwx    r_x
        0007  rwx  rwx    _ _ _
        0022  rwx  r_x    r_x
        0027  rwx  r_x    _ _ _
        0077  rwx  _ _ _  _ _ _

45. chown - change owner..

    command - sudo chown (new user/owner name) (filename)

        eg .. suppose when we do ls -l, we see that a file named new.txt has current ownership as ubuntu.. we want to change the owner of the file to user-2
        sudo chown user-2 new.txt

46. chgrp - change group...

    command - sudo chgrp (new group name) (filename)

        eg... from the above eg.. if we want change the file group to devops..
        sudo chgrp devops new.txt

# COMPRESS A FILE

47. zip - to compress a cluster of files or files + directory..

        first you need to install zip.. as we have studied earlier, to install zip..
        sudo apt install zip

        then -

        command -
            for zipping file..
            zip (zipped filename.zip) (to be zipped filename)

            for zip diretory..
            zip  -r (zipped directoryname.zip) (to be zipped directory name)

48. unzip - to unzip a zipped folder..

         command: unzip (zipped file name)

49. tar - the tar command creates, maintains, modifies, and extracts files that are archived in the tar format.

        command -
        tar [-] A --catenate --concatenate | c --create | d --diff --compare |
            --delete | r --append | t --list | --test-label | u --update |
            x --extract --get [options] [pathname ...]

            A, --catenate,
            --concatenate	Append tar files to an archive.
            c, --create	Create a new archive.
            d, --diff, --compare	Calculate any differences between the archive and the file system.
            --delete	Delete from the archive. (This function doesn't work on magnetic tapes).
            r, --append	Append files to the end of a tar archive.
            t, --list	List the contents of an archive.
            --test-label	Test the archive label, and exit.
            u, --update	Append files, but only those that are newer than the copy in the archive.
            x, --extract, --get	Extract files from an archive.

# COPY

50. scp - secure copy (used to tranfer from local to server and viceversa)

        .) suppose you have local file in documents/my-files/secret-file.txt and i want to copy it to the aws server  home/ubuntu/linux-for-devops/secure-copy

        go inside the folder in the local from where you want to copy..

        command: scp -i "(path to pem in local)" (server-username@server-address):(FULL path INCLUDING file/dir name on server) (FULL local destination path)

            eg.. if you are inside documents/my-files ..

            scp -i "users/my-pc/downloads/linux-for-devops-key.pem" ubuntu@13.234.56.78:/home/ubuntu/linux-for-devops/secure-copy/secret-file.txt users/my-pc/documents/my-files/

        .) now viceversa - suppose you have server directory or folder in the name "secure-copy"  in home/ubuntu/linux-for-devops and you want to copy it to the local documents/my-files/

            again the same thing you need to follow:

            command:

            scp -i "(path to pem in local)" -r (server-username@server-address):(FULL path INCLUDING directory name on server) (FULL local destination path)

                eg..
                scp -i "users/my-pc/downloads/linux-for-devops-key.pem" -r ubuntu@13.234.56.78:/home/ubuntu/linux-for-devops/secure-copy users/my-pc/documents/my-files/

51. rsync - this to sync any copied folder from remote to local.. rsync compares source vs destination and transfers only the differences.
    .) suppose You copied a folder from remote to local earlier. You added a new file locally in the folder - rsync-file.txt . You now want to sync ONLY the changes back to remote.

        command -

        rsync -avz --progress
            -e "ssh -i (path to pem in local)"
            (FULL local folder path INCLUDING trailing /)
            (server-username@server-address):(FULL remote destination path)

            explaination: Flags explained

                    -a   archive (permissions, timestamps, symlinks)
                    -v   verbose (shows what is happening)
                    -z   compress during transfer
                    --progress  show live progress

            First time? Do a SAFE DRY RUN.. Shows what WOULD be copied... Nothing is actually transferred

            eg..
            DRYRUN:
            rsync -avz --dry-run
                -e "ssh -i users/my-pc/downloads/linux-for-devops-key.pem"
                users/my-pc/documents/my-files/secure-copy/
                ubuntu@13.234.56.78:/home/ubuntu/linux-for-devops/secure-copy/

                Output you should expect (example)
                sending incremental file list
                rsync-file.txt
                sent 312 bytes
                total size is 10.5K

            ACTUAL SYNC:
            rsync -avz --progress
                -e "ssh -i users/my-pc/downloads/linux-for-devops-key.pem"
                users/my-pc/documents/my-files/secure-copy/
                ubuntu@13.234.56.78:/home/ubuntu/linux-for-devops/secure-copy/

           DELETE REMOTE FILES LOCALLY: This Deletes remote files NOT present locally
           rsync -avz --delete
                -e "ssh -i key.pem"
                local-folder/
                user@server:/remote-folder/

            This Deletes remote files NOT present locally

# NETWORKING COMMANDS

    CONCEPTS:

        1) What does “network” mean?

                A network is simply a group of devices (computers, phones, servers, printers) that can send data to each other.

                What is “data” here?

                Data means information, like:

                text (chat message)

                images

                videos

                files

                website pages

                login credentials (username/password)

                API response (JSON)

                How do devices “send data”?

                They use:

                cables (ethernet / fiber)

                Wi-Fi (radio signals)

                mobile network (4G/5G)

                combinations of these through the internet

        2) What is the Internet?

                The internet is a very large network made by connecting many smaller networks together:

                your home Wi-Fi network

                your office network

                your phone’s mobile network

                Amazon (AWS) network

                Google network

                So, internet = “network of networks”.

        3) What is an “address” in networking?

                Just like a courier needs your home address to deliver a parcel, computers need an address to send data to the right device.

                That address is called an IP address.

        ✅ 4) What is an IP address (in simple words)?

                An IP address is a number that identifies a device on a network, so other devices can find it and talk to it.

                Example:
                13.234.56.78

                Why is it called “IP”?

                IP = Internet Protocol
                Protocol means a set of rules that computers follow to communicate.

                So “Internet Protocol” is basically:

                the rules for addressing and sending data across networks.

        5) Why are there TWO kinds of IP addresses?

                Because we need:

                addresses that are usable inside a private place (like your home or AWS internal network), and

                addresses that are usable on the open internet (worldwide).

                The two types:

                Private IP (used inside a private network)

                Public IP (used on the internet)

                Let’s explain each properly.

        ✅ 6) What is a Private IP? (from basics)

            6.1 What is a “private network”?

                A private network is a network that is not directly visible to the whole internet.

                Examples:

                Your home Wi-Fi network

                Office network

                A company’s internal network

                AWS VPC network (AWS internal private network)

                Why call it “private”?

                Because devices inside it usually cannot be directly reached from the outside internet unless special settings are done.

            6.2 What does a Private IP do?

                A Private IP is an address used inside a private network to allow devices inside that network to communicate.

                Example in home:

                Your phone might be: 192.168.1.20

                Your laptop might be: 192.168.1.10

                Your router might be: 192.168.1.1

                Your phone and laptop can talk to each other inside your home network using these private IPs.

            6.3 Why do we need Private IPs?

                Because there are billions of devices in the world, and public IP addresses are limited.

                So we reuse private IP ranges inside homes/offices/AWS networks.

                What does “IP ranges” mean?

                Instead of one IP, we have groups of IPs reserved for private use.
                Common private ranges include:

                10.x.x.x

                172.16.x.x to 172.31.x.x

                192.168.x.x

                These are like “reserved building numbers that only work inside your colony”.

        ✅ 7) What is a Public IP? (from basics)

            7.1 What does “public” mean here?

                Public means:

                visible and reachable on the internet.

                If a device has a Public IP, other devices worldwide (in theory) can send data to it.

                Example:

                Your EC2 instance gets a public IP like 13.234.56.78

                This means your laptop can reach it over the internet:

                SSH login

                Website

                API calls

            7.2 Why do we need Public IPs?

                Because internet is global.
                To send data to a server sitting in AWS, we need an address that works globally.

                Public IP is like:

                the building’s address that courier from anywhere in the world can find.

        ✅ 8) So why TWO kinds? (Most important explanation)

                You can think like this:

                Private IP = inside building navigation

                Within a building/office:

                every room has an internal number

                room numbers can repeat in other buildings

                Public IP = outside building address

                To reach the building from outside, you need:

                street address

                city

                pincode

                Same in networking:

                Private IP works inside a private network

                Public IP works on the internet

        ✅ 9) Key Real-World Example: Home Wi-Fi

                Let’s make it super concrete.

                In your home:

                Your phone has a private IP: 192.168.1.20

                Your laptop has a private IP: 192.168.1.10

                These private IPs are NOT visible on the internet.

                But your internet provider (ISP) gives your home router one public IP.

                So from outside world:

                your entire house appears as one public IP

                Inside your home:

                all your devices have their own private IPs

        ✅ 10) Who gives IP addresses?
                Private IP assignment

                Usually your router assigns private IPs to devices automatically using something called DHCP.

                What is DHCP? (nested explanation)

                DHCP = Dynamic Host Configuration Protocol
                Meaning:

                automatic system that gives your device an IP address when it connects to the network.

                When your phone joins Wi-Fi:

                it asks: “Please give me an IP”

                router replies: “Take 192.168.1.20”

                phone uses it

                Public IP assignment

                Public IP is usually provided by:

                your ISP (home internet)

                your cloud provider (AWS gives EC2 public IP)

        ✅ 11) How does private ↔ public communication happen?

                This is where a critical term comes:

                NAT (Network Address Translation)
                What is NAT in simple words?

                NAT is the trick that allows many private devices to share one public IP.

                Your router does NAT.

                Example:

                Your phone (private IP 192.168.1.20) opens google.com.
                Google only sees your home’s public IP, not your phone’s private IP.

                Your router keeps a “mapping table” like:

                request from phone → goes out using public IP

                response comes back → router sends it to correct private device

                So NAT is like:

                the receptionist of a company who receives all parcels at one public address and then delivers to the correct internal room.

        ✅ 12) Where does AWS EC2 fit into this?

                AWS gives:

                Private IP inside AWS internal network (VPC)

                optional Public IP to allow you to access from internet

                So EC2 usually has BOTH:

                Private IP: used by other AWS services (internal traffic)

                Public IP: used by you from your laptop to SSH/login

        ✅ 13) Why should DevOps care?

                Because many times:

                your app works inside AWS (private traffic)

                but doesn’t open to public internet (public access issue)

                Example:

                Your backend running on EC2

                But you can’t open it in browser
                Possible reasons:

                app not listening

                port blocked

                security group blocked

                no public IP

                All these come from understanding public vs private IP.

        🔥 Next Concept: What is a “Port” and why we talk about ports?
        14) What is a port?

                If IP is the address of the computer, then port is the address of a specific app/service running on that computer.

                Why?

                One computer can run many services:

                SSH login

                website

                database

                email service

                So we need a “door number” for each service.

                Example:

                13.234.56.78:22 → SSH

                13.234.56.78:80 → Website (HTTP)

                13.234.56.78:443 → Website (HTTPS)

                What does : mean?

                It separates:

                IP address (machine)

                port (service)

        15) What does “listening on a port” mean?

                “Listening” means:

                The service is running and waiting to accept connections on that port.

                If SSH is running, it “listens” on port 22.

                If no service is listening:

                connection will fail

                it’s like knocking but no one is home

        ✅ 16) TCP and UDP (from basics, nested)

            16.1 What is a “protocol” again?

                Protocol = rules for communication.

                TCP and UDP are two different rule sets for sending data.

            16.2 TCP (Transmission Control Protocol)

                TCP is like a reliable courier:

                It checks if message reached

                If not, it resends

                It keeps order correct

                Why do we need TCP?

                Because accuracy matters for:

                logins (SSH)

                web pages

                file transfers

                database communication

                So TCP is used where “missing data” is unacceptable.

            16.3 UDP (User Datagram Protocol)

                UDP is like fast delivery without confirmation:

                no guarantee

                no resending

                can arrive out of order

                Why use UDP then?

                Because for:

                video/audio live calls

                gaming

                streaming
                a little data loss is okay, but speed is more important.

        ✅ 17) DNS (from basics, nested)

            17.1 Why DNS exists?

                Humans remember names:

                google.com

                facebook.com

                Computers route using IP:

                142.250....

                So DNS converts:

                domain name → IP address

            17.2 What is a “domain name”?

                A domain name is a human-friendly name that points to a server.

                Example:

                mycompany.com

            17.3 Why do we “lookup” DNS?

                Because if DNS fails:

                you can’t reach website by name

                even if the server is working

                DNS lookup helps you confirm:

                whether the name resolves to an IP

                which IP it resolves to

        ✅ 18) Network Interface, MAC address (nested)

            18.1 What is a Network Interface?

                A network interface is the “network connection part” of a device.

                Examples:

                Wi-Fi adapter

                Ethernet card

                virtual adapter (in servers)

            18.2 What is a MAC address?

                MAC is a unique hardware ID of the network interface.

                MAC is mostly used inside local networks, not on the whole internet.

                Think:

                IP can change

                MAC is tied to the hardware

        ✅ 19) Routers between you and destination (nested)

            19.1 What is a router?

                A router is a device that decides:

                “Where should this data go next?”

                It’s like a traffic police / highway junction for data.

            19.2 Why are there multiple routers?

                Because the internet is huge.
                Your data travels through many networks.

                So it goes hop by hop:

                your router

                ISP router

                backbone routers

                destination network routers

                destination server

                Each “hop” is one router step.

52. ping - this is to check if the route is tranferring the data to the host location and viceversa or not...

        eg.. ping mywebsite.com

        How does ping work internally?

            ping sends a small test message to the target computer.

            The target computer: receives the message and sends back a reply saying “I am alive”
                This message is called an ICMP echo request, and the reply is an ICMP echo reply.

            What is ICMP? (nested explanation)
                ICMP = Internet Control Message Protocol
                It is a special protocol used only for testing and error reporting, not for sending real data like files or websites.

53. netstat - it shows all the active internet connections to the system.. you might need to have to install net-tools for this in linux..

        Why do we need netstat?
            Even if a computer is reachable, a service (like SSH or a website) might not be working.
            So we need to answer:
                Which services are running?
                Which ports are open?
                Which connections are active?

                        A service is a program that:
                            runs in the background
                            waits for network requests

                                Examples:
                                SSH service (for login)
                                Web server (for websites)
                                Database server

                What is a “connection”?
                    A connection means:
                        two computers are currently talking to each other
                        data is flowing between them

        Installing netstat (because it may not exist)

            sudo apt install net-tools

        Command: netstat -tuln
            Explanation of flags:
                -t → show TCP connections
                -u → show UDP connections
                -l → show listening services
                -n → show numbers instead of names (faster)

        What does “listening” mean again?
            Listening means: A service is running and waiting for someone to connect.

54. ifconfig -

        ifconfig shows:
            interfaces
            IP addresses assigned
            MAC addresses
            whether interface is active

        ifconfig is an older command, but still commonly used in tutorials and interviews.

55. arp - IP to MAC mapping

        What problem does ARP solve?
            Inside local networks:
            computers know IP
            but data is sent using MAC
            ARP maps:
            IP address → MAC address

        Command: arp -a

        ARP = Address Resolution Protocol

        It answers: “Which MAC address belongs to this IP?”

56. ip - Understanding your machine’s network identity (MODERN & IMPORTANT)

        A computer cannot communicate on a network unless it knows:
            who it is (its address)
            how it is connected (which network path it uses)

        The ip command helps us answer:
            “How is this machine connected to the network right now?”

        Why not use ifconfig?
            old
            limited
            no longer actively developed

        Modern Linux systems use ip, which:
            replaces ifconfig
            replaces route commands
            replaces arp commands

        Using ip, you can see:
            network interfaces
            IP addresses (private/public)
            whether interface is active
            routing paths

        Command: ip a

        Example output (simplified):4
            You might see something like:
                eth0: UP
                inet 172.31.5.120

                eth0 is the name of a network interface. It is the network card (real or virtual) through which the machine connects to the network.
                    On:
                        EC2 → eth0 is the main network interface
                        Laptop → could be Wi-Fi or Ethernet

                inet stands for Internet address, which means:
                        IPv4 address assigned to this interface

57. traceroute -

        traceroute shows:
            all the hops
            all the routers
            time taken at each hop

        command: traceroute (destination)

58. tracepath - Traceroute without special permissions.
    Some systems restrict traceroute.

59. mtr - Combined ping + traceroute
    Sometimes:
    network is reachable
    but unstable or slow

        What does mtr show?
            hops (routers)
            packet loss
            delay at each hop
            updates continuously

        commands: mtr google.com

60. nslookup - Checking DNS resolution . It asks: “What IP address does this domain name point to?”
    nslookup google.com

61. telnet - Testing if a port is reachable

            What does “reachable port” mean?

                Even if:
                    computer is reachable

                A service might not be reachable if:
                    port is closed
                    firewall blocks it

            What does telnet do?
                It tries to:
                open a connection
                to a specific port

        command: telnet (destination) (port)

62. hostname - Name of the machine . Hostname is: a human-friendly name for a computer

63. iwconfig - Wireless interface info

64. ss - Viewing listening services (modern netstat)

        Why ss is important

            ss shows:
                sockets
                ports
                services

        What is a socket? (nested)

            A socket is:
                combination of IP address + port
                It uniquely identifies a communication endpoint.

        command: ss -tuln

        If your backend app runs on port 3000:
            ss will show a listening entry on port 3000
            if not shown → app is not running or crashed

65. dig - Deep inspection of DNS behavior.

        Why nslookup is sometimes not enough
            nslookup gives basic answers.

        Sometimes:
            DNS is misconfigured
            multiple DNS records exist
            caching causes confusion
        dig gives complete detail.

    What does dig show?
    which DNS server responded
    time taken
    DNS records
    authority information

66. nc (netcat) - talking directly to ports.

        Why netcat exists
            Sometimes you want to:
                check if a port is open
                send raw data
                test a service without browser

        What is a “raw connection”?
            Raw means:
                no browser
                no special protocol handling
                direct communication

        command: nc -zv google.com 80

67. whois - Understanding domain ownership

        Why domain ownership matters
            Sometimes:
                domain expired
                DNS not updating
                ownership dispute
                whois helps verify facts.

        WHOIS is a public registry that stores:
            domain owner
            registration date
            expiry date
            registrar details

        command: whois google.com

68. ifplugstatus - Checking physical connectivity

        On physical machines:
            cable may be unplugged
            network may appear down
        This command checks: “Is the network cable connected?”

        Why rare in cloud
            Cloud servers:
                use virtual networking
                don’t have physical cables

69. curl - Talking to web servers and APIs (CRITICAL)

        Why curl is essential
            Almost everything today is:
                web-based
                API-based

        curl allows you to:
            send requests
            see responses
            debug servers

        What is HTTP? (nested)
            HTTP = HyperText Transfer Protocol
            It defines:
                how requests are sent
                how responses are returned

        command: curl google.com
            Meaning:
                send HTTP request
                display response

    .) Headers-only request:
    curl -I google.com

            Headers are:
                metadata
                status codes
                server information

            Example:
                200 → success
                404 → not found
                500 → server error

    .) to get any data from server:
    curl -X GET (api address) | jq

70. wget - to download any file from the internet to the linux system..

        wget (link of the file to download)

71. route -

        Why do we need route?
        Even if your machine has:
            an IP address
            Wi-Fi/Ethernet connected

        …it still must answer: “If I want to send data to some IP address, where do I send it next?”

        What is a “routing table”?

        Think of a routing table like this:
            You are in a city (your laptop)
            You want to send a parcel to another city (destination IP)
            You need to know: “Should I send it directly?” “Or should I send it to the courier hub (router/gateway) first?”

        A routing table is simply a list of rules that say: “For these destination IP ranges, send traffic to this next hop.”

        What is a “gateway” or “default gateway”?

        A gateway is usually your router.
            If the destination is inside your local network → you don’t need a gateway
            If the destination is outside your local network → you send it to the gateway
        Default gateway means: “If I don’t know any special route, send everything to this router.”

        What does route command show?
        It shows your machine’s routing table. route -n
        What you will see in output?
            Destination
                This is “where you want to go”.
                Example:
                192.168.1.0 means that whole local network
                0.0.0.0 means “anything else” (default route)

            Gateway
                Where to send the traffic next.
                Example:
                192.168.1.1 (your router)
                0.0.0.0 might mean “directly connected”

            Genmask
                This is the subnet mask rule for that destination.
                Example:
                255.255.255.0

            Iface
                Which network interface to use.
                Example:
                eth0 (Ethernet)
                wlan0 (Wi-Fi)

            The most important entry: default route

            The most important line in routing table is often:
                Destination: 0.0.0.0
                Gateway: 192.168.1.1

            Modern note:
            In modern Linux, route is older. The modern replacement is: ip route

# PRO LINUX COMMANDS

72. awk - Reading and processing text like a “mini-programming language”

        Why do we need awk?
            Linux servers produce a lot of text:
                log files
                command outputs
                CSV-like data
                lists of users, services, IPs

            Often you want to:
                pick only one column
                filter rows
                calculate totals
                generate a new formatted output
                awk is a tool that can do this.

        awk is not just a command — it’s like a small programming language built for text processing.

        What does awk work on? (the concept of “lines” and “fields”)?

            Most Linux text is arranged like:
                lines (rows)
                each line has fields (columns), separated by spaces or commas etc.
                awk can refer to them like:
                    $1 = first field
                    $2 = second field
                    $3 = third field
                    …and so on.

            command: awk 'action' filename
            Or with command output: some_command | awk 'action'

    Example 1:
    The simplest awk example: print a column:
    suppose you have a file people.txt which has:
    rahul 5000
    mansi 7000
    suva 9000
    Now print only names (first column):
    awk '{print $1}' people.txt
    Print only salaries (second column):
    awk '{print $2}' people.txt

            What is {print $1}?
                This is the “action block”.
                { ... } means “do this for every line”
                print means “output to screen”
                $1 means “first field”
                So it reads as: “For each line, print the first column.”

    Example 2:
    Filtering with awk (like “if” logic): print only lines where salary > 6000:
    awk '$2 > 6000 {print $1, $2}' people.txt
    Meaning:
    $2 > 6000 is the condition
    if true, it prints

    Example 3:
    awk can calculate totals (programming power): Sum salaries:
    awk '{sum = sum + $2} END {print "Total =", sum}' people.txt
    Explanation:
    sum = sum + $2 adds each salary to sum
    END { ... } runs once at the end (after reading all lines)

    other expamples used for log files:
    awk /DEBUG/'{print $1,$2,$4,$5}' app.log > only_degug.log
    this is to show only the lines with DEBUG and redirect it to another file

         awk '/DEBUG/ {count++} END {print count}' app.log
            this is to count the no. of times there are DEBUG and print the count

    Why DevOps uses awk?
    Because it helps extract details quickly:
    total disk usage numbers
    count of errors in logs
    parsing ps output (running processes)
    parsing df -h output (disk)
    parsing IP addresses etc.

    So this is like a mini program.

73. sed - Editing text automatically (find/replace in files)

    Why do we need sed?
    Sometimes you need to:
    replace text in a file
    remove a word
    change configuration files
    edit logs or outputs
    But doing manual editing in vim is slow for repeated operations. sed automates editing.
    sed = stream editor
    Meaning:
    it reads input line-by-line
    applies edits
    outputs modified text

        command: sed 'operation' filename
        Or with command output: some_command | sed 'operation'

        Example1:
            Most common sed use: replace text: Suppose you have a text file demo.txt which has "Hello World"
            Replace world with linux:
                sed 's/world/linux/' demo.txt
                Explanation:
                    s means “substitute”
                    s/old/new/ pattern
                    It prints changed output to screen but does NOT change the file yet

        Example2:
            Replace ALL occurrences in a line: If a line has multiple matches:
                sed 's/world/linux/g' demo.txt
                    g means “global” (replace all matches in that line)
            Actually modify the file (in-place):
                sed -i 's/world/linux/g' demo.txt
                Important warning: -i changes the real file.
            Safer pattern:
                sed -i.bak 's/world/linux/g' demo.txt
                This creates a backup: demo.txt.bak

        Example 3:
            Delete lines using sed:
                Delete a specific line number (example delete line 3):
                    sed '3d' people.txt
                    d = delete
                Delete a range (lines 2 to 4):
                    sed '2,4d' people.txt

    Why DevOps uses sed ?
    Because it automates configuration editing:
    changing ports in config files
    replacing environment variable values
    updating domain names
    mass editing large files quickly

74. grep - Searching text (finding the lines you care about)

    Why do we need grep?
    Servers produce massive text
    logs (/var/log/...)
    outputs (ps, netstat, ss)
    config files (nginx.conf, .env)
    You often want: “Show me only lines that contain this word.”
    Basic meaning of grep: Search lines and print only matching lines.

    Command: grep "error" mylog.txt

        Example 1:
            Search a word in a file:
                grep "error" mylog.txt
                This prints only lines containing “error”.

        Example 2:
            Case-insensitive search: If you want Error, ERROR, error all matched:
                grep -i "error" mylog.txt

        Example 3:
            grep "error" *.txt

        Example 4:
            Search inside directories (recursive search): If you want to search all files inside a folder:
                grep -R "password" /etc/
                    -R means “go into subfolders too”
                    it searches everything under that path

        Example 5:
            Show line numbers (super helpful):
                grep -n "error" mylog.txt

        Example 6:
            Invert match (show lines that do NOT match):
                grep -v "error" mylog.txt
                    This shows all lines except those containing “error”.

        Example 7:
            Count matches (how many lines matched):
                grep -c "error" mylog.txt
                    This returns a number.

    Why grep is “programming-friendly”?
    Because you can combine grep with other commands.
    Example idea (not using pipe-heavy examples unless you want):
    Use grep to narrow the output
    Then process that output further

# Shell Scripting:

    Refer to notes in
        C:\Webdevpro\Devops Roadmap\Lessons\Linux\Lesson7

# AWS Volumes (EBS with EC2)

        This is used to manage storage for an EC2 instance in AWS.

        You can add additional EBS volumes from the AWS Console.

        ⚠️ Notes on device names:

        In the AWS console, device names look like: /dev/sdf, /dev/sdg, /dev/sdh

        Inside Linux, they usually appear as:

        /dev/xvdf, /dev/xvdg, /dev/xvdh

        On newer Nitro-based instances, they may appear as:

        /dev/nvme1n1, /dev/nvme2n1, etc.
        (Always confirm using lsblk)

        The root Ubuntu EC2 already comes with a minimum 8 GB root volume, usually mounted at /.

        Two Important Concepts

        Volume attachment

        Attaching an EBS volume to an EC2 instance

        Done from the AWS Console

        OS cannot use the disk until it is mounted

        Volume mounting

        Making the attached disk usable by Linux

        Requires formatting and mounting

        Useful Commands
        lsblk


        Shows all block devices (disks, partitions, LVM)

        df -h


        Shows mounted filesystems and disk usage

        Disk Management in Linux

        Disk management is commonly handled using Logical Volume Manager (LVM).

        LVM provides:

        Disk aggregation

        Flexible resizing

        Logical abstraction over physical disks

        Example Setup

        Additional disks attached:

        10 GB → /dev/sdf → /dev/xvdf

        12 GB → /dev/sdg → /dev/xvdg

        15 GB → /dev/sdh → /dev/xvdh

        We will follow two paths:

        PATH 1: Using LVM (Recommended for flexibility)
        Goal

        Combine xvdf (10GB) + xvdg (12GB) → Volume Group (22GB)

        Create a 10GB logical volume

        Keep xvdh (15GB) for direct mounting later

        Step-by-Step (LVM)
        1. Create Physical Volumes (PV)
        pvcreate /dev/xvdf /dev/xvdg


        Check:

        pvs

        2. Create a Volume Group (VG)
        vgcreate vg_data /dev/xvdf /dev/xvdg


        Check:

        vgs

        3. Create a Logical Volume (LV)

        Create a 10GB logical volume from the 22GB VG:

        lvcreate -L 10G -n lv_appdata vg_data


        Check:

        lvs

        4. Format the Logical Volume
        mkfs.ext4 /dev/vg_data/lv_appdata

        5. Create a Mount Directory
        mkdir /data

        6. Mount the Logical Volume
        mount /dev/vg_data/lv_appdata /data


        Verify:

        df -h

        7. Persistent Mount (After Reboot)

        Edit /etc/fstab:

        blkid


        Copy the UUID and add:

        UUID=<uuid-value>  /data  ext4  defaults  0  2

        PATH 2: Direct Disk Mounting (Without LVM)

        Using the 15GB disk /dev/xvdh

        Step-by-Step (Direct Mount)
        1. Format the Disk
        mkfs.ext4 /dev/xvdh

        2. Create Mount Directory
        mkdir /backup

        3. Mount the Disk
        mount /dev/xvdh /backup


        Verify:

        df -h

        4. Persistent Mount
        blkid /dev/xvdh


        Add to /etc/fstab:

        UUID=<uuid-value>  /backup  ext4  defaults  0  2

        Summary

        Attachment → AWS Console

        Mounting → Linux OS

        lsblk → See disks

        df -h → See mounted usage

        LVM → Flexible, scalable storage

        Direct mount → Simple, fast, no abstraction


    DYNAMICALLY EXTEND THE SPACE OF LV:

            VG: vg_data

            LV: lv_appdata

            Mounted at: /data

            Filesystem: ext4

            You already have free space in the VG and you want to extend the logical volume by 5 GB dynamically (no reboot, no unmount).

            Goal

            Increase:

            /dev/vg_data/lv_appdata


            by +5 GB, and make /data immediately usable with the new space.

            STEP 0 — Verify current state (always do this)
            lvs
            vgs
            df -h /data


            You should confirm:

            VG has ≥ 5 GB free

            LV is mounted

            Filesystem is ext4

            STEP 1 — Extend the Logical Volume (LVM layer)
            Command
            lvextend -L +5G /dev/vg_data/lv_appdata

            What this does (internals)

            LVM takes 5 GB from the free pool in vg_data

            Appends it to lv_appdata

            Filesystem is NOT resized yet

            At this point:

            LV size is bigger

            Filesystem still thinks old size

            Verify:

            lvs

            STEP 2 — Resize the filesystem (filesystem layer)

            Because this is ext4, and it is mounted, we can resize it live.

            Command
            resize2fs /dev/vg_data/lv_appdata

            What this does

            Reads the new LV size

            Expands the ext4 filesystem to fill it

            No downtime

            No unmount

            No reboot

            STEP 3 — Verify final result
            df -h /data


            You should now see +5 GB added.

            ONE-LINE PRODUCTION COMMAND (very important)

            In real systems, admins usually do this:

            lvextend -L +5G -r /dev/vg_data/lv_appdata

            What -r does

            -r = resize filesystem

            Internally runs:

            lvextend

            then resize2fs (for ext4)

            So:

            LV resize + filesystem resize in one safe operation

            VERY IMPORTANT SAFETY NOTES
            1️⃣ This works live because:

            Filesystem = ext4

            Kernel supports online resize

            LV is not corrupted

            2️⃣ If filesystem were xfs

            You would do:

            lvextend -L +5G /dev/vg_data/lv_appdata
            xfs_growfs /data


            ⚠️ resize2fs does NOT work for xfs.

            Final mental model (memorise)
            Extend VG space → Extend LV → Extend filesystem


            Or in layers:

            VG (storage pool)
            ↓
            LV (block device grows)
            ↓
            Filesystem (must be told to grow)

            Interview-grade answer

            “To extend a logical volume by 5GB dynamically, I use lvextend -L +5G and then resize the filesystem using resize2fs, or use lvextend -r for a single-step resize.”


    DYNAMICALLY REDUCE THE SPACE OF LV:

            🔴 First, the most important rule (memorise this)

            Extending an LV can be done online.
            Shrinking an LV is ALWAYS risky and must be done OFFLINE.

            Why?

            Filesystems can grow safely

            Shrinking risks data loss if done in the wrong order

            🧠 Current state (your system right now)

            From your output:

            LV name: /dev/one_vg/one_lv

            Current size: 15 GB

            Filesystem: ext4

            Mounted at: /mnt/lv_mount_point

            Goal: shrink back to 10 GB

            ❌ WRONG WAY (never do this)
            lvreduce -L 10G /dev/one_vg/one_lv


            ❌ This would:

            Cut the block device first

            Filesystem still thinks it’s 15 GB

            Result: filesystem corruption

            ✅ CORRECT WAY (safe, ordered, production-grade)

            Shrinking must always follow this exact order:

            1. Unmount filesystem
            2. Check filesystem
            3. Shrink filesystem
            4. Shrink logical volume
            5. Mount again


            Think of it as:

            Shrink the filesystem first, then shrink the container.

            STEP-BY-STEP (nested explanation)
            🔹 STEP 1: Unmount the filesystem (MANDATORY)

            You cannot shrink ext4 while mounted.

            umount /mnt/lv_mount_point


            Verify:

            mount | grep one_lv


            (no output = unmounted)

            🔹 STEP 2: Filesystem consistency check (REQUIRED)

            Before shrinking, ext4 must be clean.

            e2fsck -f /dev/one_vg/one_lv


            Why this matters:

            Finds and fixes inode inconsistencies

            Prevents shrinking a corrupted FS

            This step is not optional

            🔹 STEP 3: Shrink the filesystem (CRITICAL STEP)

            Now you tell ext4 to shrink to 10 GB:

            resize2fs /dev/one_vg/one_lv 10G


            What happens internally:

            ext4 moves data blocks inward

            Frees space at the end

            Updates metadata safely

            ⚠️ If you choose a size smaller than used data, this will FAIL (good safety).

            🔹 STEP 4: Shrink the logical volume

            Now that the filesystem is smaller, we shrink the LV to match.

            lvreduce -L 10G /dev/one_vg/one_lv


            You will see a warning like:

            WARNING: Reducing logical volume to 10.00 GiB
            Do you really want to reduce? [y/n]


            Type:

            y


            This is safe because the filesystem is already smaller.

            🔹 STEP 5: Mount again
            mount /dev/one_vg/one_lv /mnt/lv_mount_point


            Verify:

            df -h /mnt/lv_mount_point


            You should now see:

            ~10G total size

            🔐 OPTIONAL but STRONGLY RECOMMENDED safety step

            Run another filesystem check after shrinking:

            e2fsck -f /dev/one_vg/one_lv


            This confirms integrity.




















