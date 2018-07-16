#this generates a docker image of around 330mb vs 778mb for regular python
FROM python:3.6

RUN  apt-get update -y && \
     apt-get upgrade -y && \
     apt-get dist-upgrade -y && \
     apt-get -y autoremove && \
     apt-get clean

RUN apt-get install -y \
                    zip \
                    unzip \
                    libboost-all-dev \
                    cmake

# Install compiler and perl stuff
RUN apt-get install --yes build-essential \
                            gcc-multilib \
                            apt-utils \
                            zlib1g-dev \
                            wget

###############################################
############# Install PANDOC ##################
###############################################
RUN apt-get -yq update && apt-get install -y python-pip wget
RUN wget https://github.com/jgm/pandoc/releases/download/1.19.2.1/pandoc-1.19.2.1-1-amd64.deb && \
    dpkg -i pandoc-1.19.2.1-1-amd64.deb && \
    wget https://github.com/lierdakil/pandoc-crossref/releases/download/v0.2.6.0/linux-ghc8-pandoc-1-19.tar.gz -q -O - | tar xz && \
    mv pandoc-crossref /usr/bin/ && \
    pip install pandocfilters && \
    apt-get clean -y && \
    rm -rf pandoc-1.19.2.1-1-amd64.deb /var/lib/apt/lists/* /tmp/* /var/tmp/*

#####################################################
############# Install BIX PROGRAMS ##################
#####################################################

ADD . /bix
WORKDIR /bix

##############  Install BWA #############
############################################## 


# Get source code from git
RUN apt-get install --yes git
WORKDIR /tmp
RUN git clone https://github.com/lh3/bwa.git
WORKDIR /tmp/bwa
RUN git checkout v0.7.15

# Compile
RUN make
RUN cp -p bwa /usr/bin

# Cleanup
RUN rm -rf /tmp/bwa

##############  Install SAMTOOLS #############
############################################## 
ENV SAMTOOLS_INSTALL_DIR=/opt/samtools
WORKDIR /tmp


RUN wget https://github.com/samtools/samtools/releases/download/1.8/samtools-1.8.tar.bz2 && \
  tar --bzip2 -xf samtools-1.8.tar.bz2
WORKDIR /tmp/samtools-1.8
RUN ./configure --enable-plugins --prefix=$SAMTOOLS_INSTALL_DIR && \
  make all all-htslib && \
  make install install-htslib

WORKDIR /
RUN ln -s $SAMTOOLS_INSTALL_DIR/bin/samtools /usr/bin/samtools && \
  rm -rf /tmp/samtools-1.8

ENTRYPOINT ["/usr/bin/samtools"]

##############  Install BCFTools #############
##############################################
#Setup ENV variables
ENV BCFTOOLS_BIN="bcftools-1.3.tar.bz2" \
BCFTOOLS_PLUGINS="/usr/local/libexec/bcftools" \
BCFTOOLS_VERSION="1.3"

RUN curl -fsSL https://github.com/samtools/bcftools/releases/download/$BCFTOOLS_VERSION/$BCFTOOLS_BIN -o /opt/$BCFTOOLS_BIN \
&& tar xvjf /opt/$BCFTOOLS_BIN -C /opt/ \
&& cd /opt/bcftools-$BCFTOOLS_VERSION \
&& make \
&& make install \
&& rm -rf /opt/$BCFTOOLS_BIN /opt/bcftools-$BCFTOOLS_VERSION

##############  Install GOSSAMER #############
##############################################

ENV GOSSAMER_INSTALL_DIR=/opt/gossamer-master/build/

WORKDIR /opt
RUN wget https://github.com/data61/gossamer/archive/master.zip
RUN unzip master.zip

WORKDIR /opt/gossamer-master
RUN mkdir build

WORKDIR /opt/gossamer-master/build
RUN cmake ..
RUN make
RUN make test
RUN make install

RUN ln -s $GOSSAMER_INSTALL_DIR/src/xenome /usr/bin/xenome
RUN ln -s $GOSSAMER_INSTALL_DIR/src/gossple /usr/bin/gossple
RUN ln -s $GOSSAMER_INSTALL_DIR/src/goss /usr/bin/goss
RUN ln -s $GOSSAMER_INSTALL_DIR/src/electus /usr/bin/electus


#########################################################
##############  Install Anubis requirements #############
#########################################################

RUN pip install -r /bix/requirements/pipeline.txt
RUN pip install pytest mock



CMD ["/bin/bash"]
