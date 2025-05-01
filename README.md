# AHCI


# AHCI Service Manual  
*A step-by-step guide to preparing data and running the **TailOR** mouse‑behavior pipeline with Facebook Research’s **SAM2** model.*

---

## 1. Overview  

This manual explains, from scratch, how to:

1. **Set up the environment** (Python, libraries, GPU drivers).  
2. **Install SAM2** and its dependencies.  
3. **Prepare video data** by sampling frames with FFmpeg.  
4. **Run the `mouse.ipynb` notebook** for segmentation and behavior tagging.  
5. **Troubleshoot** the most common pitfalls.

Everything is written for a Linux workstation (Ubuntu 20.04 +) but the same steps work on macOS and WSL with minor path changes.

---

## 2. System Requirements  

| Component | Minimum                         | Recommended                    |
|-----------|---------------------------------|--------------------------------|
| OS        | Ubuntu 20.04                    | Ubuntu 22.04                   |
| Python    | 3.9                             | 3.10 (conda)                   |
| GPU       | 6 GB VRAM (e.g., GTX 1660)      | 12 GB + (RTX 3060 / A100)      |
| CUDA      | 11.7+                           | 12.x                           |
| Disk      | 10 GB free                      | 30 GB free (checkpoints + data)|

> **Tip:** If no discrete GPU is available, SAM2 still runs on CPU but is ~10× slower.

---

## 3. Environment Setup  

### 3.1 Create an isolated Conda environment  

```bash
conda create -n tailOR python=3.10
conda activate tailOR
```

### 3.2 Install PyTorch (GPU build)  

```bash
# Check https://pytorch.org/get-started/locally/ for the exact command
conda install pytorch torchvision torchaudio pytorch-cuda=12.1 -c pytorch -c nvidia
```

If you are on CPU‑only hardware:

```bash
conda install pytorch torchvision torchaudio cpuonly -c pytorch
```

---

## 4. Install SAM2  

SAM2 is the *Segment Anything Model, v2*.

```bash
git clone https://github.com/facebookresearch/sam2.git
cd sam2
pip install -e .
```

The `-e` flag installs SAM2 in **editable** mode, so any local changes are picked up automatically.

If installation fails:

* **“torch not found”** → ensure you installed PyTorch first (Section 3.2).  
* **Compiler errors** → install build tools: `sudo apt install build-essential`.

---

## 5. Install FFmpeg  

FFmpeg extracts frames from videos without re‑encoding.

```bash
sudo apt update
sudo apt install ffmpeg
ffmpeg -version   # verify installation
```

---

## 6. Directory Layout  

```text
sam2/
 ├── notebooks/
 │    ├── mouse.ipynb        ← Main analysis notebook
 │    └── videos/
 │         └── 18/           ← One folder per video (sample name)
 │              └── 000001.jpg ...
 └── sam/                    ← SAM2 source code
```

*Keep one sub‑folder per video inside **`notebooks/videos`**. The folder name can be anything (here `18`).*

---

## 7. Frame Extraction Workflow  

1. **Create a folder** for the sampled frames:

   ```bash
   mkdir -p sam2/notebooks/videos/18
   ```

2. **Extract every 10th frame** (≈ 3 fps for a 30 fps file):

   ```bash
   ffmpeg -i input_video.mp4           -vf "select=not(mod(n\,10))"           -vsync vfr -q:v 2           sam2/notebooks/videos/18/%06d.jpg
   ```

   | Parameter | Meaning |
   |-----------|---------|
   | `select=not(mod(n\,10))` | Keep frames where *frame_number mod 10 ≠ 0*. |
   | `-vsync vfr` | Prevents FFmpeg from duplicating frames. |
   | `-q:v 2` | Visually lossless JPEG quality. |
   | `%06d.jpg` | Zero‑padded filenames for stable sorting. |

3. **Verify** extraction:

   ```bash
   ls sam2/notebooks/videos/18 | head
   eog sam2/notebooks/videos/18/000001.jpg
   ```

---

## 8. Running the `mouse.ipynb` Notebook  

1. Launch Jupyter:

   ```bash
   cd sam2/notebooks
   jupyter notebook
   ```

2. Open **`mouse.ipynb`**. The notebook is divided into five logical sections:

   | Section | Purpose |
   |---------|---------|
   | **A. Config** | Set paths, sampling rate, model checkpoint. |
   | **B. Load Frames** | Reads images from `videos/<folder>`. |
   | **C. Run SAM2** | Generates segmentation masks for tails, bodies, etc. |
   | **D. Post‑process** | Filters masks, smooths tracks, tags behaviors. |
   | **E. Export** | Saves tags as CSV & overlay videos for QC. |

3. **Execute cells** top‑to‑bottom (Shift + Enter).  
   *The first SAM2 inference warms up GPU and can take 10‑20 s.*

4. **Output artifacts** appear in `notebooks/outputs/<run_date>/`.

---


## 9. Support  

* **Code issues:** open an issue on the [SAM2 GitHub](https://github.com/facebookresearch/sam2/issues).   
* **Hardware problems:** check NVIDIA driver logs `dmesg | grep -i nvrm`.

---

**Enjoy streamlined mouse‑behavior tagging with TailOR + SAM2!**
