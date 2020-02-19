Purpose of this workflow is to setup
You can find a lot of resources and how-to tutorials on how to setup a JupyterHub for your application on Azure.

Our suggestion is:

Step 1:
We follow the link here to setup our VM:
http://tljh.jupyter.org/en/latest/install/azure.html

Step 2:
In order to add GitHub authentication for the new users we follow the approach presented here:
http://tljh.jupyter.org/en/latest/howto/auth/github.html

Step 3:
Each new user connects to the public IP and opens a new terminal from the JupyterHub environment.
Then we follow:
```
Open: http://<PublicIP> and authenticate using GIT credentials
New -> Terminal
git clone https://github.com/devitocodes/devito.git
cd devito
conda env create -f environment-dev.yml
source activate devito
pip install -e .
pip install --user matplotlib
```
