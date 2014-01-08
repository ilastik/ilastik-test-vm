# -*- mode: ruby -*-
# vi: set ft=ruby :

###################################################################
###################################################################
# Root provisioning
###################################################################
###################################################################
$root_provision_script = <<END_ROOT_PROVISIONING
echo "ROOT PROVISION SCRIPT STARTING (user="`whoami`", pwd="`pwd`")"

#
# the following section runs as user 'root'
#

apt-get update

# BuildEM dependencies
apt-get install -y make
apt-get install -y cmake
apt-get install -y git
apt-get install -y mercurial
apt-get install -y build-essential
apt-get install -y gfortran

# Dependencies of ilastik not included in BuildEM
# (See BuildEM readme.)
apt-get install -y libxext-dev
apt-get install -y libgl1-mesa-dev
apt-get install -y libxt-dev
apt-get install -y libxml2-dev

# Since we'll use X11 for this VM, we must install these packages before building Qt.
# See http://qt-project.org/doc/qt-4.8/requirements-x11.html
# Note: Qt will *build* if some of these are omitted,
#        but you will encounter bugs at runtime.
apt-get install -y libfontconfig1-dev
apt-get install -y libxfixes-dev
apt-get install -y libxrender-dev
apt-get install -y libxcursor-dev
apt-get install -y libxrandr-dev
apt-get install -y libxinerama-dev

# Hudson compatibility
# Java is needed so this VM can run as a hudson slave
apt-get install -y openjdk-7-jre

# Create a workspace for Hudson to use
# (In Hudson, provide this path as the node's "Remote FS root")
HUDSON_REMOTE_FS_ROOT=/var/hudson
mkdir -p $HUDSON_REMOTE_FS_ROOT
chmod 777 $HUDSON_REMOTE_FS_ROOT

# XVFB allows us to run GUI tests in a virtual screen
apt-get install -y xvfb

# Create xvfb launch script (copied from the Travis 32-bit VM)
cat <<END_XVFB_LAUNCH > /etc/init.d/xvfb
XVFB=/usr/bin/Xvfb
XVFBARGS=":99 -ac -screen 0 1024x768x16"
PIDFILE=/tmp/cucumber_xvfb_99.pid
case "\\$1" in
  start)
    echo -n "Starting virtual X frame buffer: Xvfb"
    /sbin/start-stop-daemon --start --quiet --pidfile \\$PIDFILE --make-pidfile --background --exec \\$XVFB -- \\$XVFBARGS
    echo "."
    ;;
  stop)
    echo -n "Stopping virtual X frame buffer: Xvfb"
    /sbin/start-stop-daemon --stop --quiet --pidfile \\$PIDFILE
    rm -f \\$PIDFILE
    echo "."
    ;;
  restart)
    \\$0 stop
    \\$0 start
    ;;
  *)
  echo "Usage: /etc/init.d/xvfb {start|stop|restart}"
  exit 1
esac
exit 0
END_XVFB_LAUNCH
echo "ROOT PROVISION SCRIPT DONE"
END_ROOT_PROVISIONING
###################################################################
###################################################################


###################################################################
###################################################################
# Non-root provisioning #
###################################################################
###################################################################
$nonroot_provision_script = <<END_NONROOT_PROVISIONING
echo "NONROOT PROVISION SCRIPT STARTING (user="`whoami`", pwd="`pwd`")"
# Set up the build directory, but don't build
cd /home/vagrant
mkdir -p ilastik-build
cd ilastik-build
BUILDEM_DIR=`pwd`
if [ ! -d "$BUILDEM_DIR/ilastik-build-Linux" ]; then
    git clone https://github.com/ilastik/ilastik-build-Linux.git
else
    cd ilastik-build-Linux && git pull && cd -
fi

mkdir -p build
cd /home/vagrant

echo "export BUILDEM_DIR=$BUILDEM_DIR" >> /home/vagrant/.bashrc

echo "Writing lazyflow config file"
mkdir -p ~/.lazyflow
echo "[verbosity]" > ~/.lazyflow/config
echo "deprecation_warnings = false" >> ~/.lazyflow/config

echo "Downloading real-world test data"
TEST_DATA_DIR=/tmp/real_test_data
if [ ! -d "$TEST_DATA_DIR" ]; then
    git clone http://github.com/ilastik/ilastik_testdata $TEST_DATA_DIR
else
    cd $TEST_DATA_DIR && git pull origin master && cd -
fi

################
# Build script #
################
cat <<END_BUILD_SCRIPT > build_ilastik.sh
#!/bin/bash
set -e
cd $BUILDEM_DIR/build
cmake ../ilastik-build-Linux -DBUILDEM_DIR=$BUILDEM_DIR -DILASTIK_VERSION=master
make -j4
# make package
END_BUILD_SCRIPT
################

#################
### All Tests ###
#################
cat <<END_TEST_SCRIPT > run_all_ilastik_tests.sh
#!/bin/bash

set -e # Exit on first failure.

if [[ "\\$1" == "--use-xvfb" ]]
then
    echo "Using X Virtual Frame Buffer for GUI tests."
    echo 'type "sh -e /etc/init.d/xvfb stop" to disable.'
    export DISPLAY=:99.0
    sh -e /etc/init.d/xvfb start
fi

# Set up env
export BUILDEM_DIR=$BUILDEM_DIR
source \\$BUILDEM_DIR/bin/setenv_ilastik_gui.sh
export PATH=\\$BUILDEM_DIR/bin:\\$PATH

# Update repo to latest checkpoint
# (This updates lazyflow, volumina, and ilastik)
cd \\$BUILDEM_DIR/src/ilastik
git pull origin master
git submodule update --init --recursive

# Run tests
echo "Running lazyflow tests...."
cd lazyflow/tests
nosetests .
cd -

echo "Running volumina tests...."
cd volumina/tests
nosetests .
cd -

echo "Generating synthetic test data"
python ilastik/tests/bin/generate_test_data.py /tmp/test_data

cd ilastik/tests
echo "Running ilastik unit tests"
./run_each_unit_test.sh
echo "Running ilastik recorded GUI tests"
./run_recorded_tests.sh
cd ../..
END_TEST_SCRIPT
#################

echo "NONROOT PROVISION SCRIPT DONE"
END_NONROOT_PROVISIONING
###################################################################
###################################################################

###################################################################
## Vagrant Config
###################################################################
Vagrant.configure("2") do |config|
  # Every Vagrant virtual environment requires a box to build off of.
  config.vm.box = "ilastikci"
  config.vm.provision :shell, :inline => $root_provision_script
  config.vm.provision :shell, :privileged => false, :inline => $nonroot_provision_script
  config.vm.box_url = "http://files.vagrantup.com/precise64.box"
  config.vm.hostname = "ilastikci"

  # Enable x11 forwarding (as if using ssh -X)
  config.ssh.forward_x11 = true

  # VirtualBox settings
  config.vm.provider :virtualbox do |vb|
    # Don't boot with headless mode
    #vb.gui = true

     # Use VBoxManage to customize the VM.
     # For more options, check the help message for the VBoxManage command
     vb.customize ["modifyvm", :id,
     			   "--memory", "2048",
     			   "--cpus", "4",
     			   "--cpuexecutioncap", "100" ]
  end

  # ADDITIONAL OPTIONS:

  # Share an additional folder to the guest VM. The first argument is
  # the path on the host to the actual folder. The second argument is
  # the path on the guest to mount the folder. And the optional third
  # argument is a set of non-required options.
  # config.vm.synced_folder "../data", "/vagrant_data"

  # Create a forwarded port mapping which allows access to a specific port
  # within the machine from a port on the host machine. In the example below,
  # accessing "localhost:8080" will access port 80 on the guest machine.
  # config.vm.network :forwarded_port, guest: 80, host: 8080
  config.vm.network :forwarded_port, guest: 22, host: 8022

  # Create a private network, which allows host-only access to the machine
  # using a specific IP.
  # config.vm.network :private_network, ip: "192.168.33.10"

  # Create a public network, which generally matched to bridged network.
  # Bridged networks make the machine appear as another physical device on
  # your network.
  # config.vm.network :public_network, :bridge => 'eth0'

end
