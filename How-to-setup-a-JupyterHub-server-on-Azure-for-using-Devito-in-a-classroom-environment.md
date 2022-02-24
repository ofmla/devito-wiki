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

If after Github authentication, it is needed to add a new user as admin:
```
sudo tljh-config add-item users.admin <username>
```

**[DEPRECATED- Follow next section]Step 3:** Add the init-setup script for each user to the end of `/etc/skel/.bashrc`. 

```
# e.g. for Transform 2020
git clone https://github.com/devitocodes/devito.git # Clone latest Devito master
git clone https://github.com/devitocodes/transform2020.git # Clone Devito tutorial
cd devito && pip install --user -e . # Install Devito
pip install --user matplotlib # Matplotlib is needed for tutorials
```

Also have look: http://tljh.jupyter.org/en/latest/howto/content/share-data.html if you want to add
a priori any data files.

**[UPDATED-rice2022]Step 3:** In this step we create a bash script that setups a global/system-wide/for all users devito installation and copy the examples folder to each user's $PWD.

```bash
#!/bin/bash
#

# Install the python depedencies (pip)
source /opt/tljh/user/bin/activate
pip3 install -U pip

# Print python3 --version
python3 -c 'import sys; print(".".join(map(str, sys.version_info[:3])))'

# Save current dir
dir=$PWD

# Install Devito in home directory
# python3 -m pip install --ignore-installed --no-cache-dir devito # bit redundant setup (useful if other devito may have been installed)
pip install devito # (should be fine in a clean tljh install)

# Clone Devito, not to install (as installed in last step), but only to copy examples
cd $HOME
git clone https://github.com/devitocodes/devito
cd devito

# Make examples available or other tutos you may like
cp -r examples/seismic/tutorials /etc/skel/devito_rice_2022

# Deactivate tljh env
source /opt/tljh/user/bin/deactivate
# Return to $dir
cd $dir
```

Also, have look: http://tljh.jupyter.org/en/latest/howto/content/share-data.html if you want to add
a priori any data files.

**Step 4:** SSL encryption / Enable HTTPS
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

When the terminal is open, the script from /etc/skel/.bashrc is triggered and installs the latest Devito master on user PWD. Users can then enjoy the notebooks.


Keep in mind:
- We have seen company-firewall protected laptops not being able to pop up the terminal window.
- User install with `pip install --user ..pkg-name..`

Bug references:
- https://github.com/jupyterhub/the-littlest-jupyterhub/issues/115#issuecomment-506910089