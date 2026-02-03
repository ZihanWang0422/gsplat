# gsplat

[http://www.gsplat.studio/](http://www.gsplat.studio/)


## Installation

### Prerequisites

- **Python 3.10+**
- **CUDA 11.8 or 12.8** (recommended)
- **Conda** (for environment management)

### Step-by-Step Installation

1. **Create a Conda Environment**:
   ```bash
   conda create -n gsplat python=3.10
   conda activate gsplat
   ```

2. **Install PyTorch** (compatible with your CUDA version):
   For CUDA 12.8:
   ```bash
   pip install torch torchvision "numpy<2.0.0"
   ```

3. **Install Build Dependencies**:
   ```bash
   pip install scikit-build-core ninja cmake
   ```

4. **Install gsplat from Source**:
   ```bash
   cd gsplat
   pip install . --no-build-isolation
   cd examples
   ```

5. **Install Example Dependencies**:
   ```bash
   pip install -r requirements.txt --no-build-isolation
   ```

   Note: If you encounter issues with `fused-ssim`, install it separately:
   ```bash
   pip install "git+https://github.com/rahul-goel/fused-ssim@328dc9836f513d00c4b5bc38fe30478b4435cbb5" --no-build-isolation
   ```

6. **Fix pycolmap Compatibility** (Crucial):
   The default `pycolmap` dependency logic is outdated for Python 3 (where `map` returns an iterator). You must manually patch and reinstall it to avoid runtime errors (e.g., `Input quaternion should be a 3- or 4-vector`):

   ```bash
   # 1. Clone the specific version
   git clone https://github.com/rmbrualla/pycolmap.git pycolmap_fix
   cd pycolmap_fix
   git checkout cc7ea4b7301720ac29287dbe450952511b32125e

   # 2. Manually edit pycolmap/scene_manager.py
   # You need to wrap map(...) calls with list(...) in load_images() and load_points3D() methods.
   # For example, change: 
   #   map(float, data[1:5]) 
   # to:
   #   list(map(float, data[1:5]))
   
   # 3. Install the patched version
   pip install .
   cd ..
   ```


## Running the Examples

### Prepare Dataset

The dataset should be in COLMAP format. Place your raw photos and COLMAP outputs in `/home/wzh/colmap/gsplat/data`.

### Run Training

To train a 3D Gaussian Splatting model:

```bash
conda activate gssplat1
cd gsplat/examples
CUDA_VISIBLE_DEVICES=0 python simple_trainer.py default \
    --data_dir /home/wzh/colmap/gsplat/data \
    --data_factor 1 \
    --result_dir ./results/dataset
```

- `--data_dir`: Path to the COLMAP dataset directory.
- `--data_factor`: Downsampling factor (1 for full resolution).
- `--result_dir`: Directory to save results.

The training will run for 30,000 iterations by default. You can monitor progress via the web interface at `http://localhost:8080`.

### Evaluation

After training, evaluate the model:

```bash
python -c "
import torch
from gsplat import rasterize_gaussians
# Add your evaluation code here
"
```

For batch evaluation on benchmarks:

```bash
bash benchmarks/basic.sh
```

## Examples

We provide a set of examples to get you started!

- [Train a 3D Gaussian splatting model on a COLMAP capture.](https://docs.gsplat.studio/main/examples/colmap.html)
- [Fit a 2D image with 3D Gaussians.](https://docs.gsplat.studio/main/examples/image.html)
- [Render a large scene in real-time.](https://docs.gsplat.studio/main/examples/large_scale.html)

