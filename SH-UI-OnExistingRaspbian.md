# Installation of Spectrecoin-sh-ui on existing Raspbian system

## Install dependencies:
```
sudo apt-get update && sudo apt-get upgrade
sudo apt-get install -y --no-install-recommends \
    bc \
    ca-certificates \
	dialog \
	git \
    mc \
    libboost-chrono1.62.0 \
    libboost-filesystem1.62.0 \
    libboost-program-options1.62.0 \
    libboost-thread1.62.0 \
    libevent-2.0-5 \
    libtool \
    libseccomp2 \
    wget
```

## Go into home directory and clone Spectrecoin-sh-ui Git repository:
```
cd
git clone https://github.com/spectrecoin/spectre-rpc-sh-ui --branch improvements spectrecoin-rpc-sh-ui
```

## Define aliases for ui-start and daemon handling:
```
echo "alias ui='/home/$(whoami)/spectrecoin-rpc-sh-ui/spectrecoin_rpc_ui.sh'" >> /home/$(whoami)/.bash_aliases
echo "alias wallet-start='sudo service spectrecoind start'"                   >> /home/$(whoami)/.bash_aliases
echo "alias wallet-stop='sudo service spectrecoind stop'"                     >> /home/$(whoami)/.bash_aliases
echo "alias wallet-status='sudo service spectrecoind status'"                 >> /home/$(whoami)/.bash_aliases

```

## Create service spectrecoind
```
sudo wget https://raw.githubusercontent.com/spectrecoin/pi-gen/spectrecoin/stage3/01-install-spectrecoind/files/spectrecoind.sh -O /etc/init.d/spectrecoind
sudo wget https://raw.githubusercontent.com/spectrecoin/pi-gen/spectrecoin/stage3/01-install-spectrecoind/files/spectrecoin.conf -O usr/lib/tmpfiles.d/spectrecoin.conf
sudo systemctl enable spectrecoind
```
