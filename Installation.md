# Installation of Lea package #

## Requirements ##
Lea requires Python 2.5 or higher, including Python 3.x.
There are no OS dependencies, so it shall work on Linux, Mac OS X, Windows, etc.

## Installation procedure ##

There are several methods to install the Lea on your system. These are fairly simple and standard. The Lea package is available in the [PyPI](https://pypi.python.org/pypi) Python package index:

[PyPI Lea page](http://pypi.python.org/pypi/lea)

Lea could then be downloaded and installed with just one command, provided that _easy\_install_ or_pip_ tool is installed.

### If _easy\_install_ is installed on your system ###

  1. open a terminal (shell)
  1. if needed, logon as root user or starts the following command with _sudo_
  1. type
```
% easy_install -U lea
```

### If _pip_ is installed on your system ###

  1. open a terminal (shell)
  1. if needed, logon as root user or starts the following command with _sudo_
  1. type
```
% pip install lea
```

If a previous version of Lea is already installed, it can be upgraded with
```
% pip install --upgrade lea
```

### Otherwise ###

  1. go on [PyPI Lea page](http://pypi.python.org/pypi/lea)
  1. click on the button _Download lea-x.y.tar.gz_ and follow the instructions to save this file on your disk
  1. open a terminal (shell)
  1. if needed, logon as root user or starts the following command with _sudo_
  1. type
```
% python setup.py install
```

_For more information on PyPI package installation or for troubleshooting, see [Installing Distributions from the Python Package Index](http://wiki.python.org/moin/CheeseShopTutorial#Installing_Distributions_from_the_Python_Package_Index)._

## Installation verification ##

To verify that Lea is correctly installed, whatever the installation method,

  1. open a terminal
  1. start python interpreter
```
% python
```
  1. type the following:
```
>>> from lea import *
```

If no error message is displayed, then the Lea package is correctly installed on your system. You can type your own Lea commands or follow the examples given in the [tutorial](LeaPyTutorial.md).