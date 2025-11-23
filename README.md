# install-openstack-on-ubuntu
install-openstack-on-ubuntu

# update
sudo apt update && sudo apt upgrade -y

# install basics
sudo apt install -y git vim net-tools sudo curl

# (optional but useful) install python3 and pip for other tools
sudo apt install -y python3 python3-pip

# run the helper (run as root)
sudo /usr/src/devstack/tools/create-stack-user.sh

sudo su - stack
# prompt should change to: stack@yourhost:~$

# in the stack user's home
cd ~
git clone https://opendev.org/openstack/devstack.git
cd devstack

# Create local.conf (minimal recommended config)
###################################################
# get primary IP (use this or replace manually)
HOST_IP=$(hostname -I | awk '{print $1}')

cat > local.conf <<EOF
[[local|localrc]]
# passwords (simple example; change for production)
ADMIN_PASSWORD=Passw0rd!
DATABASE_PASSWORD=\$ADMIN_PASSWORD
RABBIT_PASSWORD=\$ADMIN_PASSWORD
SERVICE_PASSWORD=\$ADMIN_PASSWORD

# set the host IP (important)
HOST_IP=$HOST_IP

# enable services (minimal)
ENABLED_SERVICES=g-api,g-reg,placement-api,n-api,n-cpu,n-cond,n-cauth,horizon,mysql,rabbit

# disable telemetry if you don't need it (optional)
# disable_service ceilometer gnocchi

# avoid creating swap files, etc. (optional)
# disable_service n-sch
EOF
################################################

# ensure script has execute permission
chmod +x stack.sh

# start installation (runs as stack user)
./stack.sh
# It should take couple of minutes and installation should complete

# After successful run — basic checks & access
# openrc for admin will be created, often as ~/devstack/openrc or /opt/stack/... (stack.sh prints the exact path)
# Common:
source ~/devstack/openrc

# check services
openstack service list
openstack image list

# Horizon dashboard URL printed at the end of stack.sh, typically:
# http://<HOST_IP>/dashboard
# Login: admin  Password: value you set in local.conf (ADMIN_PASSWORD)

# stop/unstack the devstack services
./unstack.sh

# clean leftover resources (destroys the stack environment)
./clean.sh


