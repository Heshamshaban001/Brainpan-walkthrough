# Brainpan-walkthrough

# 0. get the Vm ip 

nmap 192.168.1.0/24 >> VM ip's 192.168.1.2 

# 1. enumeration 

nmap -sCV --script vuln 192.168.1.2 -p-

![image](https://user-images.githubusercontent.com/52453415/129484923-4e4f42b0-e7a1-4bff-89e7-7795d9bbabf9.png)

nc 192.168.1.2 9999 
prompt for input and nothing else happens 

open the browser and see what we can find from page source ; nothing 

# 1.1 brute force for files and directories 

![image](https://user-images.githubusercontent.com/52453415/129485173-0fd5cdcf-58be-4c1f-8049-9e1cf38dd0b2.png)

i found interesting dir 

![image](https://user-images.githubusercontent.com/52453415/129485190-acccf0f7-9f63-4d4b-9257-fc98dd3f8b65.png)

which happens to has an exe file
![image](https://user-images.githubusercontent.com/52453415/129485202-461ecd75-f4ca-45ce-9793-00e906eba739.png)

download and open the file to see what does it do 

![image](https://user-images.githubusercontent.com/52453415/129485216-e43fdd2d-5655-4bba-b90d-cedcadddfdfe.png)

the file just listen to the port 9999 

and copy waht ever i input ; sounds like vulnerability BOF

![image](https://user-images.githubusercontent.com/52453415/129485244-9c4174f9-9d57-4d06-b133-ea2ca923930f.png)

# 2. exploiting Buffer Over Flow 

copy the vulnerable exe to my own lab first and : 

1-determine the exact size of the buffer using msfvenom_pattern_ (creat/offest) with the assist of immunity debugger 

![image](https://user-images.githubusercontent.com/52453415/129485340-e555451b-6147-478f-8279-2dd6bf363ad4.png)

![image](https://user-images.githubusercontent.com/52453415/129485349-b99ba801-6356-44ca-8c77-f18e4631849a.png)

now the the exact size is 524 then come the next for bytes for the EIP address which holds the address for the next instruction to be fetched

2- now we need to control the EIP to jump to the ESP which will have our shell code 

using emona script and immunity debugger : searching for a module that has no memory protection and has an fixed address for the instruction 
JMP ESP 

which happens to be in module named brainpan.exe 

![image](https://user-images.githubusercontent.com/52453415/129485454-20bf6dfb-1b86-4818-b64f-57c9e9312c5a.png)


![image](https://user-images.githubusercontent.com/52453415/129485443-6bdda56d-ec93-492f-ad9e-b34d4a920f90.png)

3-now time to put our shell into ESP space but before that we need to hunt for bad characters 

4-we send whole 254 hexa chars to find out using immunity which told us that the only bad character was the null byte 

![image](https://user-images.githubusercontent.com/52453415/129485643-e52d9907-4034-4ee7-ae4a-52585d3d31fc.png)

so generate the payload from msfvenom and inject it to the esp space let the eip jump to esp and excute our shell

![image](https://user-images.githubusercontent.com/52453415/129485512-dd22bac5-8232-497a-816d-b8dfd81164cb.png)


![image](https://user-images.githubusercontent.com/52453415/129485545-8ad08bb9-aee5-4b06-94f4-d76452124b04.png)

set the listener and send the payload 

![image](https://user-images.githubusercontent.com/52453415/129485560-73ac68f6-6ea8-40b2-a6d8-0d6e82fe86b1.png)

and we got our shell 

# 3. Priv Esc 

first things first 

sudo -l 

found that i can run a program with root privlige without password 

![image](https://user-images.githubusercontent.com/52453415/129485651-d62270d3-f0dd-44ce-96be-039d06049803.png)

![image](https://user-images.githubusercontent.com/52453415/129485661-dc7cfa41-edbb-449e-839f-c24208ecc010.png)

will , the command manual has an interactive prompt if we pass a commands to it it will excute it as root 

so 

try 
sudo /home/anansi/bin/anansi_util manual id 

then return and write !/bin/sh 

whoami >> root

