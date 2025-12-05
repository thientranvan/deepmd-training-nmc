# DeepMD Training for Ni-rich NMC Cathodes

[![DOI](https://img.shields.io/badge/DOI-10.1039%2FD5RA03510D-purple.svg)](https://doi.org/10.1039/D5RA03510D)
[![DOI](https://zenodo.org/badge/1097141640.svg)](https://doi.org/10.5281/zenodo.17618776)

This repository contains a self-contained Jupyter workflow to train and validate a Deep Potential (DeePMD) model for Ni-rich NMC cathode materials directly from a single VASP `OUTCAR` file.

The example in this repository uses the file:

- `MD_075NMC622_500step-300K` (VASP `OUTCAR` format)

The code and workflow were used in the following peer-reviewed publication:

> **Nguyen Vo Anh Duy, Tran Van Thien, Nguyen Thi Bao Trang, Nguyen Huu Phuc,  
> To Van Nguyen, Truong Minh Thai, Yoshiyuki Kawazoe, and Minh Triet Dang.**  
> *Enhancement of stability and electronic properties of ultra-high nickel cathodes by aluminium and boron codoping studied by combined density functional theory and neural network models.*  
> **RSC Advances**, 2025.  
> [![DOI](https://img.shields.io/badge/DOI-10.1039%2FD5RA03510D-purple.svg)](https://doi.org/10.1039/D5RA03510D)

If you use this repository, model, or workflow, please cite the publication above.

---

## 1. Repository Structure

A typical working directory for this notebook looks like:

```text
.
├─ 00_data/                       # created/used by the notebook
├─ 01_train/                      # DeepMD training directory (contains input.json)
├─ 02_lmp/                        # (optional) LAMMPS input and examples
├─ deepmd_training_nmc.ipynb      # main Jupyter notebook
└─ MD_075NMC622_500step-300K      # VASP OUTCAR file used as example dataset
```

### What you need to prepare manually

1. **One VASP OUTCAR file** for your system  
   - In the example: `MD_075NMC622_500step-300K`  
   - You can replace it with your own OUTCAR file.

2. **Folders**
   - `00_data/` – can be empty at the beginning; the notebook will write training/validation data here.
   - `01_train/` – should contain a DeePMD `input.json` template (e.g. copied from DeePMD examples).

3. (Optional) `02_lmp/` – if you want to store LAMMPS scripts using the trained model.

> The notebook automatically:
> - Reads the OUTCAR file  
> - Splits data into training/validation sets  
> - Exports DeepMD-formatted data into `00_data/training_data` and `00_data/validation_data`  
> - Modifies `01_train/input.json` with suitable parameters  
> - Runs `dp train`, `dp freeze`, `dp compress`, and `dp test`  

---

## 2. Main Notebook: `deepmd_training_nmc.ipynb`

Open and run the notebook step by step.

### Step 0 – Set the working directory

At the top of the notebook, the variable `root_dir` defines your working directory:

```python
root_dir = os.path.abspath("/path/to/your/working/directory")
print("Root directory:", root_dir)
```

Update this path so that it points to the directory containing:

- `MD_075NMC622_500step-300K`
- `00_data/`
- `01_train/`

### Step 1 – Data preparation from OUTCAR

The notebook:

- Changes into `00_data/`
- Reads the OUTCAR file using `dpdata`:

```python
data = dpdata.LabeledSystem(
    os.path.join(root_dir, "MD_075NMC622_500step-300K"),
    fmt="vasp/outcar",
)
```

- Randomly splits the frames into:
  - 80% **training**
  - 20% **validation**

- Writes DeepMD numpy-format datasets into:

```text
00_data/training_data/
00_data/validation_data/
```

You **do not** need to create these subfolders manually; they are created by `data_training.to_deepmd_npy(...)` and `data_validation.to_deepmd_npy(...)`.

### Step 2 – Update `input.json` automatically

The notebook then opens:

```text
01_train/input.json
```

and updates key parameters, for example:

- `training.numb_steps` – number of training steps (e.g. `10000`)
- `model.type_map` – element types: `["Li", "Ni", "O", "Mn", "Co"]`
- `model.descriptor.sel` – neighbor selection numbers
- `model.descriptor.neuron` – neural network layer sizes

The updated configuration is written back to `01_train/input.json` and printed in the notebook output.

You can further tweak values in the notebook or directly in `input.json` if desired.

### Step 3 – Train, freeze, compress, and test the model

From inside `01_train/`, the notebook runs the standard DeepMD workflow:

```bash
dp train input.json
dp freeze -o graph.pb
dp compress -i graph.pb -o graph-compress.pb
dp test -m graph-compress.pb -s ../00_data/validation_data -n 100 -d tests
```

This will produce (inside `01_train/`):

- `graph.pb`
- `graph-compress.pb`
- `lcurve.out`
- `tests.e.out`
- and other DeepMD log files and test outputs.

### Step 4 – Plot learning curves and prediction accuracy

The second cell of the notebook (STEP 4) reads the output files and generates plots:

- `lcurve.out` → `lcurve.png` (loss/error vs. training steps)
- `tests.e.out` → `tests.e.png` and related plots of predicted vs DFT energies
- Optional force comparison plots, e.g. `tests.f.png`

Plots are displayed in the notebook and also saved as PNG files in `01_train/`.

---

## 3. Environment Requirements

This notebook is designed for **Jupyter Notebook** or **Google Colab**.

Recommended environment:

- Python **3.8–3.10**
- CUDA **11.x** or **12.x** (for GPU acceleration)
- cuDNN **8.x** (compatible with your CUDA version)
- TensorFlow **≥ 2.10** (GPU-enabled build)
- DeePMD-kit: `deepmd`, `dpdata`, `deepmd-kit`
- Additional Python packages:
  - `numpy`
  - `matplotlib`
  - `scikit-learn`
  - `jupyter` / `jupyterlab`

Check GPU availability:

```bash
python -c "import tensorflow as tf; print(tf.config.list_physical_devices('GPU'))"
```

---

## 4. Using the Trained Model in LAMMPS

The compressed model:

```text
01_train/graph-compress.pb
```

can be used in LAMMPS as:

```text
pair_style  deepmd graph-compress.pb
pair_coeff  * *
```

You can place your LAMMPS input scripts and auxiliary files in the `02_lmp/` folder (notebook does not depend on this folder).

---

## 5. Citation

If you use this workflow, dataset, notebook, or trained models in your research,  
please cite the following peer-reviewed publication where this workflow was used:

> **Nguyen Vo Anh Duy, Tran Van Thien, Nguyen Thi Bao Trang, Nguyen Huu Phuc,  
> To Van Nguyen, Truong Minh Thai, Yoshiyuki Kawazoe, and Minh Triet Dang.**  
> *Enhancement of stability and electronic properties of ultra-high nickel cathodes by aluminium and boron codoping studied by combined density functional theory and neural network models.*  
> **RSC Advances**, 2025.  
> [![DOI](https://img.shields.io/badge/DOI-10.1039%2FD5RA03510D-purple.svg)](https://doi.org/10.1039/D5RA03510D)

BibTeX entry:

```bibtex
@article{duy2025ultrahighnickel,
  title={Enhancement of stability and electronic properties of ultra-high nickel cathodes by aluminium and boron codoping studied by combined density functional theory and neural network models},
  author={Nguyen Vo Anh Duy and Tran Van Thien and Nguyen Thi Bao Trang and Nguyen Huu Phuc and To Van Nguyen and Truong Minh Thai and Yoshiyuki Kawazoe and Minh Triet Dang},
  journal={RSC Advances},
  year={2025},
  doi={10.1039/D5RA03510D}
}

---

## 6. License

This project is released under the **MIT License** (see `LICENSE` for details).
