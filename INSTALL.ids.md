Here's how to set up the qmail groups and the qmail users.

On some systems there are commands that make this easy. Solaris and
Linux:

   ```
   # groupadd nofiles
   # useradd -g nofiles -d /var/qmail/alias alias
   # useradd -g nofiles -d /var/qmail qmaild
   # useradd -g nofiles -d /var/qmail qmaill
   # useradd -g nofiles -d /var/qmail qmailp
   # groupadd qmail
   # useradd -g qmail -d /var/qmail qmailq
   # useradd -g qmail -d /var/qmail qmailr
   # useradd -g qmail -d /var/qmail qmails
   ```

FreeBSD 2.2:

   ```
   # pw groupadd nofiles
   # pw useradd alias -g nofiles -d /var/qmail/alias -s /nonexistent
   # pw useradd qmaild -g nofiles -d /var/qmail -s /nonexistent
   # pw useradd qmaill -g nofiles -d /var/qmail -s /nonexistent
   # pw useradd qmailp -g nofiles -d /var/qmail -s /nonexistent
   # pw groupadd qmail
   # pw useradd qmailq -g qmail -d /var/qmail -s /nonexistent
   # pw useradd qmailr -g qmail -d /var/qmail -s /nonexistent
   # pw useradd qmails -g qmail -d /var/qmail -s /nonexistent
   ```

BSDI 2.0:

   ```
   # addgroup nofiles
   # adduser -g nofiles -H/var/qmail/alias -G,,, -s/dev/null -P'*' alias
   # adduser -g nofiles -H/var/qmail -G,,, -s/dev/null -P'*' qmaild
   # adduser -g nofiles -H/var/qmail -G,,, -s/dev/null -P'*' qmaill
   # adduser -g nofiles -H/var/qmail -G,,, -s/dev/null -P'*' qmailp
   # addgroup qmail
   # adduser -g qmail -H/var/qmail -G,,, -s/dev/null -P'*' qmailq
   # adduser -g qmail -H/var/qmail -G,,, -s/dev/null -P'*' qmailr
   # adduser -g qmail -H/var/qmail -G,,, -s/dev/null -P'*' qmails
   ```

OpenBSD 7.9:

   Create a login class for the processes:

   ` grep '^qmaildaemon:' /etc/login.conf || printf '\nqmaildaemon:\\\n\t:nologin=/var/qmail/nologin:\\\n\t:datasize=256M:\\\n\t:maxproc=64:\\\n\t:openfiles-max=1024:\\\n\t:openfiles-cur=16:\\\n\t:stacksize-max=64M:\\\n\t:stacksize-cur=8M:\\\n\t:auth=reject:\\\n\t:path=/package/mail/qmail/bin:\\\n\t:umask=400:\\\n\n' >> /etc/login.conf `

   Find a tidy place to put the ids

   ```
   export ID_START=500 ID_END=550
   groupadd -g 65534 _nofiles
   groupadd -g ${ID_START} _qmail
   useradd -r ${ID_START}..${ID_END} -g _qmail -d /var/qmail -s /sbin/nologin -L qmaildaemon -c "Qmail queue access" _qmailq
   useradd -r ${ID_START}..${ID_END} -g _qmail -d /var/qmail -s /sbin/nologin -L qmaildaemon -c "Qmail remote" _qmailr
   useradd -r ${ID_START}..${ID_END} -g _qmail -d /var/qmail -s /sbin/nologin -L qmaildaemon -c "Qmail scheduler" _qmails
   useradd -r ${ID_START}..${ID_END} -g _nofiles -d /var/qmail/alias -s /sbin/nologin -L qmaildaemon -c "Qmail aliases owner" _alias
   useradd -r ${ID_START}..${ID_END} -g _nofiles -d /var/qmail -s /sbin/nologin -L qmaildaemon -c "Qmail network daemon" _qmaild
   useradd -r ${ID_START}..${ID_END} -g _nofiles -d /var/qmail -s /sbin/nologin -L qmaildaemon -c "Qmail logger" _qmaill
   useradd -r ${ID_START}..${ID_END} -g _nofiles -d /var/qmail -s /sbin/nologin -L qmaildaemon -c "Qmail pop/remote access" _qmailp
   ```
AIX:

   ```
   # mkgroup -A nofiles
   # mkuser pgrp=nofiles home=/var/qmail/alias shell=/bin/true alias
   # mkuser pgrp=nofiles home=/var/qmail shell=/bin/true qmaild
   # mkuser pgrp=nofiles home=/var/qmail shell=/bin/true qmaill
   # mkuser pgrp=nofiles home=/var/qmail shell=/bin/true qmailp
   # mkgroup -A qmail
   # mkuser pgrp=qmail home=/var/qmail shell=/bin/true qmailq
   # mkuser pgrp=qmail home=/var/qmail shell=/bin/true qmailr
   # mkuser pgrp=qmail home=/var/qmail shell=/bin/true qmails
   ```

On other systems, you will have to edit /etc/group and /etc/passwd
manually. First add two new lines to /etc/group, something like

   ```
   qmail:*:2107:
   nofiles:*:2108:
   ```

where 2107 and 2108 are different from the other gids in /etc/group.
Next (using vipw) add six new lines to /etc/passwd, something like

   ```
   alias:*:7790:2108::/var/qmail/alias:/bin/true
   qmaild:*:7791:2108::/var/qmail:/bin/true
   qmaill:*:7792:2108::/var/qmail:/bin/true
   qmailp:*:7793:2108::/var/qmail:/bin/true
   qmailq:*:7794:2107::/var/qmail:/bin/true
   qmailr:*:7795:2107::/var/qmail:/bin/true
   qmails:*:7796:2107::/var/qmail:/bin/true
   ```

where 7790 through 7796 are _new_ uids, 2107 is the qmail gid, and 2108
is the nofiles gid. Make sure you use the nofiles gid for qmaild,
qmaill, qmailp, and alias, and the qmail gid for qmailq, qmailr, and
qmails.
