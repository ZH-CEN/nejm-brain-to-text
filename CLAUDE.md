# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This repository contains the code and data for "An Accurate and Rapidly Calibrating Speech Neuroprosthesis" published in the New England Journal of Medicine (2024). It implements a brain-to-text system that converts neural signals from speech motor cortex into text using RNN models and n-gram language models.

## Development Environment Setup

### Main Environment (b2txt25)
```bash
./setup.sh
conda activate b2txt25
```

### Language Model Environment (b2txt25_lm)
```bash
./setup_lm.sh
conda activate b2txt25_lm
```

**Important**: The project requires two separate conda environments due to conflicting PyTorch versions:
- `b2txt25`: PyTorch with CUDA 12.6 for model training/evaluation
- `b2txt25_lm`: PyTorch 1.13.1 for Kaldi-based n-gram language models

### Redis Setup
Redis is required for inter-process communication. Install on Ubuntu:
```bash
curl -fsSL https://packages.redis.io/gpg | sudo gpg --dearmor -o /usr/share/keyrings/redis-archive-keyring.gpg
echo "deb [signed-by=/usr/share/keyrings/redis-archive-keyring.gpg] https://packages.redis.io/deb $(lsb_release -cs) main" | sudo tee /etc/apt/sources.list.d/redis.list
sudo apt-get update && sudo apt-get install redis
sudo systemctl disable redis-server
```

## Architecture Overview

### High-Level System Flow
1. **Neural Data Input**: 512 features (2 per electrode × 256 electrodes) binned at 20ms resolution
2. **RNN Model**: Converts neural features to phoneme logits via CTC loss
3. **Language Model**: Decodes phoneme logits to words using n-gram models + OPT rescoring
4. **Redis Communication**: Coordinates between RNN inference and language model processes

### Key Components

#### Model Training (`model_training/`)
- **Core Script**: `train_model.py` (loads config from `rnn_args.yaml`)
- **Model Architecture**: `rnn_model.py` - 5-layer GRU with 768 hidden units
- **Trainer**: `rnn_trainer.py` - Custom PyTorch trainer with CTC loss
- **Evaluation**: `evaluate_model.py` - Inference pipeline with Redis communication

#### Language Model (`language_model/`)
- **Standalone Server**: `language-model-standalone.py` - Redis-based LM server
- **Kaldi Integration**: Uses custom C++ bindings for efficient n-gram decoding
- **OPT Rescoring**: Facebook OPT 6.7B for language model rescoring
- **Build System**: Complex CMake-based build for Kaldi/SRILM integration

#### Utilities (`nejm_b2txt_utils/`)
- **General Utils**: `general_utils.py` - Shared utility functions
- **Package**: Installed via `setup.py` as `nejm_b2txt_utils`

#### Analysis (`analyses/`)
- **Jupyter Notebooks**: `figure_2.ipynb`, `figure_4.ipynb` for paper figures

## Common Development Tasks

### Training a Model
```bash
conda activate b2txt25
cd model_training
python train_model.py
```

### Running Evaluation Pipeline
1. Start Redis server:
   ```bash
   redis-server
   ```

2. Start language model (separate terminal):
   ```bash
   conda activate b2txt25_lm
   python language_model/language-model-standalone.py --lm_path language_model/pretrained_language_models/openwebtext_1gram_lm_sil --do_opt --nbest 100 --acoustic_scale 0.325 --blank_penalty 90 --alpha 0.55 --redis_ip localhost --gpu_number 0
   ```

3. Run evaluation (separate terminal):
   ```bash
   conda activate b2txt25
   cd model_training
   python evaluate_model.py --model_path ../data/t15_pretrained_rnn_baseline --data_dir ../data/hdf5_data_final --eval_type test --gpu_number 1
   ```

4. Shutdown Redis:
   ```bash
   redis-cli shutdown
   ```

### Building Language Model from Scratch
```bash
# Build SRILM (in language_model/srilm-1.7.3/)
export SRILM=$PWD
make MAKE_PIC=yes World

# Build Kaldi components (in language_model/runtime/server/x86/)
mkdir build && cd build
cmake .. && make -j8
```

## Data Structure

### Neural Data Format
- **File Type**: HDF5 files in `data/hdf5_data_final/`
- **Features**: 512 neural features per 20ms bin:
  - 0-64: ventral 6v threshold crossings
  - 65-128: area 4 threshold crossings
  - 129-192: 55b threshold crossings
  - 193-256: dorsal 6v threshold crossings
  - 257-320: ventral 6v spike band power
  - 321-384: area 4 spike band power
  - 385-448: 55b spike band power
  - 449-512: dorsal 6v spike band power

### Data Loading
Use `load_h5py_file()` in `model_training/evaluate_model_helpers.py` as reference for HDF5 data loading.

## Important Notes

- **GPU Requirements**: OPT 6.7B requires ~12.4GB VRAM; RTX 4090s recommended
- **Memory Requirements**: 3-gram LM needs ~60GB RAM, 5-gram needs ~300GB RAM
- **Environment Isolation**: Always use correct conda environment for each component
- **Redis Dependency**: Many scripts require Redis server to be running
- **Build Dependencies**: CMake ≥3.14 and GCC ≥10.1 required for language model builds

## Competition Context
This codebase also serves as baseline for the Brain-to-Text '25 Competition on Kaggle, providing reference implementations for neural signal decoding.