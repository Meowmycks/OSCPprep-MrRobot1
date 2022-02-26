# VulnHub-MrRobot

## Objective

> This VM has three keys hidden in different locations. Your goal is to find all three. Each key is progressively difficult to find.

We'll download the VM from [here](https://www.vulnhub.com/entry/mr-robot-1,151/) and set it up with VMWare Workstation 16 Player.

Upon starting the VM, we're greeted with an aesthetically pleasing login prompt.


![loginscreen](https://user-images.githubusercontent.com/45502375/155828077-4e243c66-8bc6-4fd5-9c19-37c7db487c02.png)


We can't do much just staring at this, so let's get to work on our own machine.


## Getting Key 1 of 3

We'll use the following commands to get a feel for the environment we're working with.

We'll start with ```ifconfig``` to know what our IP address, allowing us to infer what IP address the target machine may also have.


![ifconfig](https://user-images.githubusercontent.com/45502375/155828519-315216a5-1de2-40c4-89a7-8fce9b4cb5f4.png)


Knowing our IP address is ```192.168.118.128```, we'll use ```sudo nmap -sn 192.168.118.1/24``` to reveal all other machines on the network.


![scan1](https://user-images.githubusercontent.com/45502375/155828514-ae8d71c7-38be-4c5a-8aa8-125fe24b4484.png)


We can see that our target machine has the IP address of ```192.168.118.129```, now we can use more aggressive scans.

Using ```sudo nmap -sS -v -sV --version-all 192.168.118.129```, we see that the ports ```443``` and ```80``` are open. Let's dig deeper.


![scan2](https://user-images.githubusercontent.com/45502375/155828511-fa019c0c-b9ee-4734-be56-e1ecb9be9d57.png)


We'll enumerate the website for folders using the command ```sudo nmap -sS -v -sV --version-all --script http-enum.nse -p 80 192.168.118.129```.


![scan3](https://user-images.githubusercontent.com/45502375/155828526-ec80f728-86f8-4d4e-9819-5b14d4aaf973.png)


We can see loads of interesting stuff to look at now, including a ```robots.txt``` file and the ```wp-login.php``` page.

