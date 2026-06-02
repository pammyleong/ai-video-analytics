NN Model Loading Steps to RTL8735B 
==================================

.. contents::
  :local:
  :depth: 2

**Arduino SDK -- Load Model from Flash**
----------------------------------------

Step 1: Rename network binary ``.nb`` file
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

- Please rename your customized .nb file into the expected filename to ensure successful loading. You may refer to the :ref:`default_naming` section.

Step 2: Copy to Project Folder
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Windows: ``C:\Users\<USERNAME>\AppData\Local\Arduino15\packages\realtek\hardware\AmebaPro2\<VERSION>\libraries\NeuralNetwork\examples``

Linux: ``/home/<USERNAME>/.arduino15/packages/realtek/hardware/AmebaPro2/<VERSION>/libraries/NeuralNetwork/examples``

Step 3: Open Example
~~~~~~~~~~~~~~~~~~~~

E.g. File → Examples → AmebaNN → ObjectDetectionCallback

Step 4: Set Load Source
~~~~~~~~~~~~~~~~~~~~~~~

E.g. Tools → NN Model Load from: → ``Load from Flash``

Step 5: Update modelSelect()
~~~~~~~~~~~~~~~~~~~~~~~~~~~~

.. code:: c++

    ObjDet.modelSelect(OBJECT_DETECTION, CUSTOMIZED_YOLOV4TINY, NA_MODEL, NA_MODEL);

Step 6: Compile and Upload
~~~~~~~~~~~~~~~~~~~~~~~~~~

Click Upload in Arduino IDE. The ``.nb`` file is packaged into flash automatically.

**Arduino SDK -- Load Model from SD**
-------------------------------------

Step 1: Rename ``.nb`` File
~~~~~~~~~~~~~~~~~~~~~~~~~~~

- Please rename your customized .nb file into the expected filename to ensure successful loading. You may refer to the :ref:`default_naming` section.

Step 2: Open Example
~~~~~~~~~~~~~~~~~~~~

E.g. File → Examples → AmebaNN → ObjectDetectionCallback

Step 3: Set Load Source
~~~~~~~~~~~~~~~~~~~~~~~

E.g. Tools → NN Model Load from: → ``Load from SD``

Step 4: Update modelSelect()
~~~~~~~~~~~~~~~~~~~~~~~~~~~~

.. code:: c++

    ObjDet.modelSelect(OBJECT_DETECTION, CUSTOMIZED_YOLOV4TINY, NA_MODEL, NA_MODEL);

Step 5: Prepare SD card
~~~~~~~~~~~~~~~~~~~~~~~

.. code:: text
    
    SD card root/
    └── NN_MDL/
        └── yolov4_tiny.nb

Step 6: (Optional) Change SD Filename
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Edit SD_Model.cpp at: ``Arduino15\packages\realtek\hardware\AmebaPro2\<VERSION>\libraries\NeuralNetwork\src\SD_Model.cpp``

.. code:: c++

    static void *yolov4_get_SD_filename(void)
    {
        return (void *)"sd:/NN_MDL/yolov4_tiny.nb";  // update filename here
    }

Step 7: Compile, Upload, Insert SD Card
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Click Upload, then insert the SD card before pressing reset button.

**FreeRTOS SDK -- Load Model from Flash**
-----------------------------------------

Based on the ``mmf2_video_example_vipnn_rtsp_init`` YOLOv4 example.

Key Files

.. table:: Example of key files used in FreeRTOS SDK for loading customized model

    ========================================= =======================================
    File                                      Purpose
    ========================================= =======================================
    ``component/file_system/nn/nn_file_op.c`` Controls flash vs SD loading
    ``mmf2_video_example_vipnn_rtsp_init.c``  Main example with USER_LOAD_MODEL block
    ``project/.../test_model/model_yolo.c``   Pre/post-processing for YOLO models
    ========================================= =======================================

Step 1: Rename ``.nb`` File
~~~~~~~~~~~~~~~~~~~~~~~~~~~

- Please rename your customized .nb file into the expected filename to ensure successful loading. You may refer to the :ref:`default_naming` section.

Step 2: Copy ``.nb`` into SDK
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

.. code:: text

    project/realtek_amebapro2_v0_example/src/test_model/
    └── model_nb/
        └── yolov4_tiny.nb    ← replace with your customized model

Step 3: Confirm MODEL_SRC is Flash (default)
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

In ``component/file_system/nn/nn_file_op.c``:

.. code:: c++

    #define MODEL_SRC  MODEL_FROM_FLASH   // default, no change needed

Step 4: Register Model in FWFS JSON
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Edit ``project/realtek_amebapro2_v0_example/GCC-RELEASE/mp/amebapro2_fwfs_nn_models.json``:

.. code:: json

    {
        "msg_level": 3,
        "PROFILE": ["FWFS"],
        "FWFS": {
            "files": ["MODEL0"]
        },
        "MODEL0": {
            "name": "yolov4_tiny.nb",
            "source": "binary",
            "file": "yolov4_tiny.nb"
        }
    }

.. note:: Only list models you actually use — unused models bloat the final image.

Step 5: Configure the Example
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

In ``mmf2_video_example_vipnn_rtsp_init.c``:

.. code:: c

    #define YOLO_MODEL      1
    #define USE_NN_MODEL    YOLO_MODEL

    #if (USE_NN_MODEL == YOLO_MODEL)
    #define NN_WIDTH    416
    #define NN_HEIGHT   416
    static float nn_confidence_thresh = 0.4;
    static float nn_nms_thresh = 0.3;
    #endif

Enable the example in ``video_example_media_framework.c``:

.. code:: C

    mmf2_video_example_vipnn_rtsp_init();

Step 6: Build
~~~~~~~~~~~~~~

.. code:: bash

    cmake .. -G"Unix Makefiles" -DCMAKE_TOOLCHAIN_FILE=../toolchain.cmake -DVIDEO_EXAMPLE=ON
    cmake --build . --target flash_nn

Output: ``flash_ntz.nn.bin`` in ``GCC-RELEASE/build/``

Step 7: Flash to the Board
~~~~~~~~~~~~~~~~~~~~~~~~~~

Windows: ``uartfwburn.exe -p COM? -f flash_ntz.nn.bin -b 3000000 -U -x 32``

Linux: ``./uartfwburn.linux -p /dev/ttyUSB? -f ./flash_ntz.nn.bin -b 3000000 -U -x 32``

**FreeRTOS SDK -- Load Model from SD Card**
-------------------------------------------

Based on the ``mmf2_video_example_vipnn_rtsp_init`` YOLOv4 example.

Key Files

.. table:: Example of key files used in FreeRTOS SDK for loading customized model

    ========================================= =======================================
    File                                      Purpose
    ========================================= =======================================
    ``component/file_system/nn/nn_file_op.c`` Controls flash vs SD loading
    ``mmf2_video_example_vipnn_rtsp_init.c``  Main example with USER_LOAD_MODEL block
    ``project/.../test_model/model_yolo.c``   Pre/post-processing for YOLO models
    ========================================= =======================================

Step 1: Rename ``.nb`` File
~~~~~~~~~~~~~~~~~~~~~~~~~~~

- Please rename your customized .nb file into the expected filename to ensure successful loading. You may refer to the :ref:`default_naming` section.

Step 2: Prepare the SD Card
~~~~~~~~~~~~~~~~~~~~~~~~~~~

.. code:: text

    SD card root/
    └── NN_MDL/
        └── yolov4_tiny.nb

Step 3: Set MODEL_SRC to SD
~~~~~~~~~~~~~~~~~~~~~~~~~~~~

In ``component/file_system/nn/nn_file_op.c``:

.. code:: C

    #define MODEL_FROM_FLASH  0x01
    #define MODEL_FROM_SD     0x02
    #define MODEL_SRC         MODEL_FROM_SD

Step 4: Enable USER_LOAD_MODEL in Example
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

In ``mmf2_video_example_vipnn_rtsp_init.c``, set ``USER_LOAD_MODEL`` to 1 and define the model object:

.. code:: C

    #define USER_LOAD_MODEL     1

    #include "vfs.h"
    static void *example_get_model_name(void)
    {
        return (void *)"sd:/NN_MDL/yolov4_tiny.nb";
    }

    extern void yolov4_set_network_init_info(void *m);
    extern int yolo_preprocess(void *data_in, nn_data_param_t *data_param, void *tensor_in, nn_tensor_param_t *tensor_param);
    extern int yolo_postprocess(void *tensor_out, nn_tensor_param_t *param, void *res);
    extern void yolo_set_confidence_thresh(void *confidence_thresh);
    extern void yolo_set_nms_thresh(void *nms_thresh);

    nnmodel_t yolov4_tiny_from_sd = {
        .nb                    = example_get_model_name,
        .set_init_info         = yolov4_set_network_init_info,
        .preprocess            = yolo_preprocess,
        .postprocess           = yolo_postprocess,
        .model_src             = MODEL_SRC_FILE,
        .set_confidence_thresh = yolo_set_confidence_thresh,
        .set_nms_thresh        = yolo_set_nms_thresh,
        .name = "YOLOv4t_SD"
    };

Enable the example in ``video_example_media_framework.c``:

.. code:: C

    mmf2_video_example_vipnn_rtsp_init();

Step 5: Build
~~~~~~~~~~~~~

.. code:: bash

    cmake .. -G"Unix Makefiles" -DCMAKE_TOOLCHAIN_FILE=../toolchain.cmake -DVIDEO_EXAMPLE=ON
    cmake --build . --target flash_nn

Step 6: Flash to the Board
~~~~~~~~~~~~~~~~~~~~~~~~~~

Windows: ``uartfwburn.exe -p COM? -f flash_ntz.nn.bin -b 3000000 -U -x 32``

Linux: ``./uartfwburn.linux -p /dev/ttyUSB? -f ./flash_ntz.nn.bin -b 3000000 -U -x 32``

Then, insert SD card before pressing the reset button.

.. _default_naming:

Default Supported Model Filenames
---------------------------------

+----------------------+----------------------------------+
| Category             |Expected Filename                 |
+======================+==================================+
| Object Detection     | | yolov3_tiny.nb                 |
|                      | | yolov4_tiny.nb                 |
|                      | | yolov7_tiny.nb                 |
+----------------------+----------------------------------+
| Face Detection       | | scrfd_500m_bnkps_640x640_u8.nb |
|                      | | scrfd_500m_bnkps_576x320_u8.nb |
+----------------------+----------------------------------+
| Face Recognition     | | mobilefacenet_int8.nb          |
|                      | | mobilefacenet_int16.nb         |
+----------------------+----------------------------------+
| Audio Classification | | yamnet_fp16.nb                 |
|                      | | yamnet_s_hybrid.nb             |
+----------------------+----------------------------------+
| Image Classification | | mobilenetv2_int16.nb           |
|                      | | img_class_cnn.nb               |
+----------------------+----------------------------------+
| Hand Gesture         | | palm_detection_lite_int16.nb   |
|                      | | hand_landmark_lite_int16.nb    |
+----------------------+----------------------------------+

.. note:: 

    - For SD card loading, all files must be placed inside NN_MDL/ on the SD card: ``sd:/NN_MDL/<filename>.nb``
    - For flash loading (Arduino SDK), filenames must match exactly and be placed in the NeuralNetwork examples folder
    - When running object detection + face detection simultaneously, use ``scrfd_500m_bnkps_576x320_u8.nb`` paired with ``yolov4_tiny_576x320.nb`` (same resolution)
    - ``mobilefacenet_int16.nb`` is preferred over int8 for better face recognition accuracy