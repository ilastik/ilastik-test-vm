==============================
ilastik Testing VM Environment
==============================

The ilastik testing VM is a [vagrant][]-provisioned [VirtualBox][] VM image.
It is designed to provide a reproducible test environment for continuous integration testing.
Furthermore, it provides a consistent environment for running *recorded* GUI regression tests.
(See below.)

Prerequisites
=============

- Install [VirtualBox][]
- Install [vagrant][] (You might be able to install vagrant via a package manager like `apt-get`, but make sure you get at least v1.4)
- You need an X11 server.
 * On Linux, this is built-in.
 * On Mac, the X11.app that ships with OS X will suffice.
 * On Windows, the process is [a little more complicated](https://cc.jlab.org/windows/X11onWindows).

[vagrant]: http://www.vagrantup.com
[VirtualBox]: http://www.virtualbox.org

Getting started
===============

Download the repo:
------------------

    git clone http://github.com/ilastik/ilastik-test-vm

Start the VM:
-------------

    cd ilastik-test-vm
    vagrant up

Running `vagrant up` for the first time will trigger the download of a bare-bones Ubuntu-12.04 `.box` file (300 MB or so).
After the download is complete, it will be booted and "provisioned".  The provisioning process does the following:

* Uses `apt-get` to install build tools like cmake, git, etc.
* Uses `apt-get` to install ilastik dependencies that BuildEM doesn't provide
* Uses `apt-get` to install java (to allow the VM to be operated as a [Hudson][] slave if desired)
* Installs a script to allow `xvfb` for GUI testing in a virtual frame buffer
* Clones the [ilastik-build-Linux][] repo in `/home/vagrant/ilastik-build`
* Clones the [ilastik_testdata][] repo, which can be used during regression tests
* Writes, but does not execute, a script to build ilastik and all dependencies (`/home/vagrant/build_ilastik.sh`)
* Writes, but does not execute, a script to run all tests in an ilastik build (including lazyflow and volumina unit tests)
* Enables port-forwarding for ssh from host port 8022 to VM port 22
* Enables X11 forwarding over ssh (like ssh -X)
* Specifies how many CPUs to allocate to the VM (4 by default)

After the first boot, the provision script will not be auto-executed during the `vagrant up` process.
If you make changes to it, be sure to run it manually by running `vagrant provision` from the host machine.

[ilastik-build-Linux]: http://github.com/ilastik/ilastik-build-Linux
[ilastik_testdata]: http://github.com/ilastik/ilastik_testdata
[Hudson]: http://hudson-ci.org

Connect to the VM:
------------------

    vagrant ssh

You are now logged in to the VM (user/pass: vagrant/vagrant).  

Troubleshooting
---------------

If that fails, you may need to enable X11 forwarding on your host OS.  For example, on Ubuntu, check the settings in `/etc/ssh/ssh_config`.
If X11 forwarding is enabled, you should be able to launch simple X11 apps from the VM.

For example:

    sudo apt-get install x11-apps   # password: vagrant
    xclock

Build ilastik
-------------

The script `/home/vagrant/build_ilastik.sh` is provided to checkout BuildEM and build ilastik and all its dependencies.
The build is executed in parallel using `make -j4`, but it will take a long time to complete.

**Note:**  Sometimes the build will fail due to a bad tarball download. (The md5 signature is checked for each dependency.)
If you see the build stop for any reason, just resume it by re-executing `build_ilastik.sh`.

Run ilastik
-----------

Once you've completed the full BuildEM build, ilastik can be executed from the VM:

    # Log in to the VM
    vagrant ssh
    
    # Activate the BuildEM binaries
    source ilastik-build/bin/setenv_ilastik_gui.sh
    
    # Run ilastik
    cd ilastik-build/src/ilastik
    ./ilastik/ilastik.py    

Run the unit/regression tests
-----------------------------

    vagrant ssh
    bash run_all_ilastik_tests.sh

Recorded GUI regression tests
=============================

One benefit of running ilastik from a virtual machine is that you can create GUI-based test cases to *share* with other developers.

Record a new test
-----------------

To record an ilastik gui test case, use the `--start_recording` option:

    ./ilastik/ilastik.py --start_recording

Add comments to the test as you go, so it's easy to see what you were doing if the test case starts to fail in the future.
For test data to use with your test case, look in the VM's `/tmp/test_data` directory.
When you are finished, save your test somewhere in the `ilastik/tests/recorded_test_cases` directory.

To play back your test case, use the `--playback_script` parameter:

    ./ilastik/ilastik.py --playback_script=ilastik/tests/recorded_test_cases/my_recording.py

To include your new test case in the ilastik regression test suite, be sure to `commit` and `push` it back up to the main ilastik repo!
If your test case exhibits a bug in ilastik that you want to share with the development team, you can open an issue in the ilastik [github issue tracker](http://github.com/ilastik/ilastik/issues).
Instead of pasting your recording directly into a github issue, use [pastebin.com] or [gist.github.com] and link to your recording in the issue text.

Recording tips
--------------

Explain what you're doing using using the comments box in the recorder UI.
It will be much easier to see what's going wrong when your test case fails some day.

Don't interfere with tests during playback.  Stray mouse movements, etc. can still affect the app during playback.

Hopefully all of our X11 servers use the same font settings by default.  If your X11 server uses a different font size, 
then the test cases you get from other people may not pass on your machine.  (Some mouse clicks will miss their targets.)

Avoid relying on details that may not be consistent between test runs.
For example, it is usually best to select your test data by typing the path into the file browser dialog (don't use the mouse).
If the playback environment's file browser doesn't start in the same directory you started in when you recorded the test, 
then your mouse clicks won't mean the same thing!

The recording system relies on QObject.objectName() to locate widgets within the widget hierarchy.
For the most part, you don't need to know or worry about the details here.
However, you must make sure that any top-level widgets you create (e.g. custom dialogs) have unique names.
If you fail to do this, you may have trouble playing back test cases involving your dialog.

For example:

    class MyDialog(QDialog):
        def __init__(parent):
            QDialog.__init__(parent)
            self.setObjectName("MyUniqueName")

Suspending the VM
=================

To pause the virtual machine, log out of your ssh session and type

    vagrant suspend

To bring the VM back to life,

    vagrant resume

To shut down the VM, use

    vagrant halt

To start it up again, use

    vagrant up

To ensure that you are using the latest version of ilastik, lazyflow, and volumina without power-cycling the VM, you can re-provision it with

    vagrant provision

**Note:** When you *first create* your VM, `vagrant provision` checks out the build system and a default version of ilastik.
On subsequent calls to `vagrant provision`, ilastik is not updated.  To update the ilastik source manually, see the git repo 
(and submodules) in `/home/vagrant/ilastik-build/src/ilastik`.

Finally, to delete your VM entirely and start from scratch, type

    vagrant destroy
