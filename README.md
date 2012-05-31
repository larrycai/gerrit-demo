# gerrit-demo

This is the system setup for gerrit demo in one machine one account, mainly for windows, and it should work for other OS as well.

This repo will be cloned to gerrit besides guideline and some configuration data

# System setup in general
In order to demo, gerrit system needs to be installed and well configured in advance.

## Gerrit installation
1. Install [Git for Windows](http://msysgit.github.com) to have git environment
2. Install gerrit using development mode for authentication, others are use default to keep simple.

    $ java -jar gerrit-2.4.war init --batch  -d demo24
    $ git config  --file demo24/etc/gerrit.config auth.type DEVELOPMENT_BECOME_ANY_ACCOUNT
    
## Gerrit server configuration 
1. Integrate gitweb, see [blog:integrate gitweb with gerrit without apache server in windows](http://codeslife.com/2012/05/21/integrate-gitweb-with-gerrit-without-apache-server-in-windows/)
2. Integrate github issue system to demo links

Below are the `gerrit.config` after configuration

    $ less etc/gerrit.config
    [gerrit]
            basePath = git
    [database]
            type = H2
            database = db/ReviewDB
    [auth]
            type = DEVELOPMENT_BECOME_ANY_ACCOUNT
    [sendemail]
            smtpServer = localhost
    [container]
            user = RDCCAIY
            javaHome = C:\\Program Files\\Java\\jre6
    [sshd]
            listenAddress = *:29418
    [httpd]
            listenUrl = http://*:8080/
    [cache]
            directory = cache
    [gitweb]
            cgi = c:/Program Files/git/share/gitweb/gitweb.bat
    [commentlink "githubissue"]
            match = "issue\\s+#?(\\d+)"
            link = "https://github.com/larrycai/gerrit-demo/issues/$1"

## Gerrit web customization
The theme is customized as well, and it is reuse from reusing [openstack](review.openstack.org)/[eclipse](https://git.eclipse.org/r/) 's gerrit configuration

See the sample below

![gerrit demo system](https://github.com/larrycai/gerrit-demo/raw/master/demo-standard.png)

## Gerrit account configuration

Now you can start the gerrit

    $ java -jar gerrit.war daemon -d demo24

And become any account to update "Full name", "username" and upload "SSH Public key", Since the development mode is used, first account admin by default with id `1000000`.

### Create users (account)
Now let's create the user

    $ ssh -p 29418 larrycai@localhost gerrit create-account tomas
    $ ssh -p 29418 larrycai@localhost gerrit create-account vivian
    $ ssh -p 29418 larrycai@localhost gerrit create-account jenkins
    
### Create group 
Let's create the group first for normal user group and senior group

    $ ssh -p 29418 larrycai@localhost gerrit create-group --member larrycai --member tomas -d "NormalUser" user
    $ ssh -p 29418 larrycai@localhost gerrit create-group --member vivian -d "SeniorDeveloper" senior

### Create default project permission
Change the access permission for All-projects 

    Create Reference on refs/* for Administrator group
    
    Read on refs/heads/* and refs/tags/*
    Push to refs/for/refs/heads/* and refs/changes/*
    Push merge commit to refs/for/refs/heads/* and refs/changes/*
    Forge Author Identity
    Label: Code review with range -2 to +2
    Label: Verify with range -1 to +1

Then we get the matrix like below

 * Larry (larrycai) 1000000 is admintration used to create the project and permission.
 * Larry (larrycai) is normal user who submit the codes
 * Vivian (vivian) is senior group has -2..+2 permission
 * Tomas (tomas) is normal user with -1..+1 permission 
 * Larry (larrycai) is also CI user for -1..+1 verify permission
 
###  
 
## Jenkins installation
Just jenkins trigger plugin, no special needs

# Import github project into gerrit
Now you are able to create project and push cloned github project

    $ git clone git://github.com/larrycai/gerrit-demo.git # may switch to other protocol
    $ ssh -p 29418 larry@localhost gerrit create-project --name gerrit-demo
    $ cd gerrit-demo
    $ git push ssh://larry@localhost:29418/gerrit-demo *:*
    
# Use case 
## normal case
User `tomas` fix the issue with commit message

    $ git commit -m"solve the issue #1 for how to deal with multi-user
    
    1. this is blah
    2. this is blah, blah
    
    

