## Content Tabs

This is some examples of content tabs.

### Generic Content
=== "Plain"
    This is some plain text

=== "Unordered list"
    * First Item
    * Second Item
    * Third Item

=== "Ordered list"
    1. First item
    2. Second item
    3. Third item

## Install TensorFlow with pip

=== "Linux"
    !!! note "Note"

        Starting with TensorFlow **2.10**, Linux CPU-builds for Aarch64/ARM64 processors are built, maintained, tested and released by a third party: [AWS](https://aws.amazon.com/). Installing the [**tensorflow**](https://pypi.org/project/tensorflow/) package on an ARM machine installs AWS's [**tensorflow-cpu-aws package**](https://pypi.org/project/tensorflow-cpu-aws/). They are provided as-is. Tensorflow will use reasonable efforts to maintain the availability and integrity of this pip package. There may be delays if the third party fails to release the pip package. See this blog post for more information about this collaboration.

    ```shell
    python3 -m pip install tensorflow[and-cuda]
    # Verify the installation:
    python3 -c "import tensorflow as tf; print(tf.config.list_physical_devices('GPU'))"

    ```
=== "MacOS"
    ```shell
    # There is currently no official GPU support for MacOS.
    python3 -m pip install tensorflow
    # Verify the installation:
    python3 -c "import tensorflow as tf; print(tf.reduce_sum(tf.random.normal([1000, 1000])))"
    ```

=== "Windows Native"

    **Caution:** TensorFlow 2.10 was the last TensorFlow release that supported GPU on native-Windows. Starting with TensorFlow 2.11, you will need to install TensorFlow in WSL2, or install tensorflow-cpu and, optionally, try the TensorFlow-DirectML-Plugin

    ```shell
    conda install -c conda-forge cudatoolkit=11.2 cudnn=8.1.0
    # Anything above 2.10 is not supported on the GPU on Windows Native
    python -m pip install "tensorflow<2.11"
    # Verify the installation:
    python -c "import tensorflow as tf; print(tf.config.list_physical_devices('GPU'))"
    ```

=== "Windows WSL2"
    ```python
    python3 -m pip install tensorflow[and-cuda]
    # Verify the installation:
    python3 -c "import tensorflow as tf; print(tf.config.list_physical_devices('GPU'))"
    ```

=== "CPU"
    ```python
    python3 -m pip install tensorflow
    # Verify the installation:
    python3 -c "import tensorflow as tf; print(tf.reduce_sum(tf.random.normal([1000, 1000])))"
    ```
=== "Nightly"
    ```python
    python3 -m pip install tf-nightly
    # Verify the installation:
    python3 -c "import tensorflow as tf; print(tf.reduce_sum(tf.random.normal([1000, 1000])))"
    ```

### Code Blocks in Content Tabs
=== "C"

    ``` c
    #include <stdio.h>

    int main(void) {
      printf("Hello world!\n");
      return 0;
    }
    ```

=== "C++"

    ``` c++
    #include <iostream>

    int main(void) {
      std::cout << "Hello world!" << std::endl;
      return 0;
    }
    ```
