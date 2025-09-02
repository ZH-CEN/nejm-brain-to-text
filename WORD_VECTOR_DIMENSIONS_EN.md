# Word Vector Dimensions Documentation

## Overview

This brain-to-text neural prosthesis system uses multiple vector representations with different dimensionalities to process the conversion from brain signals to text output.

## Word Vector Dimensions Used in the System

### 1. Neural Signal Features
- **Dimensions**: 512-dimensional
- **Description**: Neural features extracted from speech motor cortex
- **Composition**: 
  - 256 electrodes × 2 features per electrode (threshold crossings and spike band power)
  - Total: 512 features
- **Configuration**: `n_input_features: 512` in `model_training/rnn_args.yaml`

### 2. RNN Hidden States
- **Dimensions**: 768-dimensional
- **Description**: Hidden state dimensions of the GRU neural decoder
- **Purpose**: Converts neural signal features to phoneme predictions
- **Configuration**: `n_units: 768` in `model_training/rnn_args.yaml`

### 3. Language Model Word Embeddings
- **Dimensions**: 4096-dimensional
- **Model**: Facebook OPT-6.7B
- **Description**: Pre-trained Transformer model used for language modeling and text rescoring
- **Vocabulary Size**: 50,272 tokens
- **Purpose**: Rescores and optimizes candidate sentences generated from phoneme sequences

## System Architecture Pipeline

```
Brain Signals (512-dim) → RNN Decoder (768-dim hidden states) → Phoneme Sequences → N-gram LM → OPT-6.7B Rescoring (4096-dim embeddings) → Final Text
```

## Configuration File Locations

1. **Neural Network Model**: `model_training/rnn_args.yaml`
2. **Language Model**: `language_model/language-model-standalone.py`

## Technical Specifications

### RNN Model Parameters
- Input dimensions: 512 (neural features)
- Hidden dimensions: 768
- Number of layers: 5 GRU layers
- Output classes: 41 (phoneme categories)
- Dropout rates: 0.4 (RNN), 0.2 (input layer)

### OPT-6.7B Language Model Parameters
- Word embedding dimensions: 4096
- Vocabulary size: 50,272
- Model parameters: 6.7 billion
- Attention heads: 32
- Hidden layers: 32
- VRAM requirement: ~12.4 GB for inference

## Model Performance Context

The system achieves:
- High accuracy neural decoding through 768-dimensional hidden state representations
- Enhanced text generation via 4096-dimensional word embeddings in OPT-6.7B
- Real-time performance suitable for assistive communication applications

## References

Card, N.S., Wairagkar, M., Iacobacci, C., et al. (2024). An Accurate and Rapidly Calibrating Speech Neuroprosthesis. *The New England Journal of Medicine*.