# **Advanced Lane Finding** 
[![Udacity - Self-Driving Car NanoDegree](https://s3.amazonaws.com/udacity-sdc/github/shield-carnd.svg)](http://www.udacity.com/drive)

<img src="examples/example_output.jpg" width="480" alt="Combined Image" />

Overview
---

This project detects lane lines on the road in pictures and videos using the following pipeline:
1. Camera Calibration
2. Gradients and Color Thresholding
3. Perspective Transform
4. Lane Finding (sliding window search)
5. Curvature Measurement
6. Reverse Perspective Transform
 
The code, test images, and test videos originated from [CarND-Advanced-Lane-Lines](https://github.com/udacity/CarND-Advanced-Lane-Lines)

Installation
---

To run this project, follow the setup guideline in [CarND Term1 Starter Kit](https://github.com/udacity/CarND-Term1-Starter-Kit/blob/master/README.md)

**NOTE:** If Miniconda is used, install Jupyter Notebook before creating the `carnd-term1` environment

```
conda install jupyter notebook
```

Usage
---

For a quickview of the results, open `P2.html` in the repository

**NOTE:** Activate the `carnd-term1` environment before opening the notebook

```
activate carnd-term1
```

Alternatively, change the environment in the notebook through `Kernel -> Change kernel`

This project is coded using Jupyter Notebook. There are two ways to run the notebook:

1. Open Jupyter Notebook, navigate to the repository directory, and open `P2.ipynb`

2. Open Terminal/Anaconda prompt, navigate to the repository directory, enter the following command, and open `P2.ipynb`

```
jupyter notebook
```

Once the notebook is opened, the cells in the notebook can be run to detect lane lines on the roads in test images and videos.

Images `test_images` folder and videos from main folder will be processed, and the results are saved to `output_images` and main folder, respectively.