# LMS-2-1-Automate-Deployment-Bash
Automating the deployment of the Simulations CP using Bash Scripts and packer.

## Description
To ease the task of deploying systems, system image is one of the common approaches. A system image is a collection of files containing the system configurations and data. These files are saved in an immutable image that can be used to create and deploy replica system on any other server, faster and with the exact same configurations.

The main tool/package used in this repo is:

* **Packer** is an open source tool used for creating machine images source configuration. These images can be deployed on any platform making packer a very useful tool for DevOps engineers as well as IT operations engineers.

## Pre-requisite
  1. A laptop or computer with at least 4gb of RAM
  2. **Packer**: Download and install Packer. The instructions to do this can be accessed [here](https://www.packer.io/).              
  4. **AWS Account**: Register for an account with AWS from [here](https://aws.amazon.com/)

### Steps To Set up.
If all the above pre-requisites are completed, then we can get on to setting up.

### Cloning the Repo

1. We need to clone this repo. Run this command on your terminal:
   ``` 
   git clone https://github.com/bryan-munene/LMS-2-1-Automate-Deployment-Bash.git 
   ```

2. Move inside the newly created folder by running this command on your terminal:
   ``` 
   cd LMS-2-1-Automate-Deployment-Bash
   ```
#### Customising the scripts

Inside the `LMS-2-2Automate-Deployment-Ansible` directory, there are two sub-directories `packer` and `scripts`. 

The `packer` directory contains packer template file (`devOpsDemoPacker.json`), as well as a template for declaring your variables (`sampleVars.json`). 

The `scripts` directory contains a bash script (`baseConfig.sh`). As well as a template for declaring your enviroment variables for the application (`sampleenv`).


3. Create a variables file as outlined in the `sampleVars.json` file. Name the file `vars.json`.

4. The `devOpsDemoPacker.json` contains the configurations with which Packer will use to build an AMI. You may change these configurations to suit your needs and taste. Configurations such as `ami_name`.

5. Create a varibles file for your enviroment as outlined in the `sampleenv` file. Name the file `.env`. And use these variables to replace them in the appropriate places in the `baseConfig.sh` bash script.

6. The `baseConfig.sh` file contains bash command to provision the image and they are broken down in to functions. These commands are designed to realize my goal of deploying my project, you can edit them to suit your needs.

#### Creating an AMI

_Having complete all the steps above, it is time to create an AMI(Amazon Machine Image). This will create an immutable image with the configurations and installations set in the scripts above._

7. To execute the scripts avove and create the AMI, you need first to move into the packer directory by running this command:
```
cd packer
```

8. To begin the execution of the script, run this command:
```
packer build -var-file=vars.json devOpsDemoPacker.json
```

And just like that we have created an AMI and provisioned it with bash.

Go to your AMI dashboard to view your newly created AMI.
This image can be used to create a system with these exact configurations on any platform. To learn how to do this is addressed [here](https://github.com/bryan-munene/LMS-2-1-Automate-Deployment-Bash#launching-your-ami).


**CONGRATULATIONS!!!**



## Launching Your AMI.

Now that we have created an AMI, makes sense to launch it and see what we have.
The following steps will help you achieve that.

### Pre-requisite

For this section, in addition to the earlier requirements, you'll also need the following before we get started:

1. A domain name: You can register one [here](https://my.freenom.com/) for free.

### Steps to follow

#### Launching an instance.

For this we will be launching an instance using the image we have just created.

1. Go to your AWS EC2 AMIs dashboard. 

2. Select the option `Owned by Me` at the top from the drop down menu. Select the image you wish to launch from the list that appears below that and click on the `Launch Instance` button.

_If the list is too long, you can use the search bar at the top to filter it._

3. Choose an instance type then click `Review and Launch`.

4. On the next page, click on Launch. A dialog box will pop up prompting you to create a security key pair. This will be used to authenticate you whn you try to access the instance. Give the key pair a name and **download it**. Save it in a directory where you won't accidentally delete it. 
Without this file you cannot access your instance.

5. Your instance will be created and launched after this.

6. From the panel at the bottom of your instance page. Click on the security group link attatched to your instance.

7. The next page is the security group that is attatched to your instance. On the panel at the bottom click on the `inbound` tab and then click on the `Edit` button.

8. The result is a pop up that contains the rules to the ports allowed to access th instance. `Add rule` and then add the ports we want to open. For this case we will add pots `80` for the `http` and `443` for the `https`. Then click `save`.

9. Back on our instances page, copy the public DNS on the panel at the bottom and paste it on your browser. You should be able to access your application.

#### Configuring Elastic IP

Now that our app is being served on port `80`. Lets assign it an IP address that doesn't change with the restart of the instance.

10. On the navigation panel on the left, go to the `Elastic IP` tab and click. The resulting page is where we will configure our instances elastic IP.

11. At the top click on `Allocate new Address`. Choose the `Amazon pool` option and click on create. This creates an Elastic IP. Click on the generated IP.

12. On the next page, click on `Actions` at the top of the page. And then on `Associate Address` in the resultant dropdown menu.

13. Select `instance` in the next page then choose the correct instance from the drop down menu the n click on the `Associate` button at the bottom right. That's it. Now we can use the Elastic IP we have created to acceess our app on the browser.

#### Configuring Route 53 and Linking a Domain Name

Now to make our site more conventional, we need to link it to a domain name so we don't have to access it always through the IP.

14. Access your [Route 53](https://console.aws.amazon.com/route53/home?) dash board on AWS

15. On the left navigation panel, click on `hosted zones` then on `Create Hosted zone` button from the resulting page at the top.

16. A dialog box pops on the right, add your domain name and click `Create` button at the bottom.

17. Click on the hosted zone you just created from the list and click on `created record set` at the top. This willbe the record to link your instance to your domain name.

18. A dialog box pops on the right, add your sub-domain if any. This includes `www`, `some-sub-domain`, etc. Anything you wish to appear before your domain name. This can also be empty Add the Elastic IP we configured above on the `value` field. You can add as many record sets as you deem necesary.

19. Now copy the `name servers` from the record set labeled `NS` in the type column and paste them as Name servers in your Domain name configuration page on your domain name provider's site.
Now we can access the app using our domain name.

#### Add SSL certificates

20. First access your instance from the terminal by running the command like this:
```
ssh -i "path/to/key_pair_file.pem" ubuntu@instance_public_dns
```
Substitute it with the appropriate values.

21. Once logged in to the instance, run the following commands:
```
wget https://dl.eff.org/certbot-auto
chmod a+x certbot-auto
sudo cp certbot-auto /usr/bin/
```
To install certbot

Run the following command to add `ssl` certificates to your app:
```
sudo /usr/bin/certbot-auto --nginx -d example.com -d www.example.com --debug
```
Substitute it with the appropriate values.

22. Choose to `Redirect all HTTP traffic to HTTPS` from the resulting prompt. It's ussually the second option. And all the other questions that are there.

24. Edit `nginx` configuration by adding this line to:
```
  server_name  example.com www.example.com;
```

Then restart nginx, with this command:
```
sudo service nginx restart
```
View your site on the browser and the padlock icon on the adress bar will be present showing that it is secure.


**CONGRATULATIONS YOU HAVE A SECURE WEBSITE**

