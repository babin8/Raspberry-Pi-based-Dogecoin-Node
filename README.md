# Raspberry-Pi-based-Dogecoin-Node

Objective:
To build a dogecoin node that is connected to the blockchain 24/7.

The full conversation with Grok can be found at:
https://grok.com/share/c2hhcmQtMg%3D%3D_907a23df-b235-43f8-85aa-f92374a57789

===================================================================================================================================================================
Do Only Good Everyday
===================================================================================================================================================================

Hardware used: 
   
    - Raspberry-pi 4B 2019, 4GB
    
    - Transcend 512GB External SSD

Step 1: Setup The Raspberry Pi

    Download the [Raspberry Pi Imager](https://www.raspberrypi.com/software/) and write  Raspberry Pi OS (64-bit) to the SSD. 
    Connect a keyboard and a mouse. 
    Configure the OS.

    Update the Raspberry Pi
    (Open the Terminal)
        sudo apt update
        sudo apt full-upgrade -y

Step 2: Install Dogecoin Core

    Download the latest [Dogecoin Core](https://github.com/dogecoin/dogecoin/releases) for ARM Linux. (1.14.9 at the time of setup)

        #unpack the downloaded file
        tar -xvf dogecoin-1.14.9-arm-linux-gnueabihf.tar.gz

        #move it to a handy spot, like /home
        sudo mv dogecoin-1.14.9 /home/dogecoin

        #Make the files executable
        sudo chmod +x /opt/dogecoin/bin/*

Step 3: Setup the Configuration

        #Create a folder for Dogecoin data
        mkdir ~/.dogecoin

        #Create a config file
        nano ~/.dogecoin/dogecoin.conf

        #Add the following lines (replace yourusername and yourstrongpassword with your own choices)
        rpcuser=yourusername
        rpcpassword=yourstrongpassword
        server=1
        txindex=1

        (Press ctrl + x, then y, then Enter to save and exit)\

Step 4: Fix compatibility for Dogecoin Core, which is a 32-bit software.

        #Enable Multiarch Support
        sudo dpkg --add-architecture armhf
        sudo apt update

        #Install 32-bit Compatibility Libraries
        sudo apt install libc6:armhf

        #Verify the Dynamic Linker
        ls /lib/ld-linux-armhf.so.3

        (If the file exists, your system should now be capable of running 32-bit ARM binaries.)

Step 5: Start the Dogecoin Node

        #run the dogecoind file
        /home/dogecoin/bin/dogecoind -daemon

        #get detailed information about the blockchain sync status
        /home/dogecoin/bin/dogecoin-cli getblockchaininfo

Step 6: Setup for autolaunch at startup

        #create a new service file
        sudo nano /etc/systemd/system/dogecoind.service

        #Add the content to the file (Replace User=dogecoin with User=yourusername)
        [Unit]
        Description=Dogecoin Core Daemon
        After=network.target

        [Service]
        Type=simple
        ExecStart=/home/dogecoin/bin/dogecoind
        User=dogecoin
        Restart=always

        [Install]
        WantedBy=multi-user.target

        (Ctrl + X, then Y, then Enter to save and exit)

        #Reload the systemd daemon to recognize the new service
        sudo systemctl daemon-reload

        #Enable the service to start automatically at boot
        sudo systemctl enable dogecoind.service

        #Start the service manually to test it
        sudo systemctl start dogecoind.service

        #Check the status to confirm it's running
        sudo systemctl status dogecoind.service

        (Look for "active (running)" in the output)

Step 7: Final Confirmations

  Restart the Raspberry-Pi, open the terminal and check the node's sync progress

        #sync progress
        /home/dogecoin/bin/dogecoin-cli getblockchaininfo

  Here’s what to look for in the output:

  - If blocks is less than headers, your node is still syncing. The difference between these two numbers shows how many blocks remain to be downloaded and verified.
  - If blocks equals headers, your node has caught up with the blockchain.
  - If verificationprogress is less than 1, the sync is ongoing. When it’s very close to 1 (e.g., 0.999999), the sync is nearly done.
  - If initialblockdownload is true, your node is still performing its initial sync. When it switches to false, the sync is complete.
