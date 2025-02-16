<p align="center">
<picture>
  <source media="(prefers-color-scheme: dark)" srcset="images/logo_white.png">
  <source media="(prefers-color-scheme: light)" srcset="images/logo_black.png">
  <img alt="nablaDFT logo" src="images/logo_black.png">
</picture>
</p>

# $\nabla^2$ DFT: A Universal Quantum Chemistry Dataset of Drug-Like Molecules and a Benchmark for Neural Network Potentials
<p align="left">
<a href="https://developer.nvidia.com/cuda-downloads"><img alt="CUDA versions" src="https://img.shields.io/badge/cuda-11.8~12.1-green"></a>
<a href="https://github.com/AIRI-Institute/nablaDFT/blob/main/LICENSE"><img alt="License" src="https://img.shields.io/badge/license-MIT-blue"></a>
<a href="https://github.com/psf/black"><img alt="Code style: black" src="https://img.shields.io/badge/code%20style-black-000000.svg"></a>
</p>


This is the repository for nablaDFT Dataset and Benchmark. The current version is 2.0. The code and data from the initial publication are accessible here: [1.0 branch](https://github.com/AIRI-Institute/nablaDFT/tree/1.0). <br/>
Electronic wave function calculation is a fundamental task of computational quantum chemistry. Knowledge of the wave function parameters allows one to compute physical and chemical properties of molecules and materials.<br/>
In this work we: introduce a new curated large-scale dataset of electron structures of drug-like molecules, establish a novel benchmark for the estimation of molecular properties in the multi-molecule setting, and evaluate a wide range of methods with this benchmark.<br/>

More details can be found in the [paper](https://pubs.rsc.org/en/content/articlelanding/2022/CP/D2CP03966D).

If you are using nablaDFT in your research paper, please cite us as
```
@article{10.1039/D2CP03966D,
author ="Khrabrov, Kuzma and Shenbin, Ilya and Ryabov, Alexander and Tsypin, Artem and Telepov, Alexander and Alekseev, Anton and Grishin, Alexander and Strashnov, Pavel and Zhilyaev, Petr and Nikolenko, Sergey and Kadurin, Artur",
title  ="nablaDFT: Large-Scale Conformational Energy and Hamiltonian Prediction benchmark and dataset",
journal  ="Phys. Chem. Chem. Phys.",
year  ="2022",
volume  ="24",
issue  ="42",
pages  ="25853-25863",
publisher  ="The Royal Society of Chemistry",
doi  ="10.1039/D2CP03966D",
url  ="http://dx.doi.org/10.1039/D2CP03966D"}
```

![pipeline](images/pipeline.png)


## Installation

```python
git clone https://github.com/AIRI-Institute/nablaDFT && cd nablaDFT/
pip install .
```

## Dataset

We propose a benchmarking dataset based on a subset of [Molecular Sets (MOSES) dataset](https://github.com/molecularsets/moses). Resulting dataset contains 1 936 931 molecules with atoms C, N, S, O, F, Cl, Br, H. It contains 226 424 unique Bemis-Murcko scaffolds and 34 572 unique BRICS fragments.<br/>
For each molecule in the dataset we provide from 1 to 62 unique conformations, with 12 676 264 total conformations. For each conformation, we have calculated its electronic properties including the energy (E), DFT Hamiltonian matrix (H), and DFT overlap matrix (S). All properties were calculated using the Kohn-Sham method at ωB97X-D/def2-SVP levels of theory using the quantum-chemical software package [Psi4](https://github.com/psi4/psi4), version 1.5. <br/>
We provide several splits of the dataset that can serve as the basis for comparison across different models.<br/>
As part of the benchmark, we provide separate databases for each subset and task and a complete archive with wave function files produced by the Psi4 package that contains quantum chemical properties of the corresponding molecule and can be used in further computations.
### Downloading dataset
#### Hamiltonian databases
Links to  hamiltonian databases including different train and test subsets are in file [Hamiltonian databases](./nablaDFT/links/hamiltonian_databases.json)<br/>
#### Energy databases
Links to energy databases including different train and test subsets are in file [Energy databases](./nablaDFT/links/energy_databases.json)

#### Raw psi4 wave functions
Links to tarballs: [wave functions](./nablaDFT/links/nablaDFT_psi4wfn_links.txt)

#### Summary file
The csv file with conformations index, SMILES, atomic DFT properties and wfn archive names: [summary.csv](https://a002dlils-kadurin-nabladft.obs.ru-moscow-1.hc.sbercloud.ru/data/nablaDFT/summary.csv)

The csv file with conformations index, energies and forces for optimization trajectories: [trajectories_summary.csv](https://a002dlils-kadurin-nabladft.obs.ru-moscow-1.hc.sbercloud.ru/data/nablaDFTv2/summary_relaxation_trajectories.csv.gz)
#### Conformations files
Tar archive with xyz files [archive](https://a002dlils-kadurin-nabladft.obs.ru-moscow-1.hc.sbercloud.ru/data/nablaDFTv2/conformers_archive_v2.tar)

### Accessing elements of the dataset
#### Hamiltonian database
Downloading of the smallest file (`train-tiny` data split, 14 Gb):
```bash
wget https://a002dlils-kadurin-nabladft.obs.ru-moscow-1.hc.sbercloud.ru/data/nablaDFTv2/hamiltonian_databases/train_2k.db
```
Minimal usage example:
```python
from nablaDFT.dataset import HamiltonianDatabase

train = HamiltonianDatabase("train_2k.db")
Z, R, E, F, H, S, C = train[0]  # atoms numbers, atoms positions, energy, forces, core hamiltonian, overlap matrix, coefficients matrix
```
#### Energies database
Downloading of the smallest file (`train-tiny` data split, 51 Mb):
```bash
wget https://a002dlils-kadurin-nabladft.obs.ru-moscow-1.hc.sbercloud.ru/data/nablaDFTv2/energy_databases/train_2k_v2_formation_energy_w_forces.db
```
Minimal usage example:
```python
from ase.db import connect

train = connect("train_2k_v2_formation_energy_w_forces.db")
atoms_data = train.get(1)
```
#### Working with raw psi4 wavefunctions
Downloading of the smallest file (6,8 Gb):
```bash
https://a002dlils-kadurin-nabladft.obs.ru-moscow-1.hc.sbercloud.ru/data/moses_wfns_big/wfns_moses_conformers_archive_0.tar
tar -xf wfns_moses_conformers_archive_0.tar
cd mnt/sdd/data/moses_wfns_big/
```
A variety of properties can be loaded directly from the wavefunction files. 
See main paper for more details. Properties include DFT matrices:
```python
import numpy as np
wfn = np.load('wfn_conf_50000_0.npy', allow_pickle=True).tolist()
orbital_matrix_a = wfn["matrix"]["Ca"]        # alpha orbital coefficients
orbital_matrix_b = wfn["matrix"]["Cb"]        # beta orbital coefficients
density_matrix_a = wfn["matrix"]["Da"]        # alpha electonic density
density_matrix_b = wfn["matrix"]["Db"]        # beta electonic density
aotoso_matrix = wfn["matrix"]["aotoso"]       # atomic orbital to symmetry orbital transformation matrix
core_hamiltonian_matrix = wfn["matrix"]["H"]  # core Hamiltonian matrix
fock_matrix_a = wfn["matrix"]["Fa"]           # DFT alpha Fock matrix
fock_matrix_b = wfn["matrix"]["Fb"]           # DFT betta Fock matrix 
```
and bond orders for covalent and non-covalent interactions and atomic charges: 
```python
import psi4
wfn = psi4.core.Wavefunction.from_file('wfn_conf_50000_0.npy')
psi4.oeprop(wfn, "MAYER_INDICES")
psi4.oeprop(wfn, "WIBERG_LOWDIN_INDICES")
psi4.oeprop(wfn, "MULLIKEN_CHARGES")
psi4.oeprop(wfn, "LOWDIN_CHARGES")
meyer_bos = wfn.array_variables()["MAYER INDICES"]  # Mayer bond indices
lodwin_bos = wfn.array_variables()["WIBERG LOWDIN INDICES"]  # Wiberg bond indices
mulliken_charges = wfn.array_variables()["MULLIKEN CHARGES"]  # Mulliken atomic charges
lowdin_charges = wfn.array_variables()["LOWDIN CHARGES"]  # Löwdin atomic charges
```

## Models
* [Unifying machine learning and quantum chemistry with a deep neural network for molecular wavefunctions (SchNOrb)](https://github.com/KuzmaKhrabrov/SchNOrb)
* [SE(3)-equivariant prediction of molecular wavefunctions and electronic densities (PhiSNet)](./nablaDFT/phisnet/README.md)
* [A continuous-filter convolutional neural network for modeling quantum interactions (SchNet)](./nablaDFT/ase_model/README.md)
* [Equivariant message passing for the prediction of tensorial properties and molecular spectra (PaiNN)](./nablaDFT/ase_model/README.md)
* [Fast and Uncertainty-Aware Directional Message Passing for Non-Equilibrium Molecules (DimeNet++)](./nablaDFT/dimenetplusplus/README.md)
* [EquiformerV2: Improved Equivariant Transformer for Scaling to Higher-Degree Representations (EquiformerV2)](./nablaDFT/equiformer_v2/README.md)
* [Reducing SO(3) Convolutions to SO(2) for Efficient Equivariant GNNs (eSCN)](./nablaDFT/escn/README.md)
* [GemNet-OC: Developing Graph Neural Networks for Large and Diverse Molecular Simulation Datasets (GemNet-OC)](/nablaDFT/gemnet_oc/README.md)
* [Benchmarking Graphormer on Large-Scale Molecular Modeling Datasets (Graphormer3D)](./nablaDFT/graphormer/README.md)
* [Efficient and Equivariant Graph Networks for Predicting Quantum Hamiltonian (QHNet)](./nablaDFT/qhnet/README.md)

### Run
For task start run this command from repository root directory:
```bash
python run.py --config-name <config-name>.yaml
```
For detailed run configuration please refer to [run configuration README](./nablaDFT/README.md).

### Datamodules
To create a dataset, we use interfaces from ASE and PyTorch Lightning.  
An example of the initialisation of ASE-type data classes (for SchNet, PaiNN models) is presented below:
```python
datamodule = ASENablaDFT(split="train", dataset_name="dataset_train_tiny")
datamodule.prepare_data()
# access to dataset
datamodule.dataset
```
For PyTorch Geometric data dataset initialized with PyGNablaDFTDatamodule:
```python
datamodule = PyGNablaDFTDataModule(root="path-to-dataset-dir", dataset_name="dataset_train_tiny", train_size=0.9, val_size=0.1)
datamodule.setup(stage="fit")
```
Similarly, Hamiltonian-type data classes (for SchNOrb, PhiSNet models) are initialised in the following way:
```python
datamodule = PyGHamiltonianDataModule(root="path-to-dataset-dir", dataset_name="dataset_train_tiny", train_size=0.9, val_size=0.1)
datamodule.setup(stage="fit")
```
Dataset itself could be acquired in the following ways:
```python
datamodule.dataset_train
datamodule.dataset_val
```
For more detailed list of datamodules parameters please refer to [datamodule example config](./config/datamodule/nablaDFT_pyg.yaml).

### Checkpoint
Several checkpoints for each model are available here: [checkpoints links](./nablaDFT/links/models_checkpoints.json)

### Examples

Models training and testing example: 
* [PAINN jupyter](examples/PAINN_example.ipynb)
* [Collab](https://colab.research.google.com/drive/1VaiPa05pu-55XR6eR4DXv6cC6fy3lUwJ?usp=sharing)
* [GemNet-OC jupyter](./examples/GemNet-OC_example.ipynb)

Models inference example:
* [GemNet-OC](./examples/Inference%20example.ipynb)

Molecular geometry optimization example:
* [GemNet-OC](./examples/Geometry%20Optimization.ipynb)

### Metrics
In the tables below ST, SF, CF denote structures test set, scaffolds test set and conformations test set correspondingly.

<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: center;">
      <th rowspan="3">Model</th>
      <th colspan="12"> MAE for energy prediction $\times 10^{−2} E_h$ (↓)</th>
    </tr>
    <tr>
      <th colspan="4">Test ST</th>
      <th colspan="4">Test SF</th>
      <th colspan="4">Test CF</th>
    </tr>
    <tr>
      <th>tiny</th>
      <th>small</th>
      <th>medium</th>
      <th>large</th>
      <th>tiny</th>
      <th>small</th>
      <th>medium</th>
      <th>large</th>
      <th>tiny</th>
      <th>small</th>
      <th>medium</th>
      <th>large</th>
      </tr>
  </thead>
  <tbody>
    <tr>
      <td><i>LR</i></td>
      <td><i>4.86</i></td>
      <td><i>4.64</i></td>
      <td><i>4.56</i></td>
      <td><i>4.56</i></td>
      <td><i>4.37</i></td>
      <td><i>4.18</i></td>
      <td><i>4.12</i></td>
      <td><i>4.15</i></td>
      <td><i>3.76</i></td>
      <td><i>3.61</i></td>
      <td><i>3.69</i></td>
      <td><i>3.95</i></td>
    </tr>
    <tr>
      <td><i>SchNet</i></td>
      <td><i>1.17</i></td>
      <td><i>0.90</i></td>
      <td><i>1.10</i></td>
      <td><i>0.31</i></td>
      <td><i>1.19</i></td>
      <td><i>0.92</i></td>
      <td><i>1.11</i></td>
      <td><i>0.31</i></td>
      <td><i>0.56</i></td>
      <td><i>0.63</i></td>
      <td><i>0.88</i></td>
      <td><i>0.28</i></td>
    </tr>
    <tr>
      <td><i>SchNOrb</i></td>
      <td><i>0.83</i></td>
      <td><i>0.47</i></td>
      <td><i>0.39</i></td>
      <td><i>0.39</i></td>
      <td><i>0.86</i></td>
      <td><i>0.46</i></td>
      <td><i>0.37</i></td>
      <td><i>0.39</i></td>
      <td><i>0.37</i></td>
      <td><i>0.26</i></td>
      <td><i>0.27</i></td>
      <td><i>0.36</i></td>
    </tr>
    <tr>
      <td><i>DimeNet++</i></td>
      <td><i>42.84</i></td>
      <td><i>0.56</i></td>
      <td><i>0.21</i></td>
      <td><i>0.09</i></td>
      <td><i>37.41</i></td>
      <td><i>0.41</i></td>
      <td><i>0.19</i></td>
      <td><i>0.08</i></td>
      <td><i>0.42</i></td>
      <td><i>0.10</i></td>
      <td><i>0.09</i></td>
      <td><i>0.07</i></td>
    </tr>
    <tr>
      <td><i>PAINN</i></td>
      <td><i>0.82</i></td>
      <td><i>0.60</i></td>
      <td><i>0.36</i></td>
      <td><i>0.09</i></td>
      <td><i>0.86</i></td>
      <td><i>0.61</i></td>
      <td><i>0.36</i></td>
      <td><i>0.09</i></td>
      <td><i>0.43</i></td>
      <td><i>0.49</i></td>
      <td><i>0.28</i></td>
      <td><i>0.08</i></td>
    </tr>
    <tr>
      <td><i>Graphormer3D-small</i></td>
      <td><i>1.54</i></td>
      <td><i>0.96</i></td>
      <td><i>0.77</i></td>
      <td><i>0.37</i></td>
      <td><i>1.58</i></td>
      <td><i>0.94</i></td>
      <td><i>0.75</i></td>
      <td><i>0.36</i></td>
      <td><i>0.99</i></td>
      <td><i>0.67</i></td>
      <td><i>0.58</i></td>
      <td><i>0.39</i></td>
    </tr>
    <tr>
      <td><i>GemNet-OC</i></td>
      <td><i>2.79</i></td>
      <td><i>0.65</i></td>
      <td><i>0.28</i></td>
      <td><i>0.22</i></td>
      <td><i>2.59</i></td>
      <td><i>0.59</i></td>
      <td><i>0.27</i></td>
      <td><i>0.23</i></td>
      <td><i>0.52</i></td>
      <td><i>0.20</i></td>
      <td><i>0.15</i></td>
      <td><i>0.24</i></td>
    </tr>
    <tr>
      <td><i>Equiformer_V2</i></td>
      <td><i>2.81</i></td>
      <td><i>1.13</i></td>
      <td><i>0.28</i></td>
      <td><i>0.19</i></td>
      <td><i>2.65</i></td>
      <td><i>1.13</i></td>
      <td><i>0.28</i></td>
      <td><i>0.18</i></td>
      <td><i>0.45</i></td>
      <td><i>0.23</i></td>
      <td><i>0.24</i></td>
      <td><i>0.16</i></td>
    </tr>
    <tr>
      <td><i>eSCN</i></td>
      <td><i>1.87</i></td>
      <td><i>0.47</i></td>
      <td><i>0.94</i></td>
      <td><i>0.42</i></td>
      <td><i>1.87</i></td>
      <td><i>0.47</i></td>
      <td><i>0.92</i></td>
      <td><i>0.42</i></td>
      <td><i>0.48</i></td>
      <td><i>0.31</i></td>
      <td><i>0.80</i></td>
      <td><i>0.44</i></td>
    </tr>
  </tbody>
</table>

<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: center;">
      <th rowspan="3">Model</th>
      <th colspan="12"> MAE for forces prediction $\times 10^{−2} E_h*A^{-1}$ (↓)</th>
    </tr>
    <tr>
      <th colspan="4">Test ST</th>
      <th colspan="4">Test SF</th>
      <th colspan="4">Test CF</th>
    </tr>
    <tr>
      <th>tiny</th>
      <th>small</th>
      <th>medium</th>
      <th>large</th>
      <th>tiny</th>
      <th>small</th>
      <th>medium</th>
      <th>large</th>
      <th>tiny</th>
      <th>small</th>
      <th>medium</th>
      <th>large</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td><i>SchNet</i></td>
      <td><i>0.44</i></td>
      <td><i>0.37</i></td>
      <td><i>0.41</i></td>
      <td><i>0.16</i></td>
      <td><i>0.45</i></td>
      <td><i>0.37</i></td>
      <td><i>0.41</i></td>
      <td><i>0.16</i></td>
      <td><i>0.32</i></td>
      <td><i>0.30</i></td>
      <td><i>0.37</i></td>
      <td><i>0.14</i></td>
    </tr>
    <tr>
      <td><i>DimeNet++</i></td>
      <td><i>1.31</i></td>
      <td><i>0.20</i></td>
      <td><i>0.13</i></td>
      <td><i>0.065</i></td>
      <td><i>1.36</i></td>
      <td><i>0.19</i></td>
      <td><i>0.13</i></td>
      <td><i>0.066</i></td>
      <td><i>0.26</i></td>
      <td><i>0.12</i></td>
      <td><i>0.10</i></td>
      <td><i>0.062</i></td>
    </tr>
    <tr>
      <td><i>PAINN</i></td>
      <td><i>0.37</i></td>
      <td><i>0.26</i></td>
      <td><i>0.17</i></td>
      <td><i>0.058</i></td>
      <td><i>0.38</i></td>
      <td><i>0.26</i></td>
      <td><i>0.17</i></td>
      <td><i>0.058</i></td>
      <td><i>0.23</i></td>
      <td><i>0.22</i></td>
      <td><i>0.14</i></td>
      <td><i>0.052</i></td>
    </tr>
    <tr>
      <td><i>Graphormer3D-small</i></td>
      <td><i>1.11</i></td>
      <td><i>0.67</i></td>
      <td><i>0.54</i></td>
      <td><i>0.26</i></td>
      <td><i>1.13</i></td>
      <td><i>0.68</i></td>
      <td><i>0.55</i></td>
      <td><i>0.26</i></td>
      <td><i>0.82</i></td>
      <td><i>0.54</i></td>
      <td><i>0.45</i></td>
      <td><i>0.23</i></td>
    </tr>
    <tr>
      <td><i>GemNet-OC</i></td>
      <td><i>0.14</i></td>
      <td><i>0.051</i></td>
      <td><i>0.036</i></td>
      <td><i>0.021</i></td>
      <td><i>0.10</i></td>
      <td><i>0.051</i></td>
      <td><i>0.036</i></td>
      <td><i>0.021</i></td>
      <td><i>0.073</i></td>
      <td><i>0.042</i></td>
      <td><i>0.032</i></td>
      <td><i>0.021</i></td>
    </tr>
    <tr>
      <td><i>Equiformer_V2</i></td>
      <td><i>0.30</i></td>
      <td><i>0.23</i></td>
      <td><i>0.21</i></td>
      <td><i>0.17</i></td>
      <td><i>0.31</i></td>
      <td><i>0.23</i></td>
      <td><i>0.21</i></td>
      <td><i>0.17</i></td>
      <td><i>0.16</i></td>
      <td><i>0.15</i></td>
      <td><i>0.16</i></td>
      <td><i>0.13</i></td>
    </tr>
    <tr>
      <td><i>eSCN</i></td>
      <td><i>0.10</i></td>
      <td><i>0.051</i></td>
      <td><i>0.036</i></td>
      <td><i>0.021</i></td>
      <td><i>0.10</i></td>
      <td><i>0.051</i></td>
      <td><i>0.036</i></td>
      <td><i>0.021</i></td>
      <td><i>0.065</i></td>
      <td><i>0.037</i></td>
      <td><i>0.029</i></td>
      <td><i>0.021</i></td>
    </tr>
  </tbody>
</table>

<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: center;">
      <th rowspan="3">Model</th>
      <th colspan="12"> MAE for Hamiltonian matrix prediction $\times 10^{−4} E_h$ (↓)</th>
    </tr>
    <tr>
      <th colspan="4">Test ST</th>
      <th colspan="4">Test SF</th>
      <th colspan="4">Test CF</th>
    </tr>
    <tr>
      <th>tiny</th>
      <th>small</th>
      <th>medium</th>
      <th>large</th>
      <th>tiny</th>
      <th>small</th>
      <th>medium</th>
      <th>large</th>
      <th>tiny</th>
      <th>small</th>
      <th>medium</th>
      <th>large</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td><i>SchNOrb</i></td>
      <td><i>198</i></td>
      <td><i>196</i></td>
      <td><i>196</i></td>
      <td><i>198</i></td>
      <td><i>199</i></td>
      <td><i>198</i></td>
      <td><i>200</i></td>
      <td><i>199</i></td>
      <td><i>215</i></td>
      <td><i>207</i></td>
      <td><i>207</i></td>
      <td><i>206</i></td>
    </tr>
    <tr>
      <td><i>PhiSNet</i></td>
      <td><i>1.9</i></td>
      <td><i>3.2(*)</i></td>
      <td><i>3.4(*)</i></td>
      <td><i>3.6(*)</i></td>
      <td><i>1.9</i></td>
      <td><i>3.2(*)</i></td>
      <td><i>3.4(*)</i></td>
      <td><i>3.6(*)</i></td>
      <td><i>1.8</i></td>
      <td><i>3.3(*)</i></td>
      <td><i>3.5(*)</i></td>
      <td><i>3.7(*)</i></td>
    </tr>
    <tr>
      <td><i>QHNet</i></td>
      <td><i>9.8</i></td>
      <td><i>7.9</i></td>
      <td><i>5.2</i></td>
      <td><i>6.9(*)</i></td>
      <td><i>9.8</i></td>
      <td><i>7.9</i></td>
      <td><i>5.2</i></td>
      <td><i>6.9(*)</i></td>
      <td><i>8.4</i></td>
      <td><i>7.3</i></td>
      <td><i>5.2</i></td>
      <td><i>6.8(*)</i></td>
    </tr>
  </tbody>
</table>


<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: center;">
      <th rowspan="3">Model</th>
      <th colspan="12"> MAE for overlap matrix prediction $\times 10^{−5}$(↓)</th>
    </tr>
    <tr>
      <th colspan="4">Test ST</th>
      <th colspan="4">Test SF</th>
      <th colspan="4">Test CF</th>
    </tr>
    <tr>
      <th>tiny</th>
      <th>small</th>
      <th>medium</th>
      <th>large</th>
      <th>tiny</th>
      <th>small</th>
      <th>medium</th>
      <th>large</th>
      <th>tiny</th>
      <th>small</th>
      <th>medium</th>
      <th>large</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td><i>SchNOrb</i></td>
      <td><i>1320</i></td>
      <td><i>1310</i></td>
      <td><i>1320</i></td>
      <td><i>1340</i></td>
      <td><i>1330</i></td>
      <td><i>1320</i></td>
      <td><i>1330</i></td>
      <td><i>1340</i></td>
      <td><i>1410</i></td>
      <td><i>1360</i></td>
      <td><i>1370</i></td>
      <td><i>1370</i></td>
    </tr>
    <tr>
      <td><i>PhiSNet</i></td>
      <td><i>2.7</i></td>
      <td><i>3.0(*)</i></td>
      <td><i>2.9(*)</i></td>
      <td><i>3.3(*)</i></td>
      <td><i>2.6</i></td>
      <td><i>2.9(*)</i></td>
      <td><i>2.9(*)</i></td>
      <td><i>3.2(*)</i></td>
      <td><i>3.0</i></td>
      <td><i>3.2(*)</i></td>
      <td><i>3.1(*)</i></td>
      <td><i>3.5(*)</i></td>
    </tr>
  </tbody>
</table>

We test the ability of the trained models to find low energy conformations.

<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: center;">
      <th rowspan="3">Model</th>
      <th colspan="12"> Optimization metrics</th>
    </tr>
    <tr>
      <th colspan="4">Optimization $pct$ % (↑)</th>
      <th colspan="4">Optimization $pct_{div}$ % (↓)</th>
      <th colspan="4">Optimization success $pct$ % (↑)</th>
    </tr>
    <tr>
      <th>tiny</th>
      <th>small</th>
      <th>medium</th>
      <th>large</th>
      <th>tiny</th>
      <th>small</th>
      <th>medium</th>
      <th>large</th>
      <th>tiny</th>
      <th>small</th>
      <th>medium</th>
      <th>large</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td><i>SchNet</i></td>
      <td><i>39.07</i></td>
      <td><i>40.95</i></td>
      <td><i>36.60</i></td>
      <td><i>80.25</i></td>
      <td><i>42.4</i></td>
      <td><i>38.25</i></td>
      <td><i>47.65</i></td>
      <td><i>6.05</i></td>
      <td><i>0</i></td>
      <td><i>0</i></td>
      <td><i>0</i></td>
      <td><i>3.50</i></td>
    </tr>
    <tr>
      <td><i>PAINN</i></td>
      <td><i>60.60</i></td>
      <td><i>67.30</i></td>
      <td><i>74.67</i></td>
      <td><i>98.45</i></td>
      <td><i>18.70</i></td>
      <td><i>14.55</i></td>
      <td><i>14.00</i></td>
      <td><i>1.50</i></td>
      <td><i>0</i></td>
      <td><i>0.12</i></td>
      <td><i>2.33</i></td>
      <td><i>77.36</i></td>
    </tr>
    <tr>
      <td><i>DimeNet++</i></td>
      <td><i>33.80</i></td>
      <td><i>89.30</i></td>
      <td><i>93.22</i></td>
      <td><i>96.29</i></td>
      <td><i>96.40</i></td>
      <td><i>20.70</i></td>
      <td><i>8.25</i></td>
      <td><i>1.70</i></td>
      <td><i>0</i></td>
      <td><i>12.55</i></td>
      <td><i>33.52</i></td>
      <td><i>55.14</i></td>
    </tr>
  </tbody>
</table>

Fields with - or * symbols correspond to the models, which haven't converged and will be updated in the future.
