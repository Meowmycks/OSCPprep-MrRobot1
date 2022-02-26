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

![homepage](https://user-images.githubusercontent.com/45502375/155828649-924cd7ee-0c5a-461a-b6e0-513605a9b679.png)

We've confirmed that this is in fact a webserver, given that we can see a pretty cool website upon entering the IP address into our browser.

Let's take a look at some of those other files we found in our enumeration scan. We'll go to ```192.168.118.129/robots.txt``` next.

![robotstxt](https://user-images.githubusercontent.com/45502375/155828877-a1a009cb-862a-4fad-82a6-ecece41bb633.png)

We find some interesting files to look at, one of them being our first key.

![key1of3](https://user-images.githubusercontent.com/45502375/155828880-26198b4c-9eac-4b25-9fef-3d47ebd4de29.png)

Step 1 is done.

## Getting Key 2 of 3

We also saw a file called ```fsocity.dic``` in the ```robots.txt``` list. We'll download it now.

![fsocitydic](https://user-images.githubusercontent.com/45502375/155829014-76d994a6-50f6-4c0a-8b2e-ac8633dc91f6.png)

My first instinct when downloading the file was to try looking through it for the second key, but it's just a wordlist with loads of duplicates.

![cat_fsocitydic](https://user-images.githubusercontent.com/45502375/155829048-314a062c-6a3d-44af-aa71-b784642df5b5.png)

After using ```sort -u fsocity.dic > filtered.dic``` to clean up the wordlist and checking it again, it looks much better.

![sort_and_cat_filtered](https://user-images.githubusercontent.com/45502375/155829144-a4b332d6-607f-4256-8786-37582c9dde1f.png)

The next thing I thought to do was to try and login to the Wordpress page, figuring that this wordlist would help me do that.

I tried using the username ```admin```, ```administrator```, and similar usernames hoping to find a username enumeration exploit.

![wplogin_admin](https://user-images.githubusercontent.com/45502375/155829349-a86bf3fe-7d4d-4c50-abf1-856202c6cd1d.png)

I eventually tried using the name of the main character of the TV series that this box was based on, ```elliot```. This successfully revealed that ```elliot``` was an account registered with this Wordpress site.

![wplogin_elliot](https://user-images.githubusercontent.com/45502375/155829355-bf5381e1-fe84-4ee7-ab8f-5eec62b81ecc.png)

Now that we had a valid username, we could start dictionary-attacking the account with our ```filtered.dic``` wordlist.

We'll employ the use of WPScan for this, allowing us to enumerate through a password list to brute-force the ```wp-login.php``` page and obtain credentials.

Using ```sudo wpscan --url http://192.168.118.129/wp-login.php -U "elliot" -P filtered.dic```, we eventually got the password to access Elliot's account.

![wpscan_bruteforce](https://user-images.githubusercontent.com/45502375/155829452-b6ae3460-a1fd-4511-b3f8-302698e7450c.png)

We now had the credentials ```elliot : ER28-0652``` and could now login to the Wordpress site.

![wordpress_dashboard](https://user-images.githubusercontent.com/45502375/155829708-f7a30fac-44dd-4771-bf54-a49f268fb70b.png)

Earlier, our WPScan had also revealed that there was an outdated theme installed as well.

![wpscan_theme](https://user-images.githubusercontent.com/45502375/155829779-0599a7c8-3e63-47c6-a43b-f8235c2d66cc.png)

Knowing this, let's look at the ```Appearance -> Editor``` tab.

We find that there's PHP templates included. These are webpages that will automatically load during certain situations. For example, the ```404.php``` template will show up whenever a requested resource is not found.

However, this also means that the code in these templates can be replaced with any other PHP code, and if these webpages are opened, then the new code will execute instead.

Therefore, we will replace the code in the ```404.php``` file with code to execute a PHP Reverse Shell. We place it in the ```404.php``` template because we can easily execute the code by just going to any nonexistent resource.

Kali Linux includes a few very good PHP Reverse Shell scripts, and we will be using one of those.

We'll find the absolute path for our pre-included scripts by using ```locate php-re``` and use the script that is in the ```webshells``` directory.

![locate_php-re](https://user-images.githubusercontent.com/45502375/155829992-df6b094c-0f1f-4f1e-98a3-b579a589b763.png)

