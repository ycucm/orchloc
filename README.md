# OrchLoc: In-Orchard Localization via a Single LoRa Gateway and Generative Diffusion Model-based Fingerprinting

Welcome to the OrchLoc repository, featuring the official implementation from the authors behind the paper, "OrchLoc: In-Orchard Localization via a Single LoRa Gateway and Generative Diffusion Model-based Fingerprinting."

## 1 Abstract

In orchards, tree-level localization of robots is critical for smart agriculture applications like precision disease management and targeted nutrient dispensing. However, prior solutions cannot provide adequate accuracy. We develop our system, a fingerprinting-based localization system that can provide tree-level accuracy with only one LoRa gateway. We extract channel state information (CSI) measured over eight channels as the fingerprint. To avoid labor-intensive site surveys for building and updating the fingerprint database, we design a CSI Generative Model (CGM) that learns the relationship between CSIs and their corresponding locations. The CGM is fine-tuned using CSIs from static LoRa sensor nodes to build and update the fingerprint database. Extensive experiments in two orchards validate our system's effectiveness in achieving tree-level localization with minimal overhead and enhancing robot navigation accuracy.

## 2 Folder Organization

```plaintext
orchLoc/

├── configs/
│   ├── area_a.yml
│   └── area_b.yml

├── input/
│   ├── csi_area_a.pth
│   └── csi_area_b.pth

├── output/  # Generated by Python code

├── src/  # Source code
│   ├── denoising_diffusion_process/*
│   ├── __init__.py
│   ├── autoencoder.py
│   ├── dataset.py
│   ├── EMA.py
│   ├── parameter_parser.py
│   ├── pixel_diffusion.py
│   └── utils.py

├── 0_location_representation.py
├── 1_pretrain_cgm.py
├── 2_finetune_cgm.py
├── 3_generate_csi_database.py
├── classifier_area_a.py
├── classifier_area_b.py
├── environment.yml
├── README.md
├── run_a.sh
├── run_b_test.sh
└── run_b_train.sh

```

## 3 Configuration Guide

### 3.1 Cloning the Repository

Begin by cloning the repository to your local machine using either SSH or HTTPS:

- **SSH**:
```bash
git clone git@github.com:mobisys24/orchloc.git
```

- **HTTPS**:
```bash
git clone https://github.com/mobisys24/orchloc.git
```


### 3.2 Hardware Requirement

- **GPU**: An NVIDIA GPU with a minimum of 8GB of memory.



### 3.3 Software Requirement

- **Conda**: Recommended for an easy setup process.
- **CUDA Toolkit 11**: The authors' code is tested and proven to work with version 11.7, thus recommending CUDA 11.7. Based on personal experience, any CUDA 11.X version should be compatible. Below is the guide for installing version 11.7:


[CUDA Toolkit 11.7 Installation Guide](https://developer.nvidia.com/cuda-11-7-0-download-archive?target_os=Linux&target_arch=x86_64&Distribution=Ubuntu&target_version=22.04&target_type=deb_local)

This link leads to the download page for CUDA Toolkit 11.7, targeting Linux systems with x86_64 architecture and Ubuntu 22.04 (Jammy Jellyfish). If your system configuration is different, please navigate through the NVIDIA developer site to find the correct version for your setup.

- **Note**: If you lack GPU hardware or prefer not to install the CUDA Toolkit, consider using online platforms such as Vast AI and Lambda Labs. Their virtual machines (VMs) include various versions of the CUDA Toolkit.



### 3.4 Conda Setup

To set up your environment, follow these steps:

1. **Create the Conda Environment**: Use the `environment.yml` to create a Conda environment with all required dependencies.
```bash
conda env create -f environment.yml
```

Note: Ensure CUDA Toolkit 11.X is installed on your system.

2. **Optimize Disk Space Usage**: If disk space is a concern, or if you prefer to organize your packages and environments in specific directories, configure Conda accordingly:
```bash
conda config --add pkgs_dirs <Drive>/<pkg_path>
conda env create --file environment.yml --prefix <Drive>/<env_path>/orchloc
conda activate <Drive>/<env_path>/orchloc
```
Replace `<Drive>/<pkg_path>` and `<Drive>/<env_path>/orchloc` with the actual paths where you'd like to store your Conda packages and the OrchLoc environment, respectively.




## 4 Running OrchLoc

### 4.1 For Area A

To train and test the location classifier for Area A, you'll need to run the `run_a.sh` script. This script utilizes the configuration details specified in `configs/area_a.yml`. You have the flexibility to modify these configurations directly within the file or adjust the python script parameters as necessary to fit your requirements.


**Command**: 

```bash
bash run_a.sh
```


**Input**: 
- CSI data from Area A: `intput/csi_area_a.pth`


**Output**: 
- Trained classifier for Area A: `output/classifier_area_a.ckpt`
- Printing: the classifier training and testing results including loss, accuracy, precision, and recall
- Expected results: Achieve an accuracy, precision, and recall exceeding 90%




### 4.2 For Area B

This section outlines the procedure for Area B, which is divided into two steps: data generation and subsequent evaluation for the generated data.


#### Step 1: Data Generation via `run_b_train.sh`

The generation of data for Area B involves a detailed process that may extend over several hours based on the capabilities of your hardware setup. In the `run_b_train.sh` script, the workflow encompasses CGM pre-training, CGM fine-tuning, and CSI data generation for specific locations. The outcomes of these procedures are saved in the `./output` directory. Please be aware that this operation will overwrite any existing files within the `./output` folder. Configuration adjustments can be made through `configs/area_b.yml`, or directly via python script parameters.


**Optional Step**: If you prefer to bypass the data generation step, pre-generated data for Area B is available at `output/csi_area_b_generated.pth`, allowing you to proceed directly to evaluating the data quality in Step 2.


**Command**: 

```bash
bash run_b_train.sh
```

**Input**:
- CSI data for Area A: `input/csi_area_a.pth`
- CSI data for Area B: `input/csi_area_b.pth`


**Output**: Sequentially generate the following files; typically, the outputs from one file are utilized by the next Python script.
- Location vector file: `output/0_location_vector.txt`
- Pre-trained CGM: `output/1_pretrained_cgm.ckpt`
- Fine-tuned CGM: `output/2_finetuned_cgm.ckpt`
- Generated CSI data: `output/csi_area_b_generated.pth`
- Printing: training details on CGM pre-training, finetuning, and data generating


**Note**:
- The `output/0_location_vector.txt` file is generated by the first Python script named `0_location_representation.py`. This file serves as a foundational input for all subsequent scripts executed in `run_b_train.sh`.
- Each execution of `0_location_representation.py` will overwrite the pre-existing `output/0_location_vector.txt` file, irrespective of the script’s completion status—successful, failed, or manually interrupted. 
- Users are advised to ensure a successful and uninterrupted execution of `0_location_representation.py` to prevent disruptions in the workflow and errors in subsequent script executions.



#### Step 2: Evaluating Generated Data Quality via `run_b_test.sh` 

This step focuses on assessing the quality of the dataset generated by the CGM for Area B. The core of this evaluation involves using the CGM-generated dataset to train a location classifier. The effectiveness of this classifier is then evaluated against an actual dataset collected from Area B, providing insight into the fidelity and usability of the generated data.
Configuration adjustments can be made through `configs/area_b.yml`, or directly via python script parameters.


**Command**: 

```bash
bash run_b_test.sh
```


**Input**:
- Actual and Generated Data for Area B: `output/csi_area_b_generated.pth`


**Output**:
- Trained classifier for Area B: `output/classifier_area_b.ckpt`
- Printing: the classifier training and testing results including loss, accuracy, precision, and recall
- Expected results: Achieve an accuracy, precision, and recall exceeding 90%





