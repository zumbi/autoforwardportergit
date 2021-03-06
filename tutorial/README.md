# autoforwardportergit tutorial.

This tutorial will demonstrate the use of the autoforwardporter. 
For demonstration purposes we will use a demonstration repository as our 
downstream distribution and Debian as our upstream distribution. We will
assume that you have cloned the autoforwardportergit repository into 
~/autoforwardportergit

We use the "hello" package from Debian as the subject of our demonstration. 

## installing dependencies

This is the only step that requires root privilages. All remaining steps can
be performed by a normal user.

    apt-get install build-essential git dgit python3 python3-debian python3-git moreutils rcs quilt reprepro python3-bs4

## Preparing the "old downstream version" of the demonstration package

The old downstream version of the demonstration package is shipped in the form
of a debdiff. We need to convert this to an actual package.

    cd ~/autoforwardportergit/tutorial
    ../snapshotsecure hello 2.9-2
    dpkg-source -x hello_2.9-2.dsc
    cd hello-2.9
    patch -p1 < ../hello_2.9-2+test1.debdiff
    dpkg-buildpackage -S
    cd ..

## Setting up the working repo

A configuration for the working repo is provided already in the 
tutorial/workingrepo/conf directory. This repo has two suites.

sid , this represents the downstream distribution.
sid-deb , this represents debian sid.

Reprepro needs the key for the debian repository to be in your user keyring. We
can do this with

    gpg --recv-keys 8B48AD6246925553

We also want to add the demonstration package we built above to our
downstream distribution.

    cd ~/autoforwardportergit/tutorial/workingrepo
    reprepro includedsc sid ../hello_2.9-2+test1.dsc

## creating the autoforwardportergit configuration and directories

Now we need to configure autoforwardportergit. Configuration is stored in
~/.autoforwardportergit by default. This can be overridden by the AFPGCONFIG
environment variable if desired. For the purposes of this tutorial we will
assume that the default configuration directory is used.

Create the directory ~/.autoforwardportergit

Create ~/.autoforwardportergit/afpg.ini with the following contents

    [main]
    #Path to the working repo
    workingrepo=~/autoforwardportergit/tutorial/workingrepo
    #Path to the git repos
    gitdir=~/autoforwardportergit/tutorial/workingrepo
    #Path to autoforwardportergit temporary directory. Only one instance should be
    #run with a given temporary directory at a time.
    tmp=~/autoforwardportergit/tutorial/tmp
    #output directory
    outputdir=~/autoforwardportergit/tutorial/output
    #Version marker strings for local versions and reverts
    localmarker=+test
    revertmarker=+zrvt
    #Whether to invoke sbuild and dgit push
    dosbuild=no
    dodgitpush=no
    #Names and emails used in git config
    importname=Autoforwardporter demo git importer
    importemail=fake@fake
    forwardportname=Autoforwardporter demo
    forwardportemail=fake@fake
    #netrc file used by pushtogithub
    #netrcfile=~/github-login
    #arguments for dscdirtogit
    dscdirtogitargs=--snapshot
    #section named for main suite of group.
    [sid]
    #names of the suites
    stagingsuite=sid
    upstreamsuite=sid-deb
    #name of the "working" branch in the git repos.
    workingbranch=sid-working

Create ~/.autoforwardportergit/whitelist.sid with the following contents

    hello

Create empty files and directories

    touch ~/.autoforwardportergit/whitelist.import
    mkdir ~/autoforwardportergit/tutorial/tmp
    mkdir ~/autoforwardportergit/tutorial/git
    mkdir ~/autoforwardportergit/tutorial/output

## running the autoforwardporter

Run 

    ~/autoforwardportergit/masterdriver

The resulting package should now be able to be found in 
~/autoforwardportergit/tutorial/output 

