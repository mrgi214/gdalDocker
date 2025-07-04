FROM openjdk:17

ENV GDAL_VERSION=3.1.2 
ENV OPENJPEG_VERSION=2.3.0 
ENV LIBPROJ_VERSION=7.1.0 
ENV CORES=3 
ENV VARIANT=slim

# Load assets
WORKDIR $ROOTDIR/
RUN mkdir -p $ROOTDIR/src

RUN wget -qO- \
    https://download.osgeo.org/gdal/${GDAL_VERSION}/gdal-${GDAL_VERSION}.tar.gz | \
    tar -xzC $ROOTDIR/src/
RUN wget -qO- \
    https://github.com/uclouvain/openjpeg/archive/v${OPENJPEG_VERSION}.tar.gz | \
    tar -xzC $ROOTDIR/src/
RUN wget -qO- \
    https://download.osgeo.org/proj/proj-${LIBPROJ_VERSION}.tar.gz | \
    tar -xzC $ROOTDIR/src/

RUN set -ex \
    # Runtime dependencies
    && deps=" \
       python-dev \
       python3-dev \
       python-numpy \
       python3-numpy \
       bash-completion \
       libspatialite-dev \
       libpq-dev \
       libcurl4-gnutls-dev \
       libxml2-dev \
       libgeos-dev \
       libnetcdf-dev \
       libpoppler-dev \
       libhdf4-alt-dev \
       libhdf5-serial-dev \
       libpoppler-private-dev \
       sqlite3 \
       libsqlite3-dev \
       libtiff-dev \
    " \
    # Build dependencies
    && buildDeps=" \
       build-essential \
       cmake \
       swig \
       ant \
       pkg-config \
    "\
    && apt-get update -y && apt-get install -y $buildDeps $deps --no-install-recommends \
    # Compile and install OpenJPEG
    && cd src/openjpeg-${OPENJPEG_VERSION} \
    && mkdir build && cd build \
    && cmake .. -DCMAKE_BUILD_TYPE=Release -DCMAKE_INSTALL_PREFIX=$ROOTDIR \
    && make -j${CORES} && make -j${CORES} install && make -j${CORES} clean \
    && cd $ROOTDIR && rm -Rf src/openjpeg* \
    # Compile and install PROJ
    && cd src/proj-${LIBPROJ_VERSION} && ./configure && make -j${CORES} && make install \
    && cd $ROOTDIR && rm -Rf src/proj* \
    # Compile and install GDAL
    && cd src/gdal-${GDAL_VERSION} \
    && ./configure --with-python --with-spatialite --with-pg --with-curl --with-java=$JAVA_HOME \
                   --with-poppler --with-openjpeg=$ROOTDIR \
    && make -j${CORES} && make -j${CORES} install && ldconfig \
    # Compile Python and Java bindings for GDAL
    && cd $ROOTDIR/src/gdal-${GDAL_VERSION}/swig/java && make -j${CORES} && make -j${CORES} install \
    && cd $ROOTDIR/src/gdal-${GDAL_VERSION}/swig/python \
    && python3 setup.py build \
    && python3 setup.py install \
    && cd $ROOTDIR && rm -Rf src/gdal* \
    # Remove build dependencies
    && apt-get purge -y --auto-remove $buildDeps \
    && rm -rf /var/lib/apt/lists/*
