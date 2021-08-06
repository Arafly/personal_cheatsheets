1. To get current User
	`$whoami`

2. To use previous command:
    `cd -`

3. To check your server’s IP
	`$ifconfig`
	
4. If you are on a cloud server, run the curl command below to retrieve the server’s public IP.
	`$ curl ifconfig.me`

5. In aws EBS to view your available disk devices and their mount points
	`$ lsblk`
	
6. Apply sudo for the previious command	
	`sudo !!`
	
7. You can ordinarily move files in Linux with
	`mv fileName DestinationName`

- You can still in the same vein, duplicate a file
	`cp originalFile.js duplicateFile.js`
	
8. Get a List of All Users using the /etc/passwd File
Local user information is stored in the /etc/passwd file. Each line in this file represents login information for one user. To open the file you can either use cat or less :
	`less /etc/passwd`

9. To check environmental variables
	`env`
 
10. Kernel version, distro and stuff
	`uname -a`
	
### GREP, SED & AWK
GREP finds only. SED finds and replaces.

```
^ - finds at the beginning of the line
$ - finds at the end of the line
```

- Search for a word and display the line number
    `grep -n "word" particularfile.txt`

- Search for a word at the beginning of a line
  `grep -n ^"word" particularfile.txt`

- Search for a word at the end of a line
  `grep -n "word"$ particularfile.txt`

- This gets every instance of "th" from the file
    `grep -n 'th..' partif.txt`

- Search for characters between 0 to 9
    `grep -n '[0-9]' particular.txt`
    `grep -n [A-Za-z0-9] part.md`

- Searching for square braces "[]"?
    `grep -n '\[\]' *.md'`

- Case insensitive. Use "-i" instead of "-n"

- Searches for all the pattern that starts with “lines” and ends with “empty” with anything in-between. i.e To search “lines[anything in-between]empty” in the demo_file.

`$ grep "lines.*empty" demo_file`
Two lines above this line is empty.

- Looking for strictly "is" without any substring
    `grep -w 'is'`



> Runs the last command and counts (normally what -C does is count)
"!! -c"

 
### To check space
	df -ah | htop | top
	df -h filename
	free -h
	
CPU Usage, Processes 
	ps aux | grep service
	
w | last | lastreboot
type -a cmd
ifconfig == ip addr
ip addr show eth0 | grep inet | awk '{ print $2; }' | sed 's/\/.*$//'


help cmd | man cmd

Exit status of 0 means true or success. Anything other than 0 is false
	-eq 0
	-ne == not equal
	-ge >=
	-gl <=
	
Username of user
	-un
	
Change the host name
	hostname newname
	
Renaming
	mv oldname newname
	
To search for a list of tasks running on your machine	
	sudo lsof -iTCP -sTCP:LISTEN -n -P
	
To check details about a user
	id username
	
To symbolically link files
	ln file1 file2
	
To check the difference between two similar files
	diff file1 file2
	
To view numbers in vi
	:set number

To run las command that contains a specific letter
 ! 'letter'
 
The primary user’s group is stored in the /etc/passwd file and the supplementary groups, if any, are listed in the /etc/group file..

To check for flags concerning files
	help test
	
To generate SSH key-pair
	ssh-keygen -t rsa
	
To SCP save the file under a different name, you need to specify the new file name:
	scp file.txt remote_username@10.10.0.2:/remote/directory/newfilename.txt
	scp -r folder remote_username@10.10.0.2:/remote/directory
	
Creating a new user
	sudo adduser octopus
	
Again, the default root account is disabled.. so if you’d like to give an account root access, you must add the account to the root user group.
	sudo usermod -G sudo octopus
	
An easy solution to get a more meaningful explanation for the failure is to run 
	apache2ctl configtest again
	
Update the internal package manifests
	apt-get update
	
Persist the mount
To ensure that the drive is remounted automatically after a reboot, it must be added to the /etc/fstab file. It is also highly recommended that the UUID (Universally Unique Identifier) is used in /etc/fstab to refer to the drive rather than just the device name (such as, /dev/sdc1). 

ansible mainone -m win_ping -e ansible_connection=winrm -e ansible_user=ansible-user -e ansible_password=4EsBVqpLYXTEEVN2 -e ansible_winrm_transport=basic -e ansible_winrm_server_cert_validation=ignore

sudo mount -t nfs -o rw,nosuid <NFS-Server-Private-IP-Address>:/mnt/apps /var/www

Let’s enter some texts into the terminal, followed by CRTL+D to terminate the command.

	cat > readme.txt
	This is a readme file.
	This is a new line.

The file readme.txt will now contain the two lines we’ve entered. To verify our result, we can use the cat command once again:

1. Appending Text to File Using cat
One thing we should take note of in the previous example is that it’ll always overwrite the file readme.txt. If we want to append to an existing file, we can use the “>>” operator.

	cat >> readme.txt
	This is an appended line.
	To verify that the last command has appended the file, we check the content of the file:
	
	cat >html/index.html <<EOF
	<!DOCTYPE html>
	<html>
	<head>
	<title>Success!</title>
	</body>
	</html>
	EOF
	
	
Add user n0n-interactively
Use the --gecos option to skip the chfn interactive part.
	adduser --disabled-password --gecos "" username
	adduser -u 5000 --disabled-password --gecos "" app  (this creates a username 'app' and adds to group 5000)


To generate a PEM SSH Key
	ssh-keygen -t rsa -f ~/.ssh/[KEY_FILENAME] -m PEM -P "" -C [USERNAME]
	ssh-keygen -t rsa -f ~/.ssh/id_rsa -m PEM -P "" -C araflyayinde	
	
To check Linux Distro
	Ubuntu: lsb_release -a 
	Centos: cat /etc/redhat-release
	cat /etc/os-release
	
Using Tree 
	sudo tree -L 2
	
Details of a certificate
	openssl x509 -text -noout in noobie.crt
	openssl x509 -text -noout -n noob.crt | grep Validity -A 2
