# Ubuntu

These are specific notes for installing PEcAn on Ubuntu (14.04) and will be referenced from the main [installing PEcAn](Installing-PEcAn) page. You will at least need to install the build environment and Postgres sections. If you want to access the database/PEcAn using a web browser you will need to install Apache. To access the database using the BETY interface, you will need to have Ruby installed.

This document also contains information on how to install the Rstudio server edition as well as any other packages that can be helpful.

## Install build environment

```bash
sudo -s

# point to latest R
echo "deb http://cran.rstudio.com/bin/linux/ubuntu `lsb_release -s -c`/" > /etc/apt/sources.list.d/R.list
apt-key adv --keyserver keyserver.ubuntu.com --recv-keys E084DAB9

# update package list
apt-get -y update

# install packages needed for PEcAn
apt-get -y install build-essential gfortran git r-base-core jags liblapack-dev libnetcdf-dev netcdf-bin bc libcurl4-gnutls-dev curl udunits-bin libudunits2-dev libgmp-dev python-dev libgdal1-dev libproj-dev expect

# install packages needed for ED2
apt-get -y install openmpi-bin libopenmpi-dev

# install requirements for DALEC
apt-get -y install libgsl0-dev

# install packages for webserver
apt-get -y install apache2 libapache2-mod-php5 php5

# install packages to compile docs
apt-get -y install texinfo texlive-latex-base texlive-latex-extra texlive-fonts-recommended

# install devtools
echo 'install.packages("devtools", repos="http://cran.rstudio.com/")' | R --vanilla

# done as root
exit
```

## Install Postgres

Documentation: http://trac.osgeo.org/postgis/wiki/UsersWikiPostGIS21UbuntuPGSQL93Apt

```bash
sudo -s

# point to latest PostgreSQL
echo "deb http://apt.postgresql.org/pub/repos/apt `lsb_release -s -c`-pgdg main" > /etc/apt/sources.list.d/pgdg.list
wget --quiet -O - https://www.postgresql.org/media/keys/ACCC4CF8.asc | apt-key add -

# update package list
apt-get -y update

# install packages for postgresql (using a newer version than default)
apt-get -y install libdbd-pgsql postgresql postgresql-client libpq-dev postgresql-9.4-postgis-2.1 postgresql-9.4-postgis-2.1-scripts

# install following if you want to run pecan through the web
apt-get -y install php5-pgsql

# enable bety user to login with trust by adding the following lines after
# the ability of postgres user to login in /etc/postgresql/9.4/main/pg_hba.conf
local   all             bety                                    trust
host    all             bety            127.0.0.1/32            trust
host    all             bety            ::1/128                 trust

# Once done restart postgresql
/etc/init.d/postgresql restart

exit
```

To install the BETYdb database .. 
## Apache Configuration PEcAn

```bash
# become root
sudo -s

# get index page
rm /var/www/html/index.html
ln -s ${HOME}/pecan/documentation/index_vm.html /var/www/html/index.html

# setup a redirect
cat > /etc/apache2/conf-available/pecan.conf << EOF
Alias /pecan ${HOME}/pecan/web
<Directory ${HOME}/pecan/web>
  DirectoryIndex index.php
  Options +ExecCGI
  Require all granted
</Directory>
EOF
a2enconf pecan
/etc/init.d/apache2 restart

# done as root
exit
```

## Apache Configuration BETY

```bash
sudo -s

# install all ruby related packages
apt-get -y install ruby2.0 ruby2.0-dev libapache2-mod-passenger 

# link static content
ln -s ${HOME}/bety/public /var/www/html/bety

# setup a redirect
cat > /etc/apache2/conf-available/bety.conf << EOF
RailsEnv production
RailsBaseURI /bety
PassengerRuby /usr/bin/ruby2.0
<Directory /var/www/html/bety>
  Options +FollowSymLinks
  Require all granted
</Directory>
EOF
a2enconf bety
/etc/init.d/apache2 restart
```

## Rstudio-server

*NOTE This will allow anybody to login to the machine through the rstudio interface and run any arbitrary code. The login used however is the same as the system login/password.*

Based on version of ubuntu 32/64 use either of the following

*32bit only*
```bash
wget http://download2.rstudio.org/rstudio-server-0.98.1103-i386.deb
```

*64bit only*
```bash
wget http://download2.rstudio.org/rstudio-server-0.98.1103-amd64.deb
```

```bash
# bceome root
sudo -s

# install required packages
apt-get -y install libapparmor1 apparmor-utils libssl0.9.8

# install rstudio
dpkg -i rstudio-server-*
rm rstudio-server-*
echo "www-address=127.0.0.1" >> /etc/rstudio/rserver.conf
echo "r-libs-user=~/R/library" >> /etc/rstudio/rsession.conf
rstudio-server restart

# setup rstudio forwarding in apache
a2enmod proxy_http
cat > /etc/apache2/conf-available/rstudio.conf << EOF
ProxyPass        /rstudio/ http://localhost:8787/
ProxyPassReverse /rstudio/ http://localhost:8787/
RedirectMatch permanent ^/rstudio$ /rstudio/
EOF
a2enconf rstudio
/etc/init.d/apache2 restart

# all done, exit root
exit
```

## Additional packages

HDF5 Tools, netcdf, GDB and emacs
```bash
sudo apt-get -y install hdf5-tools cdo nco netcdf-bin ncview gdb emacs ess nedit
```