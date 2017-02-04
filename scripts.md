# vinod
#!/bin/bash
# After reading this script you'll get the idea/concept. You can simply
# add/remove any commands that you wish to run.
# Script simply runs each command and appends the output of that
# command to the log file. Log file is mailed out at the end.
# I created all the temp files in /root but you could also put them in
# a sticky bit directory for extra security. You can also customize all of the file
# names and directory structure to create a unique setup.
# The following was successfully run on redhat 7.0 but was also easily modified to
# run on redhat 7.3 and FreeBSD 4.x.

# variables:
TMPLOG="/root/.sysaudit.log"
SERVERNME=`/bin/hostname`
ERRLOG="/root/.errortmp"
CHKTMP="/root/.chktmp"
TMPLOGE="/root/.sysaudit.log.asc"
ENCRPKEY="<admins email address>"
# Mail page header:
echo "##########################################" >$TMPLOG
echo " " >>$TMPLOG
echo " System Audit " >>$TMPLOG
echo " " >>$TMPLOG
echo "##########################################" >>$TMPLOG
# log the time and date
/bin/date >> $TMPLOG
# system info:
/bin/uname -a >> $TMPLOG
# System name:
echo " Server: " >>$TMPLOG
echo $SERVERNME >>$TMPLOG
# Functions
# This function is called to insert a line/cmd seperator in the log file.
# makes it easier to read the emailed log file.
seperator()
{
echo " " >>$TMPLOG
echo " *************************" >>$TMPLOG
echo " " >>$TMPLOG
}
# Commands to run:
############### File System ###############
seperator
echo "File system check:" >>$TMPLOG
echo number of SUID files: >>$TMPLOG
/usr/bin/find / -type f \( -perm -04000 -o -perm -02000 \) -ls |wc -l >>$TMPLOG
seperator
echo world writable files: >>$TMPLOG
/usr/bin/find / -type f \( -perm -2 \) -ls |wc -l >>$TMPLOG
seperator
echo world writable directories: >>$TMPLOG
/usr/bin/find / -type d \( -perm -2 \) -ls |wc -l >>$TMPLOG
seperator
echo un-owned files: >>$TMPLOG
/usr/bin/find / -nouser -o -nogroup >>$TMPLOG
seperator
echo number of core files: >>$TMPLOG
/usr/bin/find / -name core | wc -l >>$TMPLOG
seperator
echo files modified in the last day: >>$TMPLOG
/usr/bin/find /etc -mtime -1 >>$TMPLOG
seperator
echo Hard drive usage: >>$TMPLOG
/bin/df -h >>$TMPLOG
seperator
echo "permissions on /etc/shadow and others:" >>$TMPLOG
/bin/ls -l /etc/shadow >>$TMPLOG
/bin/ls -l /usr/bin/crontab >>$TMPLOG
/bin/ls -l /bin/mount >>$TMPLOG
/bin/ls -l /bin/umount >>$TMPLOG
/bin/ls -l /etc/crontab >>$TMPLOG
/bin/ls -ld /etc/cron.daily >>$TMPLOG
/bin/ls -ld /etc/cron.d >>$TMPLOG
seperator
# I ran rpm verification only on critical network tools since
# running a complete rpm database check can be verbose.
echo "rpm check on net-tools (null = OK):" >>$TMPLOG
/bin/rpm -V net-tools >>$TMPLOG
seperator
echo Script files info: >>$TMPLOG
/bin/ls -alhR /mnt/floppy/.sysa >>$TMPLOG
############# System Processes #############
seperator
echo " System Processes " >>$TMPLOG
echo Processes running: >>$TMPLOG
/bin/ps aux >>$TMPLOG
echo number of running processes: >>$TMPLOG
/bin/ps aux | wc -l >>$TMPLOG
seperator
echo services set to run: >>$TMPLOG
 /sbin/chkconfig --list >>$TMPLOG
seperator
echo modules loaded: >>$TMPLOG
/sbin/lsmod >>$TMPLOG
seperator
echo TOP >>$TMPLOG
/usr/bin/top -b -n 1 >>$TMPLOG
################ Networking #############
seperator
echo " Networking " >>$TMPLOG
echo IFCONFIG: >>$TMPLOG
/sbin/ifconfig >>$TMPLOG
seperator
echo NETSTAT current connections: >>$TMPLOG
/bin/netstat -an >>$TMPLOG
seperator
echo "RPC Info (null= no RPC services running= ok):" >>$TMPLOG
/usr/sbin/rpcinfo -p >>$TMPLOG
seperator
# I run a ping to check status on critical servers.
# Adjust or comment out.
echo Ping >>$TMPLOG
/bin/ping -c 2 <insert IP here> >>$TMPLOG
################ System Accounting #######
seperator
echo " System Accounting " >>$TMPLOG
echo system uptime: >>$TMPLOG
/usr/bin/uptime >>$TMPLOG
seperator
echo last log on list: >>$TMPLOG
/usr/bin/last >>$TMPLOG
seperator
echo who is logged on now: >>$TMPLOG
/usr/bin/who >>$TMPLOG
seperator
echo cron jobs: >>$TMPLOG
/usr/bin/crontab -l >>$TMPLOG
seperator
echo UID 0 accounts: >>$TMPLOG
#/bin/awk -F: ($3 == 0) '{ print $1 }' /etc/passwd >>$TMPLOG
seperator
echo "system accounting/trend analysis:" >>$TMPLOG
 /usr/bin/sar -u -n DEV >>$TMPLOG
################# Applications #############
seperator
# Note here that logcheck once invoked has no std output so its report
# is mailed seperately. I don't know about logcheck but, logwatch has
# the ability to configure a std output that you could append to this log.
# Substitute with your favorite log parser.
echo " Applications " >>$TMPLOG
echo "Logcheck (mailed seperately):" >>$TMPLOG
 /usr/bin/logcheck.sh
seperator
echo chkrootkit says: >>$TMPLOG
/root/chkrootkit >>$TMPLOG
# md5sum
# I used a temp file so I could grep out keywords (FAILED) since
# md5sum --check is very verbose. This will tell you how many total hashes,
# how many were OK, and which ones failed.
# Please note that you will see /etc/mtab file fail because with this
# implementation, the md5sum file was created when floppy was rw.
seperator
echo md5sum hashes reports: >>$TMPLOG
/mnt/floppy/.sysa/md5sum --check /mnt/floppy/.sysa/.hcheck/.hashdb >$CHKTMP 2>>$TMPLOG
echo "number of files checked OK:" >>$TMPLOG
/bin/grep -c OK $CHKTMP >>$TMPLOG
echo "number of files failed checksum:" >>$TMPLOG
/bin/grep FAILED $CHKTMP >>$TMPLOG
# Adding Tripwire stuff
seperator
echo Tripwire report: >>$TMPLOG
/usr/sbin/tripwire --check >>$TMPLOG
################# LOGS ###############
# Add any other system log files that you wish to
# review and archive.
seperator
echo " System Logs " >>$TMPLOG
echo the MESSAGES log: >>$TMPLOG
/bin/cat /var/log/messages >>$TMPLOG
# append the sysaudit.sh error log at the end
seperator
echo Error log >>$TMPLOG
/bin/cat $ERRLOG >>$TMPLOG
################# Done ################
# At then end of the Audit the log file is mailed out.
# encrypt the log file first:
 /usr/bin/gpg --batch -ea -r $ENCRPKEY $TMPLOG
# Then mail it. I typically send it to root then configure root’s .forward file.
# Adjust as necessary.
 /bin/mail -s sysaudit root <$TMPLOGE
# Clean up
 /bin/rm -f $ERRLOG
 /bin/rm -f $TMPLOG
 /bin/rm -f $TMPLOGE
 /bin/rm -f $CHKTMP
