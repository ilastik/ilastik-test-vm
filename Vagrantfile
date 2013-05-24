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
mkdir virtualenv
mkdir virtualenv/python2.7
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
sudo apt-get install -y qt4-dev
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
echo 'source /home/vagrant/virtualenv/python2.7/bin/activate' >> /home/vagrant/.bashrc

echo "Building drtile"
sudo sh .travis_scripts/build_drtile.sh $VIRTUAL_ENV /home/vagrant/lazyflow

echo "Configuring lazyflow"
mkdir ~/.lazyflow
echo "[verbosity]" > ~/.lazyflow/config
echo "deprecation_warnings = false" >> ~/.lazyflow/config

echo "Generating test data"
python /home/vagrant/ilastik/tests/bin/generate_test_data.py /tmp/test_data

echo "PROVISION SCRIPT DONE"
ENDSCRIPT

Vagrant.configure("2") do |config|
  # All Vagrant configuration is done here. The most common configuration
  # options are documented and commented below. For a complete reference,
  # please see the online documentation at vagrantup.com.

  # Every Vagrant virtual environment requires a box to build off of.
  config.vm.box = "precise64"

  config.vm.provision :shell, :inline => $provision_script

  # Enable x11 forwarding (as if using ssh -X)
  config.ssh.forward_x11 = true

  # The url from where the 'config.vm.box' box will be fetched if it
  # doesn't already exist on the user's system.
  # config.vm.box_url = "http://domain.com/path/to/above.box"

  # Create a forwarded port mapping which allows access to a specific port
  # within the machine from a port on the host machine. In the example below,
  # accessing "localhost:8080" will access port 80 on the guest machine.
  # config.vm.network :forwarded_port, guest: 80, host: 8080

  # Create a private network, which allows host-only access to the machine
  # using a specific IP.
  config.vm.network :private_network, ip: "192.168.33.10"

  # Create a public network, which generally matched to bridged network.
  # Bridged networks make the machine appear as another physical device on
  # your network.
  # config.vm.network :public_network

  # Share an additional folder to the guest VM. The first argument is
  # the path on the host to the actual folder. The second argument is
  # the path on the guest to mount the folder. And the optional third
  # argument is a set of non-required options.
  # config.vm.synced_folder "../data", "/vagrant_data"

  # Provider-specific configuration so you can fine-tune various
  # backing providers for Vagrant. These expose provider-specific options.
  # Example for VirtualBox:
  #
  # config.vm.provider :virtualbox do |vb|
  #   # Don't boot with headless mode
  #   vb.gui = true
  #
  #   # Use VBoxManage to customize the VM. For example to change memory:
  #   vb.customize ["modifyvm", :id, "--memory", "1024"]
  # end
  #
  # View the documentation for the provider you're using for more
  # information on available options.

  config.vm.provider :virtualbox do |vb|
    # Don't boot with headless mode
    #vb.gui = true

     # Use VBoxManage to customize the VM. For example to change memory:
     vb.customize ["modifyvm", :id, "--memory", "2048"]
  end

  # Enable provisioning with Puppet stand alone.  Puppet manifests
  # are contained in a directory path relative to this Vagrantfile.
  # You will need to create the manifests directory and a manifest in
  # the file base.pp in the manifests_path directory.
  #
  # An example Puppet manifest to provision the message of the day:
  #
  # # group { "puppet":
  # #   ensure => "present",
  # # }
  # #
  # # File { owner => 0, group => 0, mode => 0644 }
  # #
  # # file { '/etc/motd':
  # #   content => "Welcome to your Vagrant-built virtual machine!
  # #               Managed by Puppet.\n"
  # # }
  #
  # config.vm.provision :puppet do |puppet|
  #   puppet.manifests_path = "manifests"
  #   puppet.manifest_file  = "init.pp"
  # end

  # Enable provisioning with chef solo, specifying a cookbooks path, roles
  # path, and data_bags path (all relative to this Vagrantfile), and adding
  # some recipes and/or roles.
  #
  # config.vm.provision :chef_solo do |chef|
  #   chef.cookbooks_path = "../my-recipes/cookbooks"
  #   chef.roles_path = "../my-recipes/roles"
  #   chef.data_bags_path = "../my-recipes/data_bags"
  #   chef.add_recipe "mysql"
  #   chef.add_role "web"
  #
  #   # You may also specify custom JSON attributes:
  #   chef.json = { :mysql_password => "foo" }
  # end

  # Enable provisioning with chef server, specifying the chef server URL,
  # and the path to the validation key (relative to this Vagrantfile).
  #
  # The Opscode Platform uses HTTPS. Substitute your organization for
  # ORGNAME in the URL and validation key.
  #
  # If you have your own Chef Server, use the appropriate URL, which may be
  # HTTP instead of HTTPS depending on your configuration. Also change the
  # validation key to validation.pem.
  #
  # config.vm.provision :chef_client do |chef|
  #   chef.chef_server_url = "https://api.opscode.com/organizations/ORGNAME"
  #   chef.validation_key_path = "ORGNAME-validator.pem"
  # end
  #
  # If you're using the Opscode platform, your validator client is
  # ORGNAME-validator, replacing ORGNAME with your organization name.
  #
  # If you have your own Chef Server, the default validation client name is
  # chef-validator, unless you changed the configuration.
  #
  #   chef.validation_client_name = "ORGNAME-validator"
end
