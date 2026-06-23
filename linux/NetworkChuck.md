https://www.youtube.com/watch?v=gd7BXuUQ91w

ssh usernam@8.8.8.8
ls -l
ls -al
mkdir mydir
cp file.txt ./mydir/file.txt
ls mydir
mv file2.txt ./mydir/file2.txt
rmdir -r mydir/
ln -s some.txt somelink
clear

useradd nick
adduser nick

su nick
exit

sudo passwd nick

sudo apt update // && sudo apt upgrade
sudo apt install finger
which finger
whereis finger

wget url

curl url > some.txt

zip my.zip some.txt
unzip my.zip

cat some.txt
less some.txt
head some.txt
tail some.txt

cmp some.txt some2.txt
diff some.txt some2.txt

find / -name "some*"
find . -type f -empty
find . -perm /a=x

chmod +x some.sh

sudo apt install net-tools
ifconfig
ip address
ip address | grep eth0 | grep inet | awk '{print $2}'

// dns
cat /etc/resolv.conf
resolvectl status

ping 8.8.8.8
traceroute 8.8.8.8

// open ports
netstat -tulpn
ss -tulpn

// allow port 80
ufw allow 80
ufw status
ufw enable
ufw status

uname -a
neofetch

// do maths
echo "1+2" | bc

// check memory
free

// check space
df -H

// processes
ps
ps -aux
htop

ps -aux | grep  some.sh
// kill <pid> sends SIGTERM (default), kill -9 <pid> sends SIGKILL.
kill -9 12345
pkill -f some.sh

// start/restart/stop services
systemctl stop apache2
systemctl status apache2

// see history of your cmds
history

sudo reboot
sudo shutdown -h now