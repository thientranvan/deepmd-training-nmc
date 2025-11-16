FILE: deepmd_training_nmc.ipynb - chạy trên môi trường Jupyter Notebook or Google Colab
# ==========================================================
# FILE: deepmd_training_nmc.ipynb
# PURPOSE: Train and validate a Deep Potential model from VASP (OUTCAR) data of NMC
# ==========================================================

# ===========================================
# REQUIREMENTS (Environment Setup)
# ===========================================
# This script is designed to be executed in a Jupyter Notebook or Google Colab environment.
# Make sure you have Jupyter or JupyterLab installed and run this file inside it
# to visualize plots and outputs directly within the notebook.
#
# To run this notebook successfully, ensure the following environment
# is properly installed and configured:
#
# 1. Python 3.8+  (recommended: Python 3.10)
# 2. CUDA Toolkit 11.x or 12.x  (for GPU acceleration)
# 3. cuDNN 8.x  (compatible with your CUDA version)
# 4. TensorFlow >= 2.10  (GPU-enabled build)
# 5. DeePMD-kit  (dp, dpdata, deepmd) compatible with your TensorFlow version
# 6. dpdata >= 0.2.20
# 7. NumPy, Matplotlib, scikit-learn
#
# Test GPU availability:
#   python -c "import tensorflow as tf; print(tf.config.list_physical_devices('GPU'))"
# ===========================================

# ===========================================
# *** Input Data: OUTCAR file (exported from VASP) containing structural and atomic energy information of the material (e.g., NMC622 sample)
#
# *** Data Preparation and Model Training (script in cell 1 of the notebook)
#  - Read the OUTCAR data and split it into training and validation sets.
#  - Create the folders training_data and validation_data.
#  - Edit the configuration file input.json (number of training steps, neural network layers, sel parameters, etc.).
#  - Run Deep Potential (DP) model training to generate the files graph.pb and graph-compress.pb – the trained deep potential model.
#
# *** Model Evaluation and Plotting (script in cell 2 of the notebook)
#  - Use the trained model to test on the validation dataset.
#  - Generate result files:
#    + lcurve.out → training loss and error over steps
#    + tests.e.out → comparison between predicted and DFT reference energies
#  - Display plots directly in the notebook:
#    + Learning curve: shows loss/error vs. training steps
#    + Prediction vs Data: compares model-predicted energies with DFT reference data
#
# *** Final Outputs:
#  - DP model: graph-compress.pb – used for LAMMPS simulation (pair_style deepmd)
#  - Result plots: lcurve.png, tests.e.png – used to evaluate model accuracy
# ===========================================
