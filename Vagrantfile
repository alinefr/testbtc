# -*- mode: ruby -*-
# vi: set ft=ruby :

$script = <<SCRIPT
PACKAGES=(build-essential autoconf autogen libtool libssl-dev libevent-dev pkg-config libdb4.8++-dev libboost1.58-all-dev)
apt-add-repository -y ppa:bitcoin/bitcoin \
  && apt-get update \
  && apt-get -y upgrade 
for pkg in ${PACKAGES[@]}; do
  dpkg -s $pkg >/dev/null 2>&1 || apt-get -y install $pkg
done

(test -b /dev/sdc1 && test -b /dev/sdc1) || echo -e "\nn\np\n1\n\n\nw" | fdisk /dev/sdc
if [ -b /dev/sdc1 ]; then
  mountpoint /btc >/dev/null 2>&1 || mkfs.ext4 -F /dev/sdc1
fi
test -d /btc || mkdir -p /btc
mountpoint /btc >/dev/null 2>&1 || mount /dev/sdc1 /btc
if [[ $(stat -c "%U %G" /btc) != "vagrant vagrant" ]]; then
  chown -R vagrant:vagrant /btc
fi
SCRIPT

$nonroot = <<NONROOT
if [ ! -x /usr/local/bin/bitcoind ]; then
  if [ -d /btc ]; then
    test -d /btc/bitcoin/.git || git clone https://github.com/bitcoin/bitcoin.git /btc/bitcoin
    pushd /btc/bitcoin \
    && git checkout v0.15.1 \
    && ./autogen.sh \
    && ./configure \
    && make
  fi
fi
NONROOT

$btcinstall = <<BTC
if [ ! -x /usr/local/bin/bitcoind ]; then
  test -d /btc/bitcoin/.git && \
    pushd /btc/bitcoin && \
    make install
fi

if [ ! -f /lib/systemd/system/bitcoind.service ]; then
  cp /vagrant/bitcoind.service /lib/systemd/system/bitcoind.service
  systemctl daemon-reload
  systemctl enable bitcoind.service
  systemctl start bitcoind.service
fi
BTC

Vagrant.configure('2') do |config|
  config.vm.box = 'azure'

  config.ssh.private_key_path = '~/.ssh/id_rsa'
  config.vm.provider :azure do |azure, override|

    azure.tenant_id = ENV['AZURE_TENANT_ID']
    azure.client_id = ENV['AZURE_CLIENT_ID']
    azure.client_secret = ENV['AZURE_CLIENT_SECRET']
    azure.subscription_id = ENV['AZURE_SUBSCRIPTION_ID']

    azure.vm_name = 'testbtc'
    azure.resource_group_name = 'testbtc'
    azure.location = 'eastus'
    azure.vm_size = 'Standard_DS2_v2'
    azure.vm_image_urn = 'Canonical:UbuntuServer:16.04-LTS:latest'
    azure.data_disks = [
      {
        name: 'mydatadisk1',
        size_gb: 500
      }
    ]
  end
  config.vm.provision 'shell', inline: $script
  config.vm.provision 'shell', privileged: false, inline: $nonroot
  config.vm.provision 'shell', inline: $btcinstall
end
