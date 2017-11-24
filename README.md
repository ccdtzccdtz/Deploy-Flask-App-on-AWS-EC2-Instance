
# Deploy Flask App to AWS EC2

## Connect to Server

### 1. Sign up for a AWS account and launch an EC2 instance

1. Select the AMI (Amazon Machine Image) that satisfy your need. I am using __Ubuntu Server 16.04 LTS__ (HVM) <Br>
2. Select instance type. I am using __t2.micro__ which is eligible for free tier. Edit configuration if you need custom setting <Br>
3. Choose to create a new key pair or use exiting key pair.If you choose to use existing key pair, make sure you have access to that __.pem__ file. <Br>
4. Launch! <Br>

![alt text](https://github.com/ccdtzccdtz/Deploy-Flask-App-on-AWS-EC2-Instance/blob/master/img/Screenshot%20from%202017-11-23%2018-11-22.png?raw=true)

## Server Setup

### 1. Follow AWS's instructions on how to connect to the instance

To access your instance:

1. Open an SSH client. (find out how to connect using PuTTY) 
2. Locate your private key file (yourpermfilename.pem). The wizard automatically detects the key you used to launch the instance. 
3. Your key must not be publicly viewable for SSH to work. Use this command if needed: 

 `chmod 400 flask.pem`<br>

4. Connect to your instance using its Public DNS:
__your-long-amazonip.compute.amazonaws.com__

Example:

`ssh -i "yourpermfilename.pem" your-long-amazonip.compute.amazonaws.com`

### 2. Install nginx

```
sudo apt-get update
sudo apt-get install nginx
```

Once nginx is installed, you should be able to go to your public IP or public DNS and see the nginx welcome page. Similar to the example below:


![alt text](https://raw.githubusercontent.com/ccdtzccdtz/Deploy-Flask-App-on-AWS-EC2-Instance/master/img/localhost-8080-nginx.png)

Remove the default page by deleting the default file.

```
sudo rm /etc/nginx/sites-enabled/default
```

Create a new config file in the sites-available folder and create a symbolic link to it in the sites-enabled folder.

```
sudo vim /etc/nginx/sites-available/example.com
```

This is how the config file will look:

```
server {
	listen 80;

	location / {
		proxy_pass http://127.0.0.1:8000/;
	}
}
```

This config file will tell the nginx server to listen on port 80 and pass all requests with the ‘/’ prefix to the server http://127.0.0.1:8000/ We do this because Gunicorn will run your Flask app on port 8000.

Create a symbolic link from the sites-enabled directory that points to the example.com config file we created.

```
sudo ln -s /etc/nginx/sites-available/example.com /etc/nginx/sites-enabled/example.com
```

Restart the nginx web server in order for our changes to take into effect.

```
sudo service nginx restart
```


## Local Setup

### Virtual Environment

I use miniconda to download and manage my libraries and virtual environments. Compares to pip, anaconda makes it much easier to install deep learning dependencies like keras and tensorflow. Downside is your have to install miniconda on the server side as well which will take some time and space.

1. Download miniconda from https://conda.io/miniconda.html
2. Create virtual environment for your specific python version
```
conda create -n myenv python=3.6
```
3. Activate this environment
```
source activate myenv
```
4. Develop your app in this environment or run your existing app in this environment. if there are dependencies need, install them for this environment. Make sure the app is fully functional in this environment.
5. Export dependencies from this environment to a `.yml` file
```
conda env export > environment.yml
```

## Setup a Git Repository on EC2 (Optional)

Here I decide to create a remote repository on EC2. This will keep the repository private and make the file transfer from local workplace to EC2 very smooth. 

1. add EC2 idenity to ssh authentification. This prevents problems with git later, namely getting the error “Permission denied (publickey).”
```
ssh-add path/to/privateEC2key.pem
```
2. create the git repository on the EC2 instance if you are already on EC2
```
mkdir the_project.git cd the_project.git git init --bare
```
3. Back in your local machine. Set up the local repository with your flask projects and all your files.
```
cd the_project 
git init 
git add . 
git commit -m "Initial git commit message" 
git remote add origin username@hostname.com:the_project.git 
git config --global remote.origin.receivepack "git receive-pack" 
git push origin master
```
4. Then you can use git clone the remote repository from everywhere
```
git clone username@hostname.com:the_project.git
```

## App Deployment

On the EC2 server side, you could create a new local repository by git clone or git pull from the remote repository

We have to install miniconda and create an new environment from the `.yml` file. This will install all the dependencies.
```
conda env create -f environment.yml
```
Once you are in your virtual env, you should be able to run your flask app just as you run it in local machine.
In your flask app.py, please the server port to http://127.0.0.1:8000/ if otherwise.

### Notes

After running for a while, if no activity has been detected between your local and the server, your connection might be shut down. In such case, the following error message appears:
```
A line showing packet_write_wait: Connection to XXX : Broken pipe 

```
Solution:

On the host, add those lines in the file `.ssh/config`

```
Host *
  ServerAliveInterval 30
  ServerAliveCountMax 5
```  
If the file config does not exist, just create it.

# Resources

[How to Deploy a Flask App on an AWS EC2 Instance](https://chrisdtran.com/2017/deploy-flask-on-ec2/) <br>
[Setting up a Git repository on an Amazon EC2 instance](https://shirtdev.wordpress.com/2011/05/04/setting-up-a-git-repository-on-an-amazon-ec2-instance/) <br> 
[Managing environments](https://conda.io/docs/user-guide/tasks/manage-environments.html) <br>
[packet_write_wait: Connection to XXX : Broken pipe](http://thomas-cokelaer.info/blog/2017/05/packet_write_wait-connection-to-xxx-broken-pipe/) <br>
