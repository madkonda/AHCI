
# User Manual  
*A step-by-step guide to preparing data and running the **TailOR** mouse‑behavior pipeline with Facebook Research’s **SAM2** model.*

---

## 1. Overview  

This manual explains, from scratch, how to:

1. **Set up the environment** (Python, libraries, GPU drivers).  
2. **Install SAM2** and its dependencies.  
3. **Prepare video data** by downloading and sampling frames with FFmpeg.  
4. **Run the `mouse.ipynb` notebook** for segmentation and behavior tagging.  
5. **Troubleshoot** the most common pitfalls.

Everything is written for a Linux workstation (Ubuntu 20.04 +) but the same steps work on macOS and WSL with minor path changes.

---

## 2. System Requirements  

| Component | Minimum | Recommended |
|-----------|---------|-------------|
| Python | 3.9 | 3.10 (conda) |
| GPU | 6 GB VRAM (e.g., GTX 1660) | 12 GB + (RTX 3060 / A100) |
| CUDA | 11.7+ | 12.x |
| Disk | 10 GB free | 30 GB free (checkpoints + data)|

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

```bash
git clone https://github.com/facebookresearch/sam2.git
cd sam2
pip install -e .
```

If installation fails:

* “torch not found” → ensure you installed PyTorch first (Section 3.2).  
* Compiler errors → `sudo apt install build-essential`.

---

## 5. Install FFmpeg  

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
 │    ├── mouse.ipynb
 │    └── videos/
 │         └── 18/
 │              └── 000001.jpg ...
 └── sam/
```

*Keep one sub‑folder per video inside **`notebooks/videos`**. The folder name can be anything (here `18`).*

---

## 7. Data Preparation Workflow  

### 7.1 Download Example Video  

Download the sample video from Google Drive:  
<https://drive.google.com/file/d/1ypCVgHTH36CsVKEpXgjpauxfQwncaiEr/view?usp=sharing>

Save it as `input_video.mp4` inside `sam2/notebooks/` (or adjust the path below).

### 7.2 Create a folder for sampled frames  

```bash
mkdir -p sam2/notebooks/videos/18
```

### 7.3 Extract every 10th frame  

```bash
ffmpeg -i input_video.mp4        -vf "select=not(mod(n\,10))"        -vsync vfr -q:v 2        sam2/notebooks/videos/18/%06d.jpg
```

| Parameter | Meaning |
|-----------|---------|
| `select=not(mod(n\,10))` | Keep frames where *frame_number mod 10 ≠ 0*. |
| `-vsync vfr` | Prevents FFmpeg from duplicating frames. |
| `-q:v 2` | Visually lossless JPEG quality. |
| `%06d.jpg` | Zero‑padded filenames for stable sorting. |

Verify extraction:

```bash
ls sam2/notebooks/videos/18 | head
```

---

## 8. Running the `mouse.ipynb` Notebook  

```bash
cd sam2/notebooks
jupyter notebook
```

Open **`mouse.ipynb`** and run the cells top‑to‑bottom (Shift + Enter).  
The notebook sections are:

| Section | Purpose |
|---------|---------|
| **A. Config** | Set paths, sampling rate, model checkpoint. |
| **B. Load Frames** | Reads images from `videos/<folder>`. |
| **C. Run SAM2** | Generates segmentation masks. |
| **D. Post‑process** | Filters masks, smooths tracks, tags behaviors. |
| **E. Export** | Saves tags as CSV & overlay videos for QC. |

First inference warms up the GPU and may take 10‑20 s.

Outputs appear in `notebooks/outputs/<run_date>/`.

---

## 9. Support  

* Code issues: <https://github.com/facebookresearch/sam2/issues>  
* Hardware problems: `dmesg | grep -i nvrm`

---

**Enjoy streamlined mouse‑behavior tagging with TailOR + SAM2!**
