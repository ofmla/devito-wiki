You can find a lot of resources and how-to tutorials on how to setup a Jupyterhub for you application on Azure.

Our suggestion is:

Step 1:
We follow the link here:
http://tljh.jupyter.org/en/latest/install/azure.html

Step 2:
In order to add GitHub authentication we follow that:
http://tljh.jupyter.org/en/latest/howto/auth/github.html

Step 3:
Each user connects to the public IP and opens a new terminal:
Then we follow:
```
Open: http://52.191.134.114
Authenticate using GIT credentials
New-> Terminal
pip install --user git+https://github.com/devitocodes/devito.git 
git clone https://github.com/devitocodes/devito.git
cd devito
conda env create -f environment-dev.yml
source activate devito
pip install -e .
pip install --user matplotlib
```
