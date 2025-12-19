# HCGK Kernel - Quick Start Guide

## Installation

```bash
pip install hcgk-kernel
```

---

## Basic Usage

### Command Line

```bash
# Check hardware authorization
hcgk check

# View system information
hcgk info

# View configuration
hcgk config
```

### Python

```python
from hcgk_kernel import HCGKKernel

kernel = HCGKKernel()
authorized, message = kernel.authorize()

if authorized:
    # Proceed with model loading
    pass
```

---

## Common Use Cases

### 1. Pre-Load Validation

Validate hardware before attempting to load a model:

```python
from hcgk_kernel import HCGKKernel

def safe_model_load(model_path):
    kernel = HCGKKernel()
    authorized, message = kernel.authorize()
    
    if not authorized:
        raise RuntimeError(f"Cannot load model: {message}")
    
    return load_model(model_path)
```

### 2. Script Exit Codes

Use authorization status as script exit code:

```python
import sys
from hcgk_kernel import HCGKKernel

kernel = HCGKKernel(silent=True)
authorized, _ = kernel.authorize()
sys.exit(0 if authorized else 1)
```

### 3. Hardware Information

Retrieve detailed system information:

```python
from hcgk_kernel import HCGKKernel

kernel = HCGKKernel()
info = kernel.get_system_info()

print(f"RAM: {info['ram']['total_gb']}GB")
print(f"GPU: {info['gpu']['name']}")
```

### 4. Custom Requirements

Configure custom hardware requirements:

```python
from hcgk_kernel import HCGKKernel, HCGKConfig

config = HCGKConfig(
    MIN_RAM_GB=16.0,
    MIN_VRAM_GB=8.0
)

kernel = HCGKKernel(config=config)
authorized, message = kernel.authorize()
```

---

## Configuration

### Environment Variables

```bash
export HCGK_MIN_RAM_GB=8.0              # Minimum RAM for GPU mode
export HCGK_MIN_VRAM_GB=4.0             # Minimum VRAM
export HCGK_MIN_RAM_NO_GPU_GB=16.0      # Minimum RAM for CPU mode
export HCGK_RAM_SAFETY_MARGIN=0.1       # Reserve 10% RAM
```

### Programmatic Configuration

```python
from hcgk_kernel import HCGKConfig

config = HCGKConfig(
    MIN_RAM_GB=32.0,
    MIN_VRAM_GB=16.0,
    RAM_SAFETY_MARGIN=0.15,
    MAX_SCAN_RETRIES=3
)
```

---

## CLI Commands

### Authorization Check

```bash
# Basic check
hcgk check

# Silent mode (exit code only)
hcgk check --silent

# Verbose output
hcgk check --verbose
```

### System Information

```bash
# Human-readable format
hcgk info

# JSON format
hcgk info --json

# With requirements check
hcgk info --requirements
```

### Configuration

```bash
# View configuration
hcgk config

# JSON format
hcgk config --json
```

### Validation

```bash
# Validate configuration
hcgk validate
```

---

## Hardware Requirements

### GPU Mode
- RAM: 8 GB minimum (default)
- VRAM: 4 GB minimum (default)

### CPU Mode
- RAM: 16 GB minimum (default)

All requirements are configurable.

---

## API Reference

### HCGKKernel

```python
kernel = HCGKKernel(silent=False, config=None)
```

**Methods:**
- `authorize() -> Tuple[bool, str]` - Check hardware authorization
- `get_system_info() -> Dict` - Get detailed system information

### HCGKConfig

```python
config = HCGKConfig(
    MIN_RAM_GB=8.0,
    MIN_VRAM_GB=4.0,
    MIN_RAM_NO_GPU_GB=16.0,
    RAM_SAFETY_MARGIN=0.1,
    MAX_SCAN_RETRIES=2,
    STRICT_MODE=True
)
```

---

## Examples

### Example 1: Model Loading with Validation

```python
from hcgk_kernel import HCGKKernel
import torch

def load_model_safely(path):
    kernel = HCGKKernel()
    authorized, message = kernel.authorize()
    
    if not authorized:
        print(f"Hardware check failed: {message}")
        return None
    
    info = kernel.get_system_info()
    device = "cuda" if info['gpu']['available'] else "cpu"
    
    model = torch.load(path, map_location=device)
    print(f"Model loaded on {device}")
    return model

model = load_model_safely("model.pt")
```

### Example 2: Conditional Model Selection

```python
from hcgk_kernel import HCGKKernel

kernel = HCGKKernel()
info = kernel.get_system_info()

# Select model based on available resources
if info['gpu']['available'] and info['gpu']['vram_total_gb'] >= 16:
    model_name = "large-model"
elif info['ram']['total_gb'] >= 32:
    model_name = "medium-model"
else:
    model_name = "small-model"

print(f"Loading {model_name} based on available hardware")
```

### Example 3: Automated Script

```python
#!/usr/bin/env python3
import sys
from hcgk_kernel import HCGKKernel

def main():
    kernel = HCGKKernel()
    authorized, message = kernel.authorize()
    
    if not authorized:
        print(f"Error: {message}", file=sys.stderr)
        return 1
    
    # Proceed with main logic
    print("Hardware validated, proceeding...")
    return 0

if __name__ == "__main__":
    sys.exit(main())
```

### Example 4: Custom Validation Logic

```python
from hcgk_kernel import HCGKKernel, HCGKConfig

def validate_for_training():
    """Strict validation for model training."""
    config = HCGKConfig(
        MIN_RAM_GB=32.0,
        MIN_VRAM_GB=16.0,
        RAM_SAFETY_MARGIN=0.2,
        STRICT_MODE=True
    )
    
    kernel = HCGKKernel(config=config)
    return kernel.authorize()

def validate_for_inference():
    """Relaxed validation for model inference."""
    config = HCGKConfig(
        MIN_RAM_GB=8.0,
        MIN_VRAM_GB=4.0,
        STRICT_MODE=False
    )
    
    kernel = HCGKKernel(config=config)
    return kernel.authorize()
```

---

## Troubleshooting

### Import Error

**Problem:** `ModuleNotFoundError: No module named 'hcgk_kernel'`

**Solution:** Install the package:
```bash
pip install hcgk-kernel
```

### Command Not Found

**Problem:** `hcgk: command not found`

**Solution:** Ensure package is installed correctly:
```bash
pip install --force-reinstall hcgk-kernel
```

### Authorization Always Fails

**Problem:** Authorization consistently denied

**Solution:** Check system information and requirements:
```bash
hcgk info --requirements
```

Adjust configuration if needed:
```bash
export HCGK_MIN_RAM_GB=4.0
export HCGK_MIN_VRAM_GB=2.0
```

### Dependency Error

**Problem:** Missing psutil or torch

**Solution:** Install dependencies:
```bash
pip install psutil torch
```

---

## Best Practices

1. **Validate Early**: Check hardware before loading models
2. **Handle Failures**: Always check authorization result
3. **Configure Appropriately**: Set thresholds based on your models
4. **Use Silent Mode**: In scripts, use `silent=True` for cleaner output
5. **Check System Info**: Use `get_system_info()` for optimization decisions
6. **Log Authorization**: Enable logging for production deployments

---

## Next Steps

- Read the full [README](README.md) for detailed documentation
- Check the [API Reference](#api-reference) for all available methods
- Review [Examples](#examples) for common use cases
- Configure thresholds based on your specific models

---