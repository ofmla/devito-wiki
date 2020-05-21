Purpose of this workflow is to setup a JupyterHub server on Azure for using Devito in a classroom environment.
In this setup, each user logs-in with GitHub credentials and has a PWD where Devito can be installed. This solution delivers access to shared resources.

A lot of resources and how-to tutorials on how to setup a JupyterHub for your application on Azure.

Our suggestion is:

## Organizer/Educator workflow

Step 1: Setup TLJH on our VM.
We follow the link here to setup our VM:
http://tljh.jupyter.org/en/latest/install/azure.html
Setup includes: Installing The Littlest JupyterHub¶, adding users, install conda / pip packages for all users

Step 2: Add user authentication (via Github)
In order to add GitHub authentication for the new users we follow the approach presented here:
http://tljh.jupyter.org/en/latest/howto/auth/github.html

Step 3: Add the setup script for each user to `/etc/environment`
```
git clone https://github.com/devitocodes/devito.git
cd devito
pip install --user -e .[extras]
```

Step 3: Setup PWD.

## Attendee / Student workflow

Each new user connects to the public IP and opens a new terminal from the JupyterHub environment.
```
Open: http://<PublicIP> and authenticate using GIT credentials
New -> Terminal
```

When the terminal is open, the script from /etc/environment is triggered and installs latest devito master on user PWD. User can then enjoy the notebooks.

