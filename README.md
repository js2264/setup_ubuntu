# Setting up an Ubuntu machine from scratch

Setting up with following softwares: 

- Ubuntu: 22.04.2 (jammy)
- Python: 3.11.3
- Miniforge: 23.1.0
- R: 4.3.0
- Bioc: 3.17

```sh
sudo export PYTHON_VERSION=3.11.3
sudo export MINIFORGE_RELEASE=23.1.0
sudo export R_VERSION=4.3.0
```

A docker image can be used: `docker run -it ubuntu:22.04` to kickstart the 
setup. 

## Install required system libraries

```sh
sudo apt update
sudo apt install gdebi-core curl wget pkg-config -y
sudo apt install libc6 libicu66 libreadline8 -y
apt install -y \
    gcc g++ perl pkg-config \
    automake make cmake less vim nano fort77 \
    wget git curl bzip2 gfortran unzip ftp \
    libpng-dev libjpeg-dev libtiff-dev \
    texlive-latex-base default-jre build-essential \
    libbz2-dev liblzma-dev libtool \
    libxml2 libxml2-dev \
    libzstd-dev \
    libdb-dev libglu1-mesa-dev zlib1g-dev \
    libncurses5-dev libghc-zlib-dev libncurses-dev \
    libpcre3-dev libxml2-dev libblas-dev libzmq3-dev \
    libreadline-dev libssl-dev libcurl4-openssl-dev \
    libx11-dev libxt-dev x11-common libcairo2-dev \
    libreadline-dev libgsl-dev libeigen3-dev libboost-all-dev \
    libgtk2.0-dev xvfb xauth xfonts-base apt-transport-https \
    libhdf5-dev libudunits2-dev libgdal-dev libgeos-dev \
    libproj-dev libnode-dev libnode-dev libmagick++-dev \
    libharfbuzz-dev libfribidi-dev
```

## Install python (system)

### Install python from source (custom location)

```sh
sudo export PYTHON_MAIN=$(echo $PYTHON_VERSION | sed 's,.[0-9]$,,')
sudo export PYTHON_BASE=$(echo $PYTHON_VERSION | sed 's,.[0-9].[0-9]$,,')

cd ~
wget https://www.python.org/ftp/python/${PYTHON_VERSION}/Python-${PYTHON_VERSION}.tgz
tar -xvf Python-${PYTHON_VERSION}.tgz
cd Python-${PYTHON_VERSION}/
sudo ./configure --prefix=/usr/local/lib/Python-${PYTHON_MAIN} --enable-optimizations
sudo make
sudo make install
sudo ln -s /usr/local/lib/Python-${PYTHON_MAIN}/bin/python${PYTHON_MAIN} /usr/bin/python-${PYTHON_MAIN}
sudo ln -s /usr/local/lib/Python-${PYTHON_MAIN}/bin/python${PYTHON_MAIN} /usr/bin/python
```

## Install R (system)

### Install R from binaries with APT

```sh
sudo curl -O https://cdn.rstudio.com/r/ubuntu-2204/pkgs/r-${R_VERSION}_1_amd64.deb
sudo gdebi r-${R_VERSION}_1_amd64.deb --non-interactive -q
sudo ln -s /opt/R/${R_VERSION}/bin/R /usr/local/bin/R
```

### Install R from source (custom location)

```sh
# sudo export R_VERSION=4.3.0
# sudo export R_MAIN=$(echo $R_VERSION | sed 's,.[0-9]$,,')
# sudo export R_BASE=$(echo $R_VERSION | sed 's,.[0-9].[0-9]$,,')
# 
# cd ~
# wget https://cran.rstudio.com/src/base/R-${R_BASE}/R-${R_VERSION}.tar.gz
# tar -xvf R-${R_VERSION}.tar.gz
# cd R-${R_VERSION}/
# sudo ./configure --prefix=/usr/local/lib/R-${R_MAIN} --enable-R-shlib --with-blas --with-lapack --with-readline --with-recommended-packages
# sudo make
# sudo make install
# sudo ln -s /usr/local/lib/R-${R_MAIN}/bin/R /usr/bin/R-${R_MAIN}
# sudo ln -s /usr/local/lib/R-${R_MAIN}/bin/R /usr/bin/R
```

## EXTRAS

### Install useful R packages

```r
repo <- sprintf(
    "https://r-lib.github.io/p/pak/stable/%s/%s/%s", 
    .Platform$pkgType, R.Version()$os, R.Version()$arch
)
install.packages("pak", repos = repo)

## -- Bioconductor
pak::pak('BiocManager')
BiocManager::install(version = BiocManager::version())

## -- General CRAN packages 
pak::pak('reticulate')
pak::pak('tidyverse')
pak::pak('devtools')
pak::pak('umap')
pak::pak('corrplot')
pak::pak('gam')
pak::pak('ggbeeswarm')
pak::pak('ggthemes')
pak::pak('Matrix')
pak::pak('zeallot')
pak::pak('fossil')
pak::pak('rgl')

## -- Bioc packages
pak::pak('plyranges')
pak::pak('org.Mm.eg.db')
pak::pak('org.Hs.eg.db')

## -- scRNAseq packages
pak::pak('SingleCellExperiment')
pak::pak('scran')
pak::pak('scater')
pak::pak('batchelor')
pak::pak('DropletUtils')
pak::pak('AUCell')
pak::pak('SingleR')
pak::pak('slingshot')
pak::pak('tradeSeq')
pak::pak('clusterExperiment')
pak::pak('CountClust')
pak::pak('velociraptor')
pak::pak('BUSpaRse')
pak::pak('satijalab/seurat')
pak::pak('TaddyLab/maptpx')
pak::pak('MacoskoLab/liger')
pak::pak('velocyto-team/velocyto.R')
pak::pak('theislab/destiny')
pak::pak('broadinstitute/inferCNV')

## -- HiC packages
pak::pak('HiCExperiment', dependencies = TRUE)
pak::pak('HiContacts', dependencies = TRUE)
pak::pak('HiCool', dependencies = TRUE)
pak::pak('fourDNData', dependencies = TRUE)
pak::pak('DNAZooData', dependencies = TRUE)
```

### Set up a `HiC` conda env

```sh
## Create `HiC` conda env. and add other dependencies
curl -L -O "https://github.com/conda-forge/miniforge/releases/${MINIFORGE_RELEASE}/download/Mambaforge-$(uname)-$(uname -m).sh"
bash Mambaforge-$(uname)-$(uname -m).sh -p /usr/local/lib/mambaforge -b
ln -s /usr/local/lib/mambaforge/bin/mamba /usr/bin/mamba
ln -s /usr/local/lib/mambaforge/bin/conda /usr/bin/conda
mamba init
bash
mamba create -n HiC -c bioconda -c conda-forge \
    'python>=3.7.12' 'hicstuff>=3.1.5' 'chromosight>=1.6.3' \
    'cooler>=0.9.1' 'cooltools>=0.5.1' \
    'samtools>=1.7' 'bowtie2>=2.4.5' 'bedtools>=2.30.0' 'deeptools>=3.5.1' \
    'matplotlib>=3.5.3' 'tree>=2.0.0' \
    macs2 mawk java-jdk ucsc-bedGraphToBigWig fastqc
mamba activate HiC
mamba install -c conda-forge python=3 umap-learn leidenalg -y
mamba install -c conda-forge \
    numpy \
    scipy \
    pandas \
    matplotlib \
    setuptools \
    umap-learn

cd /usr/local/lib/
git clone https://github.com/js2264/tinyMapper.git
ln -s /usr/local/lib/tinyMapper/tinyMapper.sh /usr/bin/tinyMapper.sh
ln -s /usr/local/lib/tinyMapper/tinyMapper.sh /usr/bin/tinyMapper
```

### Install cellranger

```sh
cd /opt/
sudo wget -O cellranger-6.0.1.tar.gz "https://cf.10xgenomics.com/releases/cell-exp/cellranger-6.0.1.tar.gz?Expires=1622001486&Policy=eyJTdGF0ZW1lbnQiOlt7IlJlc291cmNlIjoiaHR0cHM6Ly9jZi4xMHhnZW5vbWljcy5jb20vcmVsZWFzZXMvY2VsbC1leHAvY2VsbHJhbmdlci02LjAuMS50YXIuZ3oiLCJDb25kaXRpb24iOnsiRGF0ZUxlc3NUaGFuIjp7IkFXUzpFcG9jaFRpbWUiOjE2MjIwMDE0ODZ9fX1dfQ__&Signature=IWM4eRXo7G4YRXjOrcTFKscHWsa5sU6bqrIFrWOyzkV0pzOb0OPrjI7uMtoorqV6ivIjKbE21F9PkfajqV77hfODzCIyOmVQWxi9~nV2vZ6fdmo80hB4xmFQ7RdmLNAS~MDrhAg8etcQ-VkPS6wbzwtfNai-jDfRaas7DNPcq5CtFA4UBtfMG51XTpfPFKHDt66QWQOKShD5JNi05Cq4mDcWfJD1EFC-Z5b0Nid416NeLtxzUjNl043VRpWk2EibNGn8s8qGO0Kk~5Uh1l-qW~KkLSPVv5QFWj5wAgPjC3At2bCqjBaD6c87lIcHKx7WTPv46-d-gdQvYg-ZRcmBPQ__&Key-Pair-Id=APKAI7S6A5RYOXBWRPDA"
sudo tar -xzvf cellranger-6.0.1.tar.gz
sudo wget https://cf.10xgenomics.com/supp/cell-exp/refdata-gex-mm10-2020-A.tar.gz
sudo tar -xzvf refdata-gex-mm10-2020-A.tar.gz
```
