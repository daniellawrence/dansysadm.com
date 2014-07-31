Title: How to install python packages on linux
Date: 2012-31-07 10:20
Category: how to
Tags: python


First thing you need to install is python & python-dev(el) packages.
python will already be installed, however the commands will ensure both python & dev packages are installed.

Debian/Ubuntu/Mint
    $ sudo apt-get install python python-dev

Centos/redhat/Amazon linux
    $ sudo yum install python python-devel

Once you have both python & python dev packages installed, its time to install pip.

Debian/Ubuntu/Mint
    $ sudo apt-get install python-pip

Centos/redhat/Amazon linux
    $ sudo yum install python-pip


Now you have python and python-dev, lets install a simple package.


    $ sudo pip install -U virtualenv

This will install the virtualenv python package into you system.
You can check if it worked via

    $ pip freeze |grep virtualenv
