# Looking Glass: BIRD configuration and tips.

Only BIRD on Debian GNU/Linux and how to (merely) secure an restricted ssh
user will be detailed. Other OSes were not tested.

Security will be mainly based on shell, path and ssh access restriction.
Password authentication will not even be presented here, only key based
authentication.

## Dependencies

  * Debian: `apt-get install traceroute bash iputils-ping`
  * BIRD: a functionnal installation of BIRD, assuming group bird exists
    (requires at least the Debian package from wheezy-backports)

## Security and user access

Looking Glass directly calls `birdc "bird command"` and
`birdc6 "bird command"`. Thus, the `lg` user only needs to run `birdc`,
`birdc6`, `ping` and `traceroute`. To achieve this, we recommend the use of
`rbash` (restricted bash, see [1]), ssh key based authentication and a bit of
dark magic.

## Configuration

Rough steps ahead (maybe more doc later):

```
# create the "lg" unix user and add it to "bird" group
root@bird-router ~# adduser lg
(boring questions)
root@bird-router ~# adduser lg bird

# log in as lg user
root@bird-router ~# su -l lg

# create ssh userdir and authorized the looking glass RSA pubkey with limited access and features
lg@bird-router ~# mkdir ~/.ssh/
lg@bird-router ~# echo 'from="lg.example.com,$IP4-OF-YOUR-LG",no-port-forwarding,no-x11-forwarding,no-agent-forwarding ssh-rsa $RSA-PUBKEY-HERE lg@looking-glass' >| ~/.ssh/authorized_keys

# truncate the profile dotfile (Debian-based distributions)
# for CentOS/RHEL, apply the below steps to ~/.bashrc instead
lg@bird-router ~# echo >| ~/.profile

# set up a limited PATH
lg@bird-router ~# echo "export PATH=/opt/lg-bin" >| ~/.profile
lg@bird-router ~# exit

# render the profile dotfile immutable, the lg user will not be able to truncate/edit it
root@bird-router ~# chattr +i ~lg/.profile

# change lg user shell to restricted bash
root@bird-router ~# chsh -s /bin/rbash lg

# set up the restricted PATH with the only necessary binaries simlinks
root@bird-router ~# mkdir -p /opt/lg-bin
root@bird-router ~# for cmd in birdc birdc6 ping ping6 traceroute traceroute6; do ln -s $(which $cmd) /opt/lg-bin/; done
root@bird-router ~#
```

You can disable password authentication for the lg user in the sshd config:

```
Match user lg
  PasswordAuthentication no
```

and reload sshd:

`service ssh reload`

## Debug

Test the SSH connection from the server where the looking glass is installed:

`ssh -i lg-user-id_rsa.key lg@bird-router.example.com`

After successful login, verify that only built-in functions and `birdc`,
`birdc6`, `ping` and `traceroute` are available and functionnal.

## References

  * [1] http://en.wikipedia.org/wiki/Restricted_shell
