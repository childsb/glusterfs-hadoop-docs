
part 1 : finding a ldap server 

0) Set up or borrow an LDAP server.  There are many ways to do it.  One easy way: JumbBox on EC2.
 
One way to do this is written up here: http://jayunit100.blogspot.com/2013/12/an-easy-way-to-centralize.html.  

Just replace the anonymized 111.222.333 url with your jumpbox server url , the rest of the details are identical. 

part 2 : setting up a client

Now that you have your LDAP server setup, its time to setup your CLIENTS.  To see how it is that clients can magically subvert /etc/passwd to login, you can read these articles:

    http://en.wikipedia.org/wiki/Name_Service_Switch
    http://www.howtoforge.com/linux_ldap_authentication (see the reference to nsswitch.cont)

Anyways, here's the instructions to setup clients to authenticate to my mock ldap server on EC2:

1) Install some stuff:

yum install sssd
yum install pam-ldap 

2) disable TLS in /etc/sssd/sssd.conf (set "ldap_id_use_start_tls" to "False")

3) Run this super-awesome cmd line :

authconfig --useshadow --enablesssd --enablesssdauth --enablesssdauth --passalgo=sha512 --enableldap --ldapserver=ec2-111-222-333-444.compute-1.amazonaws.com --ldapbasedn='ou=users,o=Directory' --enablecachecreds --enablelocauthorize --update --enableldapauth

part 3 : do stuff with your ldap authenticated users !!! 

[root@m1 ~]# rm -rf /mnt/glusterfs/sally
[root@m1 ~]# mkdir -p /mnt/glusterfs/bob
[root@m1 ~]# chown bob /mnt/glusterfs/bob
[root@m1 ~]# chmod -R 775 /mnt/glusterfs/bob
[root@m1 ~]# runuser -l sally -c 'touch /mnt/glusterfs/bob/c'

Test 1: That directories arent world writable and that LDAP groups are honored

[root@m1 ~]# runuser -l jayunprivileged_for_test -c 'touch /mnt/glusterfs/bob/d'
touch: cannot touch `/mnt/glusterfs/bob/d': Permission denied

Test 2: That bob can write to his own directory: 
[root@m1 ~]# runuser -l bob -c 'touch /mnt/glusterfs/bob/a'

Test 3: That sally cannot write to bobs directories: 
[root@m1 ~]# runuser -l sally -c 'touch /mnt/glusterfs/bob/b'
touch: cannot touch `/mnt/glusterfs/bob/b': Permission denied

