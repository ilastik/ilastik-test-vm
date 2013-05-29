# -*- mode: ruby -*-
# vi: set ft=ruby :

# The travis VM image has a user named 'travis'
# and has some python virtualenvs installed.
# We need to create a similar environment.
$provision_script = <<ENDSCRIPT
echo "PROVISION SCRIPT STARTING"

# To allow us to use the same paths as travis, make a link to /home/travis
sudo ln -s /home/vagrant /home/travis

sudo apt-get update

sudo apt-get install -y make
sudo apt-get install -y cmake
sudo apt-get install -y git
sudo apt-get install -y mercurial
sudo apt-get install -y xvfb

# Create xvfb launch script (copied from the Travis 32-bit VM)
cat <<EOF > /etc/init.d/xvfb
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
EOF

# Setup python virtualenv
sudo apt-get install -y python-virtualenv
mkdir -p virtualenv
mkdir -p virtualenv/python2.7
virtualenv virtualenv/python2.7

# Activate the virtualenv
source virtualenv/python2.7/bin/activate

echo "Using python: " `which python` "and pip: " `which pip`

# Clone latest ilastik and cd into it.
# All subsequent steps are performed WITHIN ilastik dir.
rm -rf /home/travis/ilastik 2> /dev/null
git clone http://github.com/ilastik/ilastik /home/travis/ilastik
chown -R vagrant /home/travis/ilastik
cd /home/travis/ilastik

# Install dependencies
sudo apt-get install -y libboost-python-dev
sudo apt-get install -y libjpeg-dev
sudo apt-get install -y libtiff4-dev
sudo apt-get install -y libpng12-dev
sudo apt-get install -y libfftw3-dev
sudo apt-get install -y libhdf5-serial-dev
sudo apt-get install -y libqt4-dev
sudo apt-get install -y libicu48
sudo apt-get install -y python-qt4 python-qt4-dev python-sip python-sip-dev
ln -s /usr/lib/python2.7/dist-packages/PyQt4/ $VIRTUAL_ENV/lib/python2.7/site-packages/
ln -s /usr/lib/python2.7/dist-packages/sip.so $VIRTUAL_ENV/lib/python2.7/site-packages/
ln -s /usr/lib/python2.7/dist-packages/sipdistutils.py $VIRTUAL_ENV/lib/python2.7/site-packages/
ln -s /usr/lib/python2.7/dist-packages/sipconfig.py $VIRTUAL_ENV/lib/python2.7/site-packages/
ln -s /usr/lib/python2.7/dist-packages/sipconfig_nd.py $VIRTUAL_ENV/lib/python2.7/site-packages/

echo "Installing development stage 1"
pip install -r requirements/development-stage1.txt --use-mirrors

echo "Installing development stage 2"
pip install -r requirements/development-stage2.txt --use-mirrors

echo "Installing VIGRA"
sudo sh .travis_scripts/install_vigra.sh $VIRTUAL_ENV

echo "Cloning volumina/lazyflow"
rm -rf /home/vagrant/volumina /home/vagrant/lazyflow 2> /dev/null
git clone http://github.com/ilastik/volumina /home/vagrant/volumina
git clone http://github.com/ilastik/lazyflow /home/vagrant/lazyflow
chown -R vagrant /home/vagrant/volumina
chown -R vagrant /home/vagrant/lazyflow

# Set up python on login
echo 'export PYTHONPATH=/home/vagrant/lazyflow:/home/vagrant/volumina:$PYTHONPATH' >> /home/vagrant/.bashrc
echo 'export LD_LIBRARY_PATH=/usr/local/lib' >> /home/vagrant/.bashrc
echo 'source /home/vagrant/virtualenv/python2.7/bin/activate' >> /home/vagrant/.bashrc

echo "Building drtile"
sudo sh .travis_scripts/build_drtile.sh $VIRTUAL_ENV /home/vagrant/lazyflow

echo "Building drtile"
echo "Configuring lazyflow"
mkdir -p ~/.lazyflow
echo "[verbosity]" > ~/.lazyflow/config
echo "deprecation_warnings = false" >> ~/.lazyflow/config

echo "Downloading real-world test data"
git clone http://github.com/ilastik/ilastik_testdata /tmp/real_test_data

# Needed to use vigra in the python script(s) below
export LD_LIBRARY_PATH=/usr/local/lib

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

  # Create a private network, which allows host-only access to the machine
  # using a specific IP.
  # config.vm.network :private_network, ip: "192.168.33.10"

  # Create a public network, which generally matched to bridged network.
  # Bridged networks make the machine appear as another physical device on
  # your network.
  # config.vm.network :public_network

end
