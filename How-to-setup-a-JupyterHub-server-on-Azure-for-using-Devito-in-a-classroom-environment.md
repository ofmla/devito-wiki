The purpose of this workflow is to set up a JupyterHub server on Azure for using Devito in a classroom environment.
In this setup, we prepare a server where each user logs-in with GitHub credentials and has a PWD where Devito is installed. This solution delivers access to shared resources.

A lot of resources and how-to tutorials on how to set up a JupyterHub for your application on Azure can be found out there.

Our suggestion is:

## Organizer/Educator workflow

Step 1: Setup TLJH on our VM.
We follow the link here to set up our VM:
http://tljh.jupyter.org/en/latest/install/azure.html
Setup includes: Installing The Littlest JupyterHub¶, adding users, install conda / pip packages for all users

What we installed for the full Devito experience:

sudo apt-get install texlive-base (notebooks rendering)

sudo apt-get install mpich libmpich-dev (for those brave for MPI)




Step 2: Add user authentication (via Github)
In order to add GitHub authentication for the new users we follow the approach presented here:
http://tljh.jupyter.org/en/latest/howto/auth/github.html

Step 3: Add the init-setup script for each user to the end of `/etc/skel/.bashrc` (Install latest devito master)
```
git clone https://github.com/devitocodes/devito.git
cd devito
pip install --user -e .[extras]
```
Also look: http://tljh.jupyter.org/en/latest/howto/content/share-data.html


## Attendee / Student workflow

Each new user connects to the public IP, authenticates with Github credentials, and opens a new terminal from the JupyterHub environment.
```
Open: http://<PublicIP> and authenticate using GIT credentials
New -> Terminal
```

When the terminal is open, the script from /etc/environment is triggered and installs latest devito master on user PWD. User can then enjoy the notebooks.




Keep in mind:
- We have seen company-firewall protected laptops not being able to pop-up the terminal window.
- User install with `pip install --user ..pkg-name..`