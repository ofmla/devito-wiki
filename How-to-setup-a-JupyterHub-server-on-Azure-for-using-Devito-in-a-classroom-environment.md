The purpose of this workflow is to set up a JupyterHub server on Azure for using Devito in a classroom environment.
In this setup, we prepare a server where each user logs-in with GitHub credentials and has a PWD where Devito is installed. This solution delivers access to shared resources.

A lot of resources and how-to tutorials on how to set up a JupyterHub for your application on Azure can be found out there.

What we do:

## Organizer/Educator workflow

**Step 1:** Setup TLJH on our VM.
We follow the link here to set up our VM:
http://tljh.jupyter.org/en/latest/install/azure.html
Setup includes: Installing The Littlest JupyterHub¶, adding users, install conda / pip packages for all users

- Note here: Depending on the number of users expected to connect we may need around 560M of disk size per user. So it is recommended that we increase disk size. (e.g. for around 20 users, 11G were occupied)

What we installed for the full Devito experience:
```
sudo apt-get install texlive-full # (notebooks rendering)
sudo apt-get install mpich libmpich-dev # (for those brave for MPI)
```
All the rest were already in the JupyterHub


**Step 2:** Add user authentication (via Github)

In order to add GitHub authentication for the new users we follow the approach presented here:
http://tljh.jupyter.org/en/latest/howto/auth/github.html

(To reset authentication follow: http://tljh.jupyter.org/en/latest/howto/auth/firstuse.html#howto-auth-firstuse)

**Step 3:** Add the init-setup script for each user to the end of `/etc/skel/.bashrc`. 

```
# e.g. for Transform 2020
git clone https://github.com/devitocodes/devito.git # Clone latest Devito master
git clone https://github.com/devitocodes/transform2020.git # Clone Devito tutorial
cd devito && pip install --user -e . # Install Devito
pip install --user matplotlib # Matplotlib is needed for tutorials
```

Also have look: http://tljh.jupyter.org/en/latest/howto/content/share-data.html if you want to add
a priori any data files.

**Step 4:** SSL encryption / Enbale HTTPS
You must have a domain name set up to point to the IP address on which TLJH is accessible before you can set up HTTPS. (You can do this via the Azure portal)

To enable HTTPS via letsencrypt:
```
sudo tljh-config set https.enabled true
sudo tljh-config set https.letsencrypt.email you@example.com
sudo tljh-config add-item https.letsencrypt.domains yourhub.yourdomain.edu
```
To install a certificate via certbot:
```
# Install let's encrypt certbot
sudo snap install --classic certbot
sudo ln -s /snap/bin/certbot /usr/bin/certbot

# Open ports
sudo ufw allow 80
sudo ufw allow 443

# (optional) Check which ports are listening
sudo apt install net-tools
netstat -ln
# Stop services to free ports
systemctl stop jupyterhub.service
systemctl stop traefik.service
# Begin certification (Follow instructions, easy TO ADD details)
sudo certbot certonly --standalone
# Re-enable Jupyterhub
systemctl start traefik.service
systemctl start jupyterhub.service
```
Now you can SECURELY access the server at https://yourhub.yourdomain.edu

## Attendee / Student workflow

Each new user connects to the public IP, authenticates with Github credentials, and opens a new terminal from the JupyterHub environment.
```
Open: http://<PublicIP> and authenticate using GIT credentials
New -> Terminal
```

When the terminal is open, the script from /etc/skel/.bashrc is triggered and installs latest Devito master on user PWD. User can then enjoy the notebooks.


Keep in mind:
- We have seen company-firewall protected laptops not being able to pop-up the terminal window.
- User install with `pip install --user ..pkg-name..`