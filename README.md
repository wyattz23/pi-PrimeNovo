## Environment Setup

Note: we have developed our algorithm in CentOS Linux Version 7, other OS system need to check compability themselves. 

Create a new conda environment first:

```
conda create --name PrimeNovo python=3.10
```

This will create an anaconda environment 

Activate this environment by running:

```
conda activate PrimeNovo
```

then install dependencies:

```
pip install -r ./requirements.txt
```



installing gcc and g++:

```bash
conda install -c conda-forge gcc
conda install -c conda-forge cxx-compiler
```

then install ctcdecode, which is the package for ctc-beamsearch decoding


```bash
git clone --recursive https://github.com/WayenVan/ctcdecode.git
cd ctcdecode
pip install .
cd ..  #this is needed as ctcdecode can not be imported under the current directory
rm -rf ctcdecode
```


(if there is no errors, ignore next line and preceed to CuPy install)


if you encountered issues with C++ (gxx and gcc) version errors in this step, install gcc with version specified as :  
```bash
conda install -c conda-forge gcc_linux-64=9.3.0
```




lastly, install CuPy to use our CUDA-accelerated precise mass-control decoding:

**_Please install following Cupy package in a GPU available env, If you are using a slurm server, this means you have to enter a interative session with sbatch to install Cupy, If you are using a machine with GPU already on it (checking by nvidia-smi), then there's no problem_**


**Check your Cuda Version using command nvidia-smi, CUDA version will be on right top corner**

| cuda version | command |
|-------|-------|
|v10.2 (x86_64 / aarch64)| pip install cupy-cuda102 |
|v11.0 (x86_64)| pip install cupy-cuda110 |
|v11.1 (x86_64)| pip install cupy-cuda111 |
|v11.2 ~ 11.8 (x86_64 / aarch64)| pip install cupy-cuda11x |
|v12.x (x86_64 / aarch64)| pip install cupy-cuda12x |


## Model Settings

**n_beam**: number of CTC-paths (beams) considered during inference. We recommend a value of 40.

**mass_control_tol**: This setting is only useful when **PMC_enable** is ```True```. The tolerance of PMC-decoded mass from the measured mass by MS, when mass control algorithm (PMC) is used. For example, if this set to 0.1, we will only obtain peptitde that falls under the mass range [measured_mass-0,1, measured_mass+0,1]. ```Measured mass``` is calculated by : (pepMass - 1.007276) * charge - 18.01. pepMass and charge are given by input spectrum file (MGF).


**PMC_enable**: Weather use PMC decoding unit or not, either ```True``` or ```False```.

**n_peaks**: Number of the most intense peaks to retain, any remaining peaks are discarded. We recommend a value of 800.

**min_mz**: Minimum peak m/z allowed, peaks with smaller m/z are discarded. We recommend a value of 1.

**max_mz**: Maximum peak m/z allowed, peaks with larger m/z are discarded. We recommend a value of 6500.

**min_intensity**: Min peak intensity allowed, less intense peaks are discarded. We recommend a value of 0.0.


## Run Instructions

**Note!!!!!!!!!!!!!!!!!!:** All the following steps should be performed under the main directory: `PrimeNovo-main`. Do **not** use `cd PrimeNovo` !!!!!!!!!!!!!!!!!!!

### Step 1: Download Required Files

To evaluate the provided test MGF file (you can replace this MGF file with your own), download the following files:

1. **Model Checkpoint**: [model_massive.ckpt](https://drive.google.com/file/d/12IZgeGP3ae3KksI5_82yuSTbk_M9sKNY/view?usp=share_link)
2. **Test MGF File**: [Bacillus.10k.mgf](https://drive.google.com/file/d/1HqfCETZLV9ZB-byU0pqNNRXbaPbTAceT/view?usp=drive_link)

**Note:** If you are using a remote server, you can use the `gdown` package to easily download the content from Google Drive to your server disk.

### Step 2: Choose the Mode

The `--mode` argument can be set to either:

- `eval`: Use this mode when evaluating data with a labeled dataset.
- `denovo`: Use this mode for de novo analysis on unlabeled data.

**Important**: Select `eval` only if your data is labeled.

### Step 3: Run the Commands

Execute the following command in the terminal:

```bash
python -m PrimeNovo.PrimeNovo --mode=eval --peak_path=./bacillus.10k.mgf --model=./model_massive.ckpt
```

This automatically uses all GPUs available in current machine.


## Sample run output:

we include a sample running output ```./output.txt```

The performance for evaluaiton will be reported at the end of output file. 