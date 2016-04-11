# Learn Capistrano
------

This is my project to learn, how to work with Capistrano

---

## Prepare server

* [Authentication](#authentication)
* [Authorisation](#authorisation)

### Authentication & Authorisation

To create this deploy user we'll assume something like the following has been
done:

``` sh
root@remote $ adduser deploy
root@remote $ passwd -l deploy
```

#### Authentication

We need to create the key.

``` sh
me@localhost $ ssh-keygen -t rsa -C 'me@my_email_address.com'
```


We can see which keys are loaded in the SSH agent by running ssh-add -l

``` sh
me@localhost $ ssh-add -l
2048 af:ce:7e:c5:93:18:39:ff:54:20:7a:2d:ec:05:7c:a5 /Users/me/.ssh/id_rsa (RSA)
```

If you don't see any keys listed, you can simply run ssh-add:

``` sh
me@localhost $ ssh-add
Identity added: /Users/me/.ssh/id_rsa (/Users/me/.ssh/id_rsa)
```

Typically, ssh-add will ask you for the passphrase when you add a key.


To view public key:

``` sh
me@localhost $ ssh-add -L
ssh-rsa jccXJ/JRfGxnkh/8iL........dbfCH/9cDiKa0Dw8XGAo01mU/w== /Users/me/.ssh/id_rsa
```

Add public key to remote server's `authorized_keys`:

``` sh
me@localhost $ ssh-add -L
ssh-rsa jccXJ/JRfGxnkh/8iL........dbfCH/9cDiKa0Dw8XGAo01mU/w== /Users/me/.ssh/id_rsa
```

If we did all that correctly, we should now be able to do something like this:

``` sh
me@localhost $ ssh deploy@one-of-my-servers.com 'hostname; uptime'
one-of-my-servers.com
19:23:32 up 62 days, 44 min, 1 user, load average: 0.00, 0.01, 0.05
```

#### SSH Agent Forwarding

Here's how we can check if that works, first get the URL of the repository:

``` sh
me@localhost $ git config remote.origin.url
git@github.com:capistrano/rails3-bootstrap-devise-cancan.git
```


We can try to access the repository via our server by doing the following:

``` sh
# List SSH keys that are loaded into the agent
me@localhost $ ssh-add -l
# Make sure they key is loaded if 'ssh-add -l' didn't show anything
me@localhost $ ssh-add
me@localhost $ ssh -A deploy@one-of-my-servers.com 'git ls-remote git@github.com:capistrano/rails3-bootstrap-devise-cancan.git'
```

If you get the error "host key verification failed." log in into your server and
run as the deploy user the command ssh git@github.com to add github.com to the list of known hosts.

## Authorisation

To configure this hierarchy, ignoring for the moment the passwordless sudo access that you may or may not need depending how well your servers are setup:

``` sh
me@localhost $ ssh root@remote
# Capistrano will use /var/www/....... where ... is the value set in
# :application, you can override this by setting the ':deploy_to' variable
root@remote $ deploy_to=/var/www/rails3-bootstrap-devise-cancan-demo
root@remote $ mkdir -p ${deploy_to}
root@remote $ chown deploy:deploy ${deploy_to}
root@remote $ umask 0002
root@remote $ chmod g+s ${deploy_to}
root@remote $ mkdir ${deploy_to}/{releases,shared}
root@remote $ chown deploy ${deploy_to}/{releases,shared}
```


Check permissions:

``` sh
root@remote # stat -c "%A (%a) %n" ${deploy_to}/
drwx--S--- (2700)  /var/www/rails3-bootstrap-devise-cancan-demo

root@remote # stat -c "%A (%a) %n" ${deploy_to}/*
drwxrwsr-x (2775)  /var/www/rails3-bootstrap-devise-cancan-demo/releases
drwxrwsr-x (2775)  /var/www/rails3-bootstrap-devise-cancan-demo/shared
```
