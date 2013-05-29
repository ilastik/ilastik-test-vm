ilastik Testing VM Environment
==============================

The ilastik testing VM is a [vagrant](http://www.vagrantup.com)-provisioned [VirtualBox](http://www.virtualbox.org) VM image.
It is designed to mimic* the VM images used by Travis-CI, so tests developed on the ilastik testing VM (gui tests in particular) 
can be incorporated into ilastik's regression test suite, which is executed on Travis-CI. 

*Note: Ideally, we would simply download the same VM image that Travis-CI uses to run our tests, but that apparently isn't available for download.
Instead, we create our own "good-enough" facsimile.

Prerequisites
=============

- Install [VirtualBox](http://www.virtualbox.org/wiki/Downloads)
- Install [vagrant](http://downloads.vagrantup.com/) (You might be able to install vagrant via a package manager like `apt-get`, but make sure you get at least v1.1)
- You need an X11 server.
 * On Linux, this is built-in.
 * On Mac, the X11.app that ships with OS X will suffice.
 * On Windows, the process is [a little more complicated](https://cc.jlab.org/windows/X11onWindows).

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
After the download is complete, it will be booted and "provisioned" with the ilastik testing setup according to the settings in the provided `Vagrantfile`.
This is a lengthy process, as it involves installing several software packages, including numpy, h5py, pyqt, vigra, etc.

Connect to the VM:
------------------

    vagrant ssh

You are now logged in to the VM (user/pass: vagrant/vagrant).  Our `Vagrantfile` enabled X11 forwarding (`ssh -X`), so you should be able to launch X11-based GUI applications.
Additionally, the `.bashrc` file activates the appropriate `virtualenv` and configures `$PYTHONPATH` for running ilastik.

You should be able to run the following command immediately after connecting:

    ./ilastik/ilastik.py

Troubleshooting
---------------

If that fails, you may need to enable X11 forwarding on your host OS.  For example, on Ubuntu, check the settings in `/etc/ssh/ssh_config`.
If X11 forwarding is enabled, you should be able to launch simple X11 apps.

For example:

    sudo apt-get install x11-apps   # password: vagrant
    xclock

Record a new test
=================

The primary benefit of running ilastik from a virtual machine is that you can create GUI-based test cases to share with other developers.
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

Note: the `vagrant up` command always *re-provisions* the VM on bootup.  This means it will re-run the script that installs and configures your environment.
But since you've already provisioned it once, this won't take as long as it did the first time (it won't need to reinstall qt, numpy, etc.).

To ensure that you are using the latest version of ilastik, lazyflow, and volumina without power-cycling the VM, you can re-provision it with

    vagrant provision

Note: `vagrant provision` (and therefore also `vagrant up`) will wipe away your VM's ilastik, lazyflow, and volumina repos.
Be sure to push any changes (e.g. new test cases) before reprovisioning.    

Finally, to delete your VM entirely and start from scratch, type

    vagrant destroy

Travis-CI compatibility
=======================

The ilastik-test-vm is designed mimic the environment provided by Travis-CI.
If your test involves hard-coded paths to files in `/home/vagrant`, use the symlink `/home/travis` instead.

