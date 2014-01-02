# -*- mode: ruby -*-
# vi: set ft=ruby :

$provision_script = <<ENDSCRIPT
echo "PROVISION SCRIPT STARTING (user="`whoami`", pwd="`pwd`")"

#
# the following section runs as user 'root'
#

apt-get update

# BuildEM dependencies
apt-get install -y make
apt-get install -y cmake
apt-get install -y git
apt-get install -y mercurial

# Java is needed so this VM can run as a hudson slave
apt-get install -y openjdk-7-jre

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

#
# change to vagrant user
#
su --login vagrant

# Set up the build directory, but don't build
mkdir -p ilastik-build
cd ilastik-build
BUILDEM_DIR=`pwd`
git clone https://github.com/ilastik/ilastik-build-Linux.git || (cd ilastik-build-Linux && git pull && cd -)
mkdir -p build
cd build

#
# Build script
#
cat <<END_BUILD_SCRIPT > build_ilastik.sh
#!/bin/bash
cd $BUILDEM_DIR/build
cmake ../ilastik-build-Linux -DBUILDEM_DIR=$BUILDEM_DIR
make
# make package
END_BUILD_SCRIPT

#
# Update script
#

#
# Test script: lazyflow
#
cat <<END_LAZYFLOW_TEST_SCRIPT > run_lazyflow_tests.sh
#!/bin/bash
echo TODO...
END_LAZYFLOW_TEST_SCRIPT


echo "Configuring lazyflow"
mkdir -p ~/.lazyflow
echo "[verbosity]" > ~/.lazyflow/config
echo "deprecation_warnings = false" >> ~/.lazyflow/config

echo "Downloading real-world test data"
git clone http://github.com/ilastik/ilastik_testdata /tmp/real_test_data || (cd /tmp/real_test_data && git pull && cd -)

echo "Generating synthetic test data"
python /home/vagrant/ilastik/tests/bin/generate_test_data.py /tmp/test_data

echo "PROVISION SCRIPT DONE"
ENDSCRIPT

Vagrant.configure("2") do |config|
  # Every Vagrant virtual environment requires a box to build off of.
  config.vm.box = "precise64"
  config.vm.provision :shell, :inline => $provision_script
  config.vm.box_url = "http://files.vagrantup.com/precise64.box"

  # Enable x11 forwarding (as if using ssh -X)
  config.ssh.forward_x11 = true

  # VirtualBox settings
  config.vm.provider :virtualbox do |vb|
    # Don't boot with headless mode
    #vb.gui = true

     # Use VBoxManage to customize the VM. For example to change memory:
     vb.customize ["modifyvm", :id, "--memory", "2048"]
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
  # config.vm.network :forwarded_port, guest: 22, host: 8022

  # Create a private network, which allows host-only access to the machine
  # using a specific IP.
  # config.vm.network :private_network, ip: "192.168.33.10"

  # Create a public network, which generally matched to bridged network.
  # Bridged networks make the machine appear as another physical device on
  # your network.
  # config.vm.network :public_network, :bridge => 'eth0'

end
