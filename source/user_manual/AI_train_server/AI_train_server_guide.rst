===========================
AI Training Server Setup
===========================

Follow these steps to set up your AI Training Server:

1. Prepare a Ubuntu os system with nvidia gpu (recommend version: ubuntu-24.04)
2. Install nvidia driver

* Open Software & Updates
* Navigate to the Additional Drivers tab
* Select the driver labeled "proprietary, tested" (e.g., nvidia-driver-560)
* Click Apply Changes and reboot
* After rebooting, verify the installation by running nvidia-smi

.. figure:: ../../_static/user_manual/AI_train_server/ai_train_server_setup_2_1.png
   :align: center


3. Install docker

* please follow the instructions from: https://docs.docker.com/engine/install/ubuntu/
* add user into docker group and reboot

.. code-block:: bash

   sudo usermod -aG docker $USER

* Use docker ps to verify the installation

4. Create a folder to put the scripts and tar.gz files inside (ex. AI_train_server), the folder structure will be similar as follow

.. code-block:: bash

   AI_train_server/
   |-- docker_images/
   |   |-- IMAGES.txt --> docker images list
   |   |-- load_docker_images.sh  --> installation scripts
   |   |-- acuity_converter_v1.1.tar.gz  --> docker image file
   |   |-- training-server-train_latest.tar.gz  --> docker image file
   |   |-- training-server-importer_latest.tar.gz  --> docker image file
   |   |-- nvidia_cuda_12.1.1-cudnn8-runtime-ubuntu22.04.tar.gz  --> docker image file
   |-- base
   |-- base-20260109-165208.tar.gz
   |-- workspaces_example
   |-- workspaces-example-20251223-135111.tar.gz
   |-- INSTALLATION.md


5. Install the scripts

.. code-block:: bash

   tar -xzf base-<timestamp>.tar.gz
   tar -xzf workspaces-example-<timestamp>.tar.gz
   cd docker_images && ./load_docker_images.sh
   cd ../base
   sudo ./install.sh
   cd ../workspaces_example && ./install_workspaces_example.sh (optional, generate default example)

6. After installation, there will an shortcut on the desktop, or you can login by "http://localhost:8080/login".

.. figure:: ../../_static/user_manual/AI_train_server/ai_train_server_setup_6_1.jpg
   :align: center

   System login interface

7. On the model training interface, log in with the default username “admin@realtek.com” and password "admin123” After logging in, you can change the password if you want.


===========================
AI Training Server Run
===========================

1. **Log into the Server**

   Ensure you have access to the server and log in with the appropriate credentials.

2. **Start the Training**

   Once you have successfully logged into the server, you can upload your own dataset or download the example datasets from hugging face, user can also adjust the training configuration.

.. figure:: ../../_static/user_manual/AI_train_server/ai_train_server_run_2.png
   :align: center

   Importing dataset

3. **Download the Model**

   When the training is completed, a download button will appear. Click this button to download the trained model, which you can use on AmebaPro2.

.. figure:: ../../_static/user_manual/AI_train_server/ai_train_server_run_3.png
   :align: center

   'Run' and 'Download Model' button