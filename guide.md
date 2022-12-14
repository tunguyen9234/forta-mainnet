# Install and Configure Docker
- Install Docker from the link https://docs.docker.com/engine/install/, then check docker version must be 20.10 or later
````
wget -O get-docker.sh https://get.docker.com 
sudo sh get-docker.sh
rm -f get-docker.sh
docker --version
````        
- Add a file called daemon.json to your /etc/docker directory with the following contents:
````
 vi /etc/docker/daemon.json
````  
copy and pase below 
````
{
"default-address-pools": [
{
"base":"172.17.0.0/12",
"size":16
},
{
"base":"192.168.0.0/16",
"size":20
},
{
"base":"10.99.0.0/16",
"size":24
}
]
}
````        
- Restart docker 
````        
systemctl restart docker
````       
# Install Forta
- Install via APT (Used for Ubuntu, Debian etc.) - i use this
````   
sudo curl https://dist.forta.network/pgp.public -o /usr/share/keyrings/forta-keyring.asc -s
echo 'deb [signed-by=/usr/share/keyrings/forta-keyring.asc] https://dist.forta.network/repositories/apt stable main' | sudo tee -a /etc/apt/sources.list.d/forta.list
sudo apt-get update
sudo apt-get install forta
````    
# Initial Setup
- Initialize Forta Directory
````        
 forta init --passphrase **YOUR_PASSWORD**
````        
 Your result will be as below
        
 > Scanner address: 0x8edxxxx....xxxxx37D7F       <<<<< Write down your scan address
 Successfully initialized at /root/.forta
 Please make sure that all of the values in config.yml are set correctly.
 Please fund your scanner address with some MATIC.
 Please enable it for the chain ID in your config by doing 'forta register --owner-address <your_owner_wallet_address>'
 > 
 - Configure systemd
 ````    
        sudo tee /etc/systemd/system/forta.service > /dev/null <<EOF
        [Unit]
        Description=Forta Node
        After=network.target
        
        [Service]
        Type=simple
        User=root
        WorkingDirectory=/root/
        ExecStart=$(which forta) run --passphrase=**YOUR_PASSWORD**
        Restart=on-failure
        RestartSec=3
        LimitNOFILE=10000
        
        [Install]
        WantedBy=multi-user.target
        EOF
   ````     
 - Override Forta systemd service environment  
  ````     
  mkdir /etc/systemd/system/forta.service.d && touch /etc/systemd/system/forta.service.d/env.conf
  vi /etc/systemd/system/forta.service.d/env.conf
  ````      
 - Copy below text into the file
 ````       
        [Service]
        Environment="FORTA_DIR=/root/.forta"
        Environment="FORTA_PASSPHRASE=**YOUR_PASSWORD**"
 ````       
              Press Esc and :wq to save & quit Vi editor.
        
 - Reload and enable Forta system service
 ````       
 sudo systemctl daemon-reload
 sudo systemctl enable forta
 ````       
 # Create app on Alchemy and get API
        
 Login the link https://dashboard.alchemyapi.io/apps, then select Apps ⇒ Create App.Then select Chain is Polygon, Network is Polygon Mainnet , then press Create App
 After your App is created, press View Key, and write down your jsonRPC in HTTP, example https://polygon-mainnet.g.alchemy.com/v2/******

 # Configure config.yml
  ````      
        cd /root/.forta/ && cp config.yml config.yml-bak
        sed -i '3,3 s/\(chainId: \).*/\1137/g' config.yml
        API_KEY=https://polygon-mainnet.g.alchemy.com/v2/*********
        sed -i '8,8 s|\(.url: \).*|\1'$API_KEY'|g' config.yml
        sed -i '12 s/jsonRpc:/enabled: false/g' config.yml
        sed -i '13d' config.yml
   ````     
 # Send 1 $Matic from your Metamask Address to Forta your scan node address.
        
 - Show your Forta scan node Address
 ````       
 forta account address
````    
- Send 1 $MATIC to the scan node address
    
- Register Scan Node
    
    Run below command, YOUR_METAMASK_ADDRESS is different with scan node address, 
 ````  
    forta register --owner-address YOUR_METAMASK_ADDRESS --passphrase YOUR_PASSPHRASE
 ````   
# Run Forta and check status
-  Run Forta
````
sudo systemctl enable forta
sudo systemctl start fortad.service
````        
- Check service
````
journalctl -u forta.service -f -o cat
forta status
forta status --show all --format oneline
````
# delete node
````
systemctl stop forta && systemctl disable forta.service && rm -rf $HOME/.forta/
rm -rf config.yml
````
       
