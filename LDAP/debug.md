# clean log file

If you use the openLDAP-server you will probably run into the problem that the *.log-files located in the DB’s folder will comsume more and more free space.
It seems that the automatic maintenance service that should clean up no longer used log-files is somehow broken.
Luckly there is a command that removes any log-files that aren’t needed any more:

db_archive -d -h

In CentOS, the DB files normally reside at /var/lib/ldap, so the resulting line would be:

db_archive -d -h /var/lib/ldap

Of couse, you could also add this as a daily cronjob:

## LDAP DB maintenance

```sh
0 3 * * * /usr/bin/db_archive -d -h /var/lib/ldap
```