# Deep Learning on AWS

In this article you're going to learn how to setup a Deep Learning Server on AWS so that you can run all of your favorite Neural Network models on the hardware you need. Not only that, I'll also show you how to setup a Jupyter Notebook Server to make your neural network experiments that much easier.

AWS is an excellent alternative to buying your own GPU. Running an Amazon GPU instance costs a fraction compared to new hardware and you won't have to deal with setting your machine up from scratch.

Furthermore, if you're a student in high school or college, you can easily get a bunch of free AWS credits from [AWS Educate](http://www.awseducate.com/)

For this guide we'll use the AMI managed by Github user [Miej](https://github.com/Miej) called [GoDeeper](https://github.com/Miej/GoDeeper). This AMI has a bunch of common deep learning packages ranging from Tensorflow, Keras, Torch and even OpenCV so that you can run all of that cutting-edge research you desire with ease. The repo has more details on what else is installed in the AMI.

Now let's move on to the meat of this problem. If you were to go straight to running p2 instance in Northern Oregon, you'll burn through your money rather quickly. However, there is a better method than simply launching instances. I typically use these things called spot requests. Basically, you bid for server time at a price significantly lower than that of dedicated instances. There's certainly a risk that you'll be outbid while running some program, causing you to be kicked off. However, I've avoided this problem by setting a reasonably high maximum bid that is still cheaper than the price of the dedicated instance.

You'll also want to prioritize the region of your machine based on pricing.
Amazon publishes the [current spot pricing](https://aws.amazon.com/ec2/spot/pricing/) for all machine types in every region. Make note that only a few regions have p2 instances. For me in the US, the two closest regions that fit this criteria are North Virginia and Oregon. I'll typically go to the spot pricing portal and check which region has the cheapest pricing at the moment. However if you live in a different continent, you'll want to look for regions that are close to you that have p2 instances. You can use the spot pricing tool to see whether any region has a p2 instance. If you see an N/A next to the p2 instance, that means this type of machine is not available in that region.

Now let's get on with the tutorial. To get started, you'll need to login to the AWS console. Here you'll want to click Services, then EC2. At the top left you'll want to confirm that you are in a region that has gpu instances. I'll be using the Oregon region because it was the cheapest when I checked the spot pricing. On the left panel, click Spot Requests and then click the Big Blue `Request Spot Instances`.

In the Spot Request wizard, you'll want to go down to the AMI drop down and click Select. You'll be met with a window. Change the dropdown to `Community AMIs`. Now go to the [GoDeeper repo](https://github.com/Miej/GoDeeper) and find the specific ami id for the region you are in. Since I am using Oregon, I will use `ami-da3096ba`.

In the next entry, you'll be selecting the machine type. First, remove default instance type by clicking the `x` in the gray box. Next, click the Select button. In the window that pops up, scroll down to the p2.xlarge row, click it, and press OK.

Everything else should be set as default except Maximum Price if you want to put a limit on how much you'll bid for an instance. I'll typically set the max price to the maximum price of the past week + a few cents of leeway. You can determine this price by clicking on the price history button that appears after you select `Set max price per hour`.

Click Next. You don't really need to worry about the size of the EBS volume here as the AMI already comes with 100GB. So change the volume as you please. Note that larger volumes cost slightly more money. A [pricing schedule is available here](https://aws.amazon.com/ebs/pricing/).

Next you'll want to make a key-pair if you haven't already. Make sure that you save this somewhere you will remember, because you need to use it to ssh into your machine. However, make sure you don't add it to any publicly hosted git repos for privacy purposes. I typically save my keys in the `~/.ssh/` directory. Once you've moved the key you'll want to change the permissions to make the key safe. Simply run
```
sudo chmod ~/.ssh/<key-name> 400
```

Now you'll want to create a security group called Jupyter that has 3 inbound rules-
1. SSH
2. HTTPS
3. Custom TCP Rule - 8888

After you've set those, you'll want to make sure that you change the source to `Anywhere` for each option. Setting the exact source certainly increases the security of your instance, however, this can be problematic if you plan to switch networks or you don't have a static ip on your network. I just leave it as `Anywhere` because I don't use my gpu instances for longer than a few days anyways.

Return to the wizard, refresh the security groups panel and you should see your security group up. Select the checkbox next to it.

Finally, I'll set a timeout limit for the instance. This is just to make sure I don't accidentally leave the instance running for weeks on end, wracking up a bunch of charges on my account. I'll set mine for a week from now because I won't be using this instance for a very long time.

Click Next, then Launch Instance.

Now you'll want to confirm that the instance request has been fulfilled. Click the instances tab in the left panel and on the new page you should see an instance up and running. In the bottom tab you'll see a description. Copy the public dns that you see in right column and navigate to your terminal.

You'll want to run the command
```
ssh -i /path/to/key.pem icarus@<ec2-public-dns>
```
it'll prompt you for the password, which is `changetheworld`
And you should have access!

## Setting up Jupyter
Now here comes the fun part, let's setup a Jupyter notebook for our server. I found this answer in a [Quora post](https://www.quora.com/How-do-I-create-Jupyter-notebook-on-AWS) a while back and have modified it slightly for this tutorial.

The steps here are simply run
```
git clone https://gist.github.com/philkuz/4b7fda8bc2eba4f9a1ba71c54321c126 nb
. nb/jupyter_notebook_ec2.sh
```
From this you'll be prompted to enter a password. Then you'll be given a series of questions about the certs. I just leave them as default as they are not important for our purposes.

Now run these commands
```
cd;mkdir notebook;cd notebook
tmux new -s nb
jupyter notebook --certfile=~/certs/mycert.pem --keyfile ~/certs/mycert.key
```
*Note: you can use screen instead of tmux - this is meant so you can exit ssh and leave the jupyter notebook running*

And with that setup, all you need to do is navigate to ```https://<ec2-public-dns>:8888``` and you'll have your notebook up and running and accessible! *Don't forget the `https` part of the url otherwise it wont' work*.

You should be met with a self-signed cert error. Even though it's scary, this is fine. You should be able to find a link that allows you to proceed anyways.

As a test of whether this works, let's open and run my repo [Neural Network Zoo](https://github.com/philkuz/Neural-Network-Zoo).

Start a new terminal, then enter
```
git clone https://github.com/philkuz/Neural-Network-Zoo
```
Once you see a success signal, close the window you are in, and you should see a new folder called `Neural-Network-Zoo`. Enter the folder and open any notebook you'd like, try running the cells and seeing if they work. On initial runs, it'll take some more time because tensorflow has to initialize the gpus first, but you'll have no problems on later runs.

And with that embark on your deep learning journeys friend. Let me know through git issues or pull requests what you think should be changed to this guide. 
