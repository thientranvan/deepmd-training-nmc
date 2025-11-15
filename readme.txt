FILE: deepmd_training_nmc.ipynb - chạy trên môi trường Jupyter Notebook or Google Colab

*** Dữ liệu đầu vào: file OUTCAR (xuất từ VASP) chứa thông tin cấu trúc, năng lượng nguyên tử của vật liệu (VD ở đây là mẫu NMC622)

*** Chuẩn bị dữ liệu và huấn luyện mô hình (code ở cell 1 trong notebook)
 - Đọc dữ liệu OUTCAR, chia thành tập huấn luyện và tập kiểm thử.
 - Tạo thư mục training_data và validation_data.
 - Chỉnh sửa file cấu hình input.json (số bước học, mạng neuron, tham số sel,...).
 - Chạy huấn luyện mô hình Deep Potential (DP) để tạo các file graph.pb và graph-compress.pb – mô hình thế năng học sâu.
 
 *** Kiểm tra kết quả và vẽ đồ thị (code ở cell 2 trong notebook)
  - Dùng mô hình đã huấn luyện để kiểm thử trên tập dữ liệu kiểm định.
  - Sinh các file kết quả:
    + lcurve.out → dữ liệu đường cong huấn luyện
    + tests.e.out → so sánh năng lượng dự đoán và DFT
   - Vẽ đồ thị hiển thị trực tiếp trong notebook:
    + Learning curve: độ lỗi theo bước học.
    + Prediction vs Data: năng lượng dự đoán được từ mô hình so với dữ liệu tham chiếu.
    
 *** Dữ liệu đầu ra cuối cùng:
  - Mô hình DP: graph-compress.pb – dùng cho mô phỏng LAMMPS (pair_style deepmd).
  - Đồ thị kết quả: lcurve.png, tests.e.png – đánh giá độ chính xác mô hình.
  
  
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
