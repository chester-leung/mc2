Installation
============
You can install MC\ :sup:`2` Client from source or choose to build a Docker image from a provided Dockerfile with MC\ :sup:`2` Client, Opaque SQL, and all dependencies pre-installed. Building the Docker image is much faster than building from source (takes about 7 min), but should be used for development/testing purposes only.


Building with Docker for a local deployment
-------------------------------------------
To quickly play with MC\ :sup:`2` Client and Opaque SQL locally, you can use the provided Dockerfile to build a container (takes ~7 min) with all MC\ :sup:`2` Client and Opaque SQL dependencies. To do so, you must have `Docker <https://docs.docker.com/get-docker/>`_ installed.

The container will have the contents of this ``mc2`` directory at ``/mc2/client``. Opaque SQL will be at ``/mc2/opaque-sql``

For ease of use, we recommend that you create a directory within your host ``mc2`` directory that will serve as your playground, and then mount your ``playground`` directory to the Docker container. Mounting will ensure that changes you make in your ``playground`` directory outside the container will be reflected inside the container, and vice versa. If you're bringing your own data, you can either copy your data over to your playground directory, or separately mount your data directory to the container.

.. code-block:: bash

    # Clone the `mc2` repo
    # Build a Docker image called `mc2_img`
    docker build -t mc2_img .

    # Run the container, mounting your playground to the container, and open a shell into the container
    docker run -it -v </absolute/path/to/mc2/playground>:/mc2/client/playground mc2_img /bin/bash

Building from Source
--------------------
To install MC\ :sup:`2` Client, you'll need to be running Ubuntu 18.04. Ubuntu 16.04 should also work but is untested.

MC\ :sup:`2` Client is written in both C++ and Python. As a result, we'll have to first build the C++ source, and then build and install the Python package.

1. Install dependencies.

.. code-block:: bash

    # CMake
    wget https://github.com/Kitware/CMake/releases/download/v3.15.6/cmake-3.15.6-Linux-x86_64.sh
    sudo bash cmake-3.15.6-Linux-x86_64.sh --skip-license --prefix=/usr/local

    # Open Enclave
    echo 'deb [arch=amd64] https://download.01.org/intel-sgx/sgx_repo/ubuntu bionic main' | sudo tee /etc/apt/sources.list.d/intel-sgx.list && \
    wget -qO - https://download.01.org/intel-sgx/sgx_repo/ubuntu/intel-sgx-deb.key | sudo apt-key add - && \
    echo "deb http://apt.llvm.org/bionic/ llvm-toolchain-bionic-7 main" | sudo tee /etc/apt/sources.list.d/llvm-toolchain-bionic-7.list && \
    wget -qO - https://apt.llvm.org/llvm-snapshot.gpg.key | sudo apt-key add - && \
    echo "deb [arch=amd64] https://packages.microsoft.com/ubuntu/18.04/prod bionic main" | sudo tee /etc/apt/sources.list.d/msprod.list && \
    wget -qO - https://packages.microsoft.com/keys/microsoft.asc | sudo apt-key add - && \
    sudo apt update
    sudo apt -y install clang-8 libssl-dev gdb libsgx-enclave-common libsgx-quote-ex libprotobuf10 libsgx-dcap-ql libsgx-dcap-ql-dev az-dcap-client open-enclave=0.12.0

    # Azure CLI
    curl -sL https://aka.ms/InstallAzureCLIDeb | sudo bash

    # Mbed TLS and Pip
    sudo apt-get install -y libmbedtls-dev python3-pip
    pip3 install --upgrade pip

    # MC2 Client Python package dependencies
    git clone https://github.com/mc2-project/mc2.git
    cd mc2
    pip3 install -r requirements.txt 
    cd ..

    # Opaque Systems `sequencefile` Python package
    git clone https://github.com/opaque-systems/sequencefile.git
    cd sequencefile
    sudo python3 setup.py install
    cd ..

2. Clone the MC\ :sup:`2` Client GitHub repo and build the C++ source.

.. code-block:: bash

    cd mc2/src
    mkdir build
    cd build
    cmake ..
    make -j4
    cd ../..

3. Once you've built the binary, install the Python package.

.. code-block:: bash

    cd python-package
    sudo python3 setup.py install


You're done! Try importing the ``mc2client`` Python package to check that your installation was successful.

.. code-block::

    $ python3
    Python 3.8.7 (default, Dec 30 2020, 10:13:08)
    [Clang 12.0.0 (clang-1200.0.32.28)] on darwin
    Type "help", "copyright", "credits" or "license" for more information.

    >>> import mc2client as mc2

Azure Login
-----------
If you want to manage your Azure resources using MC\ :sup:`2` Client, authenticate to Azure and set your subscription ID. Find your subscription ID by following `these instructions <https://docs.microsoft.com/en-us/azure/media-services/latest/how-to-set-azure-subscription?tabs=portal>`_.

.. code-block:: bash

    az login
    az account set -s <YOUR_SUBSCRIPTION_ID>
