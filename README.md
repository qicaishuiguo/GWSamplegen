# GWSamplegen

A library of tools and scripts for generating compact binary merger samples in real LIGO noise. Specialised for creating SNR time series datasets for use in deep learning.


## Installation

First, ensure you have a python virtual environment you want to install GWSamplegen into (version 3.10 or above). Next, clone this repository with:

```
git clone https://github.com/alistair-mcleod/GWSamplegen.git
```
then `cd` into GWSamplegen and run

```
bash install.sh
```

and the repository should be installed into your virtual environment.

## Usage instructions

To use GWsamplegen, you must first create an arguments file which specifies the parameter distributions of the dataset you wish to create. An example arguments file is provided in `configs/args.json`. You can then run `bash ${GWSAMPLEGEN_DIR}/share/generate_configs_workflow.sh /path/to/args.json` to build the workflow, which will first generate the parameter file, then generate the SNR time series dataset. The generated dataset and parameter file will be saved in the `project_dir` specified in the arguments file.

## Usage notes

#### Reusing matched-filter buffers

`NumpyMatchedFilterBatch` reuses a template batch's conjugate, power and FFT
input buffer across detectors. Each call computes normalization with that
detector's PSD. A single strain row broadcasts across templates.

```python
from GWSamplegen.snr_utils_np import NumpyMatchedFilterBatch

batch = NumpyMatchedFilterBatch(templates, N, kmin, kmax, duration)
h_snr = batch.filter(h_strain, h_psd)
l_magnitude = batch.filter_magnitude(l_strain, l_psd, start, stop)
```

Use complex128 templates with shape `(templates, kmax-kmin)`, strain with the
same shape or one row, and a PSD covering that frequency band. The full inverse
FFT is retained; `filter_magnitude` normally scales, rounds and takes the
magnitude only over the requested sample interval. A singleton interval across
multiple templates keeps the full-series layout to preserve NumPy's rounding.
The method accepts an existing float32 `out` array.
Keep one instance per template batch and use it sequentially. The original
`numpy_matched_filter` function remains available.

#### Condor compatibility
Currently, the job submission scripts are currently written with Slurm syntax, and most of the repository is built around the Slurm job scheduling system. Adapting this code to a Condor-based cluster (i.e. CIT) is currently under development. 

#### Noise availability

While this repository does work Gaussian noise, it is primarily designed to work with real LIGO noise. If you do not have LIGO noise already available, use the `fetch_noise.py` script to download desired segments of LIGO data.
