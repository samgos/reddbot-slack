# Reddbot - Slack Reddcoin Tipbot.

#### Reddcoin crypto currency tipbot for [Slack](https://slack.com)

## Setup

We're using [digitalocean.com](https://digitalocean.com) so these instructions will be specific to that plaform.

### Create and configure droplet

#### Create droplet 

* Go to digitalocean.com and create a new droplet
  * hostname: reddbot
  * Size
    * Pick either w/ 2GB/2CPUs $20 a month or w/ 1GB/1CPUs $10 a month.
  * Region
    * Amsterdam or whatever location is closest to you.
  * One-click Apps
    * Dokku 0.94 on 16.04
  * Add SSH keys
    * Digital Ocean provide a easy-following tutorial for this that can be accessed once you click on add SSH key’s option, you’ll see a html link above the ssh key input console for creating and using SSH keys. 

#### Configure hostname

* Copy/Paste IP address into URL bar after the droplet has installed which might take 5/10 minutes, to be faced with the Dokku setup.
  * Put in `hostname` for whatever domain you are using. 
    * `example.com`
  * Check `Use virtualhost naming for apps
    * `http://<app-name>.example.com`
  * Finish Setup

#### Add DNS

* You'll need a domain for this. For this documentation I'm using `example.com`
* Point your domain's nameservers to digitalocean
  * `ns1.digitalocean.com`
  * `ns2.digitalocean.com`
  * `ns3.digitalocean.com`
* In digitalocean's `DNS` section set an `A-Record` for your `hostname` from your previous step
  * Make the `hostname` be the name of your app
    * `reddbot`
  * Make the IP address be the one provided by digitalocean for your droplet.
* After the DNS propogates which may take 10/20 minutes.
  * In the `Zone file` of the DNS section of digital ocean you'll see:
    * `reddbot IN A 143.143.243.143`
 * Go to a console/terminal on your Desktop and enter the following to command to see if the domain is live and connected to your droplet, you should see a response with your droplet's IP. 
 * `ping reddbot.example.com`
    * `PING reddbot.example.com (143.143.243.143): 56 data bytes`

#### SSH into your new virualized box

* `ssh -o "IdentitiesOnly yes" -i ~/location/id_rsa root@droplet-ip` or `ssh root@droplet-ip`
  * If you correctly added your SSH keys you'll get signed in

#### Warning 
* To allow port open on 4567 and 45543 the firewall needs to be disabled, will update with port-forwarding method ASAP. If you can do this yourself great, if not wait or continue at your own risk. 
* use command `sudo ufw disable`




#### Compile reddcoind

* Download the source code
  * `git clone https://github.com/reddcoin-project/reddcoin`

* Install Dependencies 
* ` wget https://launchpad.net/ubuntu/+archive/primary/+files/miniupnpc_1.9.20140610.orig.tar.gz` 
 * `tar xzvf miniupnpc_1.9.20140610.orig.tar.gz`
  * `cd miniupnpc-1.9.20140610`
   * `make && make install`
      * `cd`
	* `sudo apt-get -y install build-essential`
	* `sudo apt-get -y install libtool autotools-dev autoconf`
	* `sudo apt-get -y install libssl-dev`
	* `sudo add-apt-repository ppa:bitcoin/bitcoin`
	* `sudo apt-get update`
	* `sudo apt-get -y install libdb4.8`
	* `sudo apt-get -y install libdb4.8-dev`
	* `sudo apt-get -y install libdb4.8++-dev`
	* `sudo apt-get -y install libminiupnpc-dev`
	* `sudo apt-get -y install libqt4-dev libprotobuf-dev protobuf-compiler`
	* `sudo apt-get -y install libqrencode-dev`
	* `sudo apt-get -y install libqt5gui5`
	* `sudo apt-get -y install qttools5-dev`
	* `sudo apt-get -y install qttools5-dev-tools`
	* `sudo apt-get -y install libprotobuf-dev`
	* `sudo apt-get update && sudo apt-get -y install pkg-config`

	

* Note if you are using a 10$ Droplet with 1GB ram you will need accommodate for more memory usage.
	* `free`
	* `dd if=/dev/zero of=/var/swap.img bs=1024k count=1000`
	* `mkswap /var/swap.img`
	* `swapon /var/swap.img`
	* Confirm that you actually did free some memory by comparing it to the first output of `free`

* Compile reddcoind
	* `cd reddcoin`
	* `./autogen.sh`
	
* Build Berkeley DB 4.8
	* `BITCOIN_ROOT=$(pwd)`

* Pick some path to install BDB to, here we create a directory within the reddcoin directory
	* `BDB_PREFIX="${BITCOIN_ROOT}/db4"`
	* `mkdir -p $BDB_PREFIX`

* Fetch the source and verify that it is not tampered with
	* `wget 'http://download.oracle.com/berkeley-db/db-4.8.30.NC.tar.gz'`
	* `echo '12edc0df75bf9abd7f82f821795bcee50f42cb2e5f76a6a281b85732798364ef  db-4.8.30.NC.tar.gz' | sha256sum -c`
* Output should be: `db-4.8.30.NC.tar.gz: OK`
	* `tar -xzvf db-4.8.30.NC.tar.gz`

* Build the library and install to our prefix
	* `cd db-4.8.30.NC/build_unix/`

* Note: Do a static build so that it can be embedded into the exectuable, instead of having to find a .so at runtime
	* `../dist/configure --enable-cxx --disable-shared --with-pic --prefix=$BDB_PREFIX`
	* `make install`

* Configure Reddcoin Core to use our own-built instance of BDB	
	* `cd $BITCOIN_ROOT`
	* `sudo apt-get -y install libboost-all-dev`
	* `./configure LDFLAGS="-L${BDB_PREFIX}/lib/" CPPFLAGS="-I${BDB_PREFIX}/include/"`
	* `make` - This will take some time, let it do it’s thing. 
	* `make install`
	* `cd ` 
	* `cd /usr/local/bin`
	* `strip reddcoind`

* Add a user and move reddcoind
  *  Ubuntu has a password error when attempting to switch back to root from your new user so make sure to re-enter or make a new
  password for sudo `sudo passwd root`
	* `adduser reddcoin && usermod -g users reddcoin && usermod -aG sudo reddcoin && delgroup reddcoin && chmod 0701 /home/reddcoin`
	* `mkdir /home/reddcoin/bin`
	* `cp ~/reddcoin/src/reddcoind /home/reddcoin/bin/reddcoin`
	* `chown -R reddcoin:users /home/reddcoin/bin`
	* `cd && rm -rf reddcoin`

* Run the daemon
 * `su reddcoin`
 * `cd && bin/reddcoin`    
  * On the first run, reddcoin will return an error and tell you to make a configuration file, named reddcoin.conf, in order to add a username and password to the file.
	 * `nano ~/.reddcoin/reddcoin.conf && chmod 0600 ~/.reddcoin/reddcoin.conf`
* Add the following to your config file, changing the username and password to something secure. Make sure to take note of the `rpcuser` and `rpcpassword` because you'll need them in a couple of steps
	   
* daemon = 1
* rpcuser=reddrpc
* rpcpassword=Z01BBDFKF
* rpcthreads=300
* rpcallowip=droplet_ip
* listen=1
* txindex=1

* To exit nano editor use Ctrl + X + Enter to save.

  * Run the daemon again
    * `cd && bin/reddcoin` 
  * To confirm that the daemon is running
    * `cd && bin/reddcoin getinfo`
  * Add the bootstrap to speed up syncing times
    * `cd ~/.reddcoin`
    * `wget https://github.com/reddcoin-project/reddcoin/releases/download/v2.0.0.0/bootstrap.dat.xz`
    * `unxz bootstrap.dat.xz`



## Let the blockchain sync which may take up to around 10-15 hours. (You can still progress but balances won't update until synced)




#### Clone the Reddbot Bot git repo

* `cd`
* `git clone https://github.com/samgos/reddbot`
* `cd reddbot`

* Install Ruby 2.4.0 and rvm
 * `sudo apt-get install libgdbm-dev libncurses5-dev automake libtool bison libffi-dev`
	* `gpg --keyserver hkp://keys.gnupg.net --recv-keys 409B6B1796C275462A1703113804BB82D39DC0E3`
	* `curl -sSL https://get.rvm.io | bash -s stable`
	* `source /home/reddcoin/.rvm/scripts/rvm` 
* `rvm install ruby-2.4.0`
* `rvm use 2.4.0 --default`
* `ruby -v`

* Install bundler
* `gem install bundler`
* `bundle update`
* `sudo apt install ruby-bundler`

* To start using RVM you need to run `source rvm`
* Run `bundle`

* Parsing dependencies. 
* `chmod +x disp.sh`
* `sudo apt-get install html-xml-utils`

* Enter droplet IP in the `bitcoin_client_extensions.rb` file where it says `Host IP` 
* ` nano bitcoin_client_extensions.rb`
* `        { host: 'Host IP', port: 45443, ssl: false} )`
*    Ctrl + X + Enter to save.




* Install node.js and slack-sdk client.
* `sudo apt install -y nodejs-legacy && sudo apt install -y npm && git clone https://github.com/slackapi/node-slack-sdk`
* `cd node-slack-sdk `
* `sudo npm install -g forever`
* `sudo npm install -g @slack/client --save`
* `sudo npm install -g --save-dev capture-console`
* `cd`
* `cd reddbot`





#### Setup node.js client. 
* https://api.slack.com/custom-integrations/legacy-tokens
* Request a legacy token for your slack channel and copy it to the clipboard. 
* Paste this value in the `leaderboard.js` file and `message.js` file in the following positon where the value `API_KEY` is.
* `var token = process.env.SLACK_API_TOKEN || 'API_KEY' ; `
* `nano leaderboard.js` -> Ctrl + X + Enter to save.
* `nano message.js` -> Ctrl + X + Enter to save.





#### Set up the Slack integration: as an "outgoing webhook" 

* https://yoursite.slack.com/services/new/outgoing-webhook
* Write down the api token they show you in this page
* Set the trigger word, use `reddbot`
* Set the Url to the server you'll be deploying on http://example.com:4567/tip





#### Launch the server!
* `source rvm`
* `bundle`
* `forever start message.js`
* `forever start leaderboard.js`
* If you want to keep the servering running use can use the unbuntu screen command
* Enter your sudo password you made or re-entered earlier -> `su`
* `screen`
* `su reddcoin`
* `RPC_USER=reddrpc RPC_PASSWORD=your_pass SLACK_API_TOKEN=your_api_key COIN=reddcoin bundle exec ruby tipper.rb -p 4567`
*  Ctrl/Cmd + A + D to detach from the screen. 

### Emoji setup
There is three emoji's included , `reddbank` , `reddcoin` and `bitcoin` 
Setup each of them here -> https://yoursite.slack.com/customize/emoji
* `bitcoin` -> https://ibb.co/dRkSbk
* `reddbank` -> https://ibb.co/eJpaU5
* `reddcoin` -> https://ibb.co/cNFFU5



### Example
![Alt text](https://image.ibb.co/bKUdGk/Screen_Shot_2017_08_22_at_04_56_13.png "Preview")




### Usage
Note: Tip over 10 times before initating the leaderboard command. 

say any of the reddbot commands for example `reddbot hi` , `reddbot tip @username 100` , `reddbot deposit` use the command `reddbot help` to find out more.

## Security

This wallet is unecrypted please go to extra efforts to secure the system from attacks, aka encrypting the reddcoin core wallet and SSL encryption improvement. 

## Credits
Thanks to one of the reddcoin community members making the reddbank gif.

Using the [Reddcoin Core v2.0](https://github.com/reddcoin-project/reddcoin)  library by the Reddcoin Core Developers but also incoperating some of the build instructions to the tutorial. 

Which was then forked from [coin-agnostic tipbot](https://github.com/blocktech/slack_tipbot) by [cgcardona](https://github.com/cgcardona) along with aspects of the tutorial.

This project was originally forked from [dogetip-slack](https://github.com/tenforwardconsulting/dogetip-slack) by [tenforwardconsulting](https://github.com/tenforwardconsulting)

