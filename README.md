# Boot2Root CTF - *Mr-Robot: 1*

## Objective

> This VM has three keys hidden in different locations. Your goal is to find all three. Each key is progressively difficult to find.

## Beginning the Challenge

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

![404phptemplate](https://user-images.githubusercontent.com/45502375/155830042-8ae883b8-40af-4ae7-85e4-2876488844c5.png)

We find that there's PHP templates included. These are webpages that will automatically load during certain situations. For example, the ```404.php``` template will show up whenever a requested resource is not found.

However, this also means that the code in these templates can be replaced with any other PHP code, and if these webpages are opened, then the new code will execute instead.

Therefore, we will replace the code in the ```404.php``` file with code to execute a PHP Reverse Shell. We place it in the ```404.php``` template because we can easily execute the code by just going to any nonexistent resource.

Kali Linux includes a few very good PHP Reverse Shell scripts, and we will be using one of those.

We'll find the absolute path for our pre-included scripts by using ```locate php-re``` and use the script that is in the ```webshells``` directory.

![locate_php-re](https://user-images.githubusercontent.com/45502375/155829992-df6b094c-0f1f-4f1e-98a3-b579a589b763.png)

We will then replace the code in the ```404.php``` template with the code from our ```php-reverse-shell.php``` script.

![404phptemplate_replaced](https://user-images.githubusercontent.com/45502375/155830102-1068b9f0-644f-4ad0-87ef-43d8dec5ea5b.png)

We will also change the ```$ip``` and ```$port``` values to ones of our host machine so that we can receive the connection from the reverse shell.

![ipandport](https://user-images.githubusercontent.com/45502375/155830155-3d8e5e14-4e52-4b30-b9d3-c46db090d9d2.png)

Let's test to see if this worked.

First we start a NetCat listener using ```nc -lvvp 5555```...

![nc1](https://user-images.githubusercontent.com/45502375/155830240-7a03203b-02a2-43ef-a580-8e8824b9a278.png)

...then we request a nonexistent resource...

![thispagedoesnotexist](https://user-images.githubusercontent.com/45502375/155830247-885f0ef9-2897-4a13-99d8-c26176ea9e9e.png)

...and we're in.

![nc2](https://user-images.githubusercontent.com/45502375/155830258-825f522c-a92e-4cec-be83-973f4aae38e5.png)

Now that we have a shell, we'll "upgrade" it to a semi-tty shell by using the command ```python -c 'import pty; pty.spawn("/bin/bash")'```

![pythontty](https://user-images.githubusercontent.com/45502375/155830345-864192cd-e7f5-4e3c-bb7d-4d2b3e36765c.png)

We see that we are logged in as ```daemon```, and we start looking around the machine for stuff.

Here, we've found that there's another account named ```robot```, and in its home folder is the second key and a file called ```password.raw-md5```.

![lsandcdintorobot](https://user-images.githubusercontent.com/45502375/155830405-805162cc-5510-4db0-880d-6fce9aaa9ebc.png)

Let's try to open them both.

We can't print the second key, but we *can* print the password file. Doing so gives us an MD5 hash, unsurprisingly.

![catkey_catpassword](https://user-images.githubusercontent.com/45502375/155830460-f136fc32-730c-476d-9410-4a5087da8dbc.png)

We can throw this hash into Hashcat and crack it quickly using the RockYou wordlist.

![hashcat](https://user-images.githubusercontent.com/45502375/155830695-42cd5920-4077-4892-ad8a-462c7926a6a9.png)

The password is revealed to be the very long, secure, and complicated string of ```abcdefghijklmnopqrstuvwxyz```.

So now we have the credentials of ```robot : abcdefghijklmnopqrstuvwxyz```, and we can use ```su robot``` to switch to his account and reveal our second key.

![key2of3](https://user-images.githubusercontent.com/45502375/155830834-8ffe951f-c68c-4a9e-9ce9-20eade092ed3.png)

## Getting Key 3 of 3

We first went from a Service User account to a Regular User account. Now, we'll have to go from a Regular User account to a Super User account.

However, we can't use ```sudo```, meaning we can't just use the ```sudo su``` command.

![nosudo](https://user-images.githubusercontent.com/45502375/155830925-b508da40-2661-4150-aa17-e2acecdeda80.png)

So if we can't do things as root, then we'll try to find other files that *can* run as root.

To do so, we'll use the command ```find / -perm -u=s -type f 2>/dev/null``` to find files that have root privileges.

![findrootfiles](https://user-images.githubusercontent.com/45502375/155831039-5f717606-316c-4c04-afad-e13c67b1f586.png)

We find the program ```nmap``` here, so we will check what version it is.

![nmapversion](https://user-images.githubusercontent.com/45502375/155831065-539abc94-619f-457f-b0ca-646d7d4efefa.png)

The version is 3.81. This version of Nmap supported an option called “interactive.” With this option, users were able to execute shell commands by using an nmap “shell” (interactive shell). Therefore, we will search for ways to exploit this shell.

By searching online for exploits for Nmap version 3.81, we find that there are several results for a privilege escalation exploit. This is exactly what we need to be able to obtain root and get the final key.

![searchforexploits](https://user-images.githubusercontent.com/45502375/155831335-c38582cf-a2b2-4dcf-a83e-4520b9421a18.png)

The first result shows us the commands necessary to execute the privilege escalation exploit.

![privilegeescalation](https://user-images.githubusercontent.com/45502375/155831602-a46340b3-46ea-4a5b-9a47-eddffb63c719.png)

- First we enter Nmap's interactive mode using ```nmap --interactive```.
- Then we create a file in the user's folder using ```!touch x```.
- After this, we use the command ```!find x -exec /bin/sh \;```.
- Now we are root, and we can verify this by seeing the ```#``` prompt, and by using ```whoami```.

![gainingroot](https://user-images.githubusercontent.com/45502375/155831613-c6b74060-e81d-43d2-8c99-905e82699e44.png)

After that, it's as simple as searching through the ```root``` folder and finding the last key.

![key3of3](https://user-images.githubusercontent.com/45502375/155831652-a6500628-2423-47ab-9899-d2eba7e38f20.png)

## Notes

This challenge took several hours to complete, and I couldn't have done it alone. 

Not only did I come out of this learning new skills and fortifying existing ones, I also learned that sometimes it's okay to ask for help. (It's certainly much better than sitting stuck on one step forever.)

The box was not too difficult except for two points where I had figure out how to create the PHP Reverse Shell and how to exploit the Nmap program.

Regardless, this was a really cool learning experience and I am eager to try other challenges.

[here](https://www.vulnhub.com/entry/napping-101,752/)
