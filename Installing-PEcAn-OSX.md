# Mac OSX

These are specific notes for installing PEcAn on Mac OSX and will be referenced from the main [installing PEcAn](Installing-PEcAn) page. You will at least need to install the build environment and Postgres sections. If you want to access the database/PEcAn using a web browser you will need to install Apache. To access the database using the BETY interface, you will need to have Ruby installed.

This document also contains information on how to install the Rstudio server edition as well as any other packages that can be helpful.


## Install build environment

```bash
# install R
# download from http://cran.r-project.org/bin/macosx/

# install gfortran 
# download from http://hpc.sourceforge.net
sudo tar -xvf ~/Downloads/gcc-mlion.tar -C /

# install OpenMPI
curl -o openmpi-1.6.3.tar.gz http://www.open-mpi.org/software/ompi/v1.6/downloads/openmpi-1.6.3.tar.gz
tar zxf openmpi-1.6.3.tar.gz
cd openmpi-1.6.3
./configure --prefix=/usr/local
make all
sudo make install
cd ..

# install szip
curl -o szip-2.1-MacOSX-intel.tar.gz ftp://ftp.hdfgroup.org/lib-external/szip/2.1/bin/szip-2.1-MacOSX-intel.tar.gz
tar zxf szip-2.1-MacOSX-intel.tar.gz
sudo mv szip-2.1-MacOSX-intel /usr/local/szip

# install HDF5

curl -o hdf5-1.8.11.tar.gz http://www.hdfgroup.org/ftp/HDF5/current/src/hdf5-1.8.11.tar.gz
tar zxf hdf5-1.8.11.tar.gz
cd hdf5-1.8.11
sed -i -e 's/-O3/-O0/g' config/gnu-flags 
./configure --prefix=/usr/local/hdf5 --enable-fortran --enable-cxx --with-szlib=/usr/local/szip
make
# make check
sudo make install
# sudo make check-install
cd ..
```

## Install Postgres

For those on a Mac I use the following app for postgresql which has
postgis already installed (just make sure you have 9.3)
http://postgresapp.com/

To get postgis run the following commands in psql:

```bash
## Enable PostGIS (includes raster)
CREATE EXTENSION postgis;
## Enable Topology
CREATE EXTENSION postgis_topology;
## fuzzy matching needed for Tiger
CREATE EXTENSION fuzzystrmatch;
## Enable US Tiger Geocoder
CREATE EXTENSION postgis_tiger_geocoder;
```

To check your postgis run the following command again in psql: `SELECT PostGIS_full_version();`

## Additional installs


### Install JAGS

For more instructions see http://martynplummer.wordpress.com/2011/11/04/rjags-3-for-mac-os-x/


### Install udunits

Installing udunits-2 on MacOSX is done from source.

* download most recent [version of Udunits here](http://www.unidata.ucar.edu/downloads/udunits/index.jsp)
* instructions for [compiling from source](http://www.unidata.ucar.edu/software/udunits/udunits-2/udunits2.html#Obtain)


```bash
wget ftp://ftp.unidata.ucar.edu/pub/udunits/udunits-2.1.24.tar.gz
tar -xvf udunits-2.1.24.tar.gz
cd udunits-2.1.24
./configure; make; make check; make clean
```

## Apache Configuration

```bash
sudo sed -i '' 's/^#LoadModule php5_module/LoadModule php5_module/' /etc/apache2/httpd.conf

sudo mkdir /var/mysql
sudo ln -s /tmp/mysql.sock /var/mysql/mysql.sock

cat > /etc/apache2/users/pecan.conf << EOF
Alias /pecan ${PWD}/pecan/web
<Directory ${PWD}/pecan/web>
  DirectoryIndex index.php
  Options +All
  Order allow,deny
  Allow from all
</Directory>
EOF
```

## Ruby

For Ruby on the mac please use [JewelryBox](https://jewelrybox.unfiniti.com/).

## Rstudio Server

For the mac you can download [Rstudio Desktop](http://www.rstudio.com/).