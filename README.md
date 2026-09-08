<div align="center">

# XSpecCorr
### Cross-Spectral Dense Correspondence for Multimodal Spectral Medical Imaging

**Eric L. Wisotzky · Jost Triller · Simon W. Härtl · Oliver T. Bruns · Peter Eisert · Anna Hilsmann**

**2nd Data Curation & Augmentation in Medical Imaging Workshop · ECCV 2026**

[Paper](https://arxiv.org/abs/2608.28341) | [Generator](synth/README.md) | [Datasets](#data) | [Pretrained models](#pretrained-models) | [Citation](#citation)

</div>

Dense correspondence across wavelengths: synthetic ground-truth data and cross-spectral training for robust multimodal spectral image alignment.

XSpecCorr studies dense pixel-wise correspondence between images acquired at different wavelengths. Cross-spectral training exposes existing correspondence networks to material-dependent appearance changes and intensity inversions while preserving geometric supervision. The planned release brings together the **Synth dataset generator**, **our datasets**, and **trained cross-spectral models**.

## Release status

This repository is being prepared for release. The status below describes this checkout.

| Component | Availability |
|---|---|
| Synthetic dataset generator | Source in [`synth/`](synth/); paper configuration reconciliation pending |
| Synth benchmark archives | Pending; no download published here yet |
| Own real cross-spectral acquisitions | Pending; release scope and files to be supplied |
| Cross-spectral model weights | Pending for RAFT, DIP, GMA, SEA-RAFT and SKFlow |
| Training augmentation and evaluation code | Not yet included |
| Software license | [Fraunhofer academic-use license](LICENSE) |
| Data and model terms | To be specified with the artifact releases |

See the [publication checklist](docs/release-plan.md) for the remaining work and known differences between the imported generator and the paper.

## Setup

Use Python 3.10 or newer and `uv`.

1. Clone the repository:

   ```bash
   git clone https://github.com/fraunhoferhhi/XSpecCorr.git
   cd XSpecCorr
   ```

2. Create the generator environment:

   ```bash
   cd synth
   uv sync --locked
   ```

The following generation commands assume `synth/` is the working directory. Detailed options and output formats are described in the [generator documentation](synth/README.md).

## How to use

### Generate cross-spectral pairs

Download and extract `usgs_splib07.zip` from the [USGS source archive](https://www.sciencebase.gov/catalog/item/5807a2a2e4b0841e59e3a18d). Then replace the raw-data path below with your extracted `ASCIIdata` directory:

```bash
uv run python genflow.py --raw-folder /path/to/usgs_splib07/ASCIIdata --out-dir ./synthetic_flow_samples --bands Visible,NIR --samples-per-combination 1 --global-seed 0
```

This requests eight samples: two motion models × two band directions × two channel modes. Generation can return fewer samples if material sampling fails within its retry budget. Always use a fresh output directory: the current script can overwrite existing sample files.

The first run builds `synth/usgs.db` from the external raw data. Images and Middlebury `.flo` files are written below the output directory, with text indexes for selecting subsets. See the [generator documentation](synth/README.md) for options and the flow convention.

## Synth benchmark

Synth separates geometric displacement from wavelength-dependent appearance variation. Layered polygon scenes use measured material reflectance spectra from the USGS Spectral Library Version 7. Virtual cameras integrate different spectral intervals, producing paired images and dense displacement ground truth. Synth is a controlled correspondence benchmark, not a tissue simulator.

The paper describes images at **768 × 512 pixels**, six layered polygonal objects, and **100 samples per motion regime and spectral range**:

| Regime | Geometry |
|---|---|
| Synth-Zero | No displacement; tests false motion induced by appearance changes |
| Synth-Perspective | Depth-dependent parallax |
| Synth-Extreme | Smooth object-dependent non-rigid displacement |

The evaluated spectral pairings are LWIR–MWIR, LWIR–SWIR, MWIR–SWIR, NIR–SWIR, NIR–UV, NIR–VIS, SWIR–UV, SWIR–VIS and UV–VIS.

**The current generator defaults are not the frozen paper benchmark configuration.** It generates `unrestrained` and `perspective` motion, requests one to six polygons, defaults to 30 samples per combination, and includes both band directions and two channel modes. A Zero CLI preset is not yet included.

## Data

**Synth:** benchmark archives, exact generation settings, splits and checksums are pending.

**Own acquisitions:** the intended release will be documented separately from Synth. The paper evaluates VIS–NIR MSI stereo, RGB–SWIR imaging and hyperspectral light-field data qualitatively; these acquisitions do not provide dense correspondence GT. The exact subset to be distributed has not yet been specified in this repository.

Download links, acquisition metadata, preprocessing, licenses and archive checksums will be added when the artifacts are available.

## Pretrained models

| Architecture | Cross-spectral checkpoint | Inference/configuration |
|---|---|---|
| RAFT | Pending | Pending |
| DIP | Pending | Pending |
| GMA | Pending | Pending |
| SEA-RAFT | Pending | Pending |
| SKFlow | Pending | Pending |

Each release will need its upstream code revision, single-channel input adaptation, normalization, training configuration, license and checksum. No inference command is provided until the corresponding implementation and checkpoint are available.

## Reproduce the results

Table 4 of the [paper](https://arxiv.org/abs/2608.28341) reports the following end-point errors (EPE in pixels, lower is better), using preprocessed inputs. These are published results, not measurements reproduced from this checkout.

| Architecture | Training | Synth-Extreme | Synth-Perspective | Synth-Zero |
|---|---|---:|---:|---:|
| RAFT | Original | 53.107 | 40.682 | 16.423 |
| RAFT | **Cross-spectral** | **7.389** | **6.054** | **0.108** |
| DIP | Original | 51.521 | 36.628 | 13.214 |
| DIP | **Cross-spectral** | **8.491** | **7.757** | **0.015** |
| GMA | Original | 67.632 | 53.030 | 27.697 |
| GMA | **Cross-spectral** | **9.305** | **7.097** | **0.076** |
| SEA-RAFT | Original | 100.450 | 57.816 | 27.707 |
| SEA-RAFT | **Cross-spectral** | **6.653** | **4.389** | **0.018** |
| SKFlow | Original | 44.515 | 35.621 | 13.297 |
| SKFlow | **Cross-spectral** | **6.182** | **4.620** | **0.164** |

Evaluation must exclude invalid/occluded GT pixels. The generator encodes them as `2e9` in the flow file; the provided reader returns a validity mask. Evaluation scripts and the full paper aggregation protocol remain to be released.

## Training

The paper combines a normalized single-channel input representation with view-dependent RGB channel selection and structured nonlinear intensity mappings, including inversions, square-root, power-law and logarithmic functions. These alter the radiometric relationship while retaining geometric supervision.

The training implementation and architecture adaptations are not yet included in this checkout.

## Repository contents

```text
README.md               Project overview and artifact status
CITATION.cff            Machine-readable paper citation
LICENSE                 Fraunhofer academic-use software license
synth/                  Dataset generation and visualization source
  README.md             Generator usage and output format
  pyproject.toml        Python dependencies
  uv.lock               Locked dependency resolution
docs/release-plan.md    Publication checklist and reproducibility gaps
docs/license-provenance.md  License source and adaptation record
```

## License

The Fraunhofer-provided XSpecCorr software is distributed under the [Software Copyright License for Academic Use of XSpecCorr, Version 2.0](LICENSE). It permits internal non-commercial evaluation, testing and academic research. Commercial use requires another license from Fraunhofer; contact details are in the license.

Dataset and pretrained-weight terms will be stated with their releases. Third-party software and materials, including the USGS source data and upstream model implementations, retain their own applicable terms.

## Acknowledgements

We thank **Tom Runia** for [OpticalFlow Visualization](https://github.com/tomrunia/OpticalFlow_Visualization). Our [`synth/flow_viz.py`](synth/flow_viz.py) is copied from the project's [`flow_vis/flow_vis.py`](https://github.com/tomrunia/OpticalFlow_Visualization/blob/master/flow_vis/flow_vis.py), distributed under the [MIT License](https://github.com/tomrunia/OpticalFlow_Visualization/blob/master/LICENSE.txt). The original author and license notices are retained.

This work was funded by the **BMFTR – Federal Ministry of Research, Technology and Space**, project **REFRAME**, grant **01IS2407A**. We acknowledge the U.S. Geological Survey for the measured reflectance spectra.

## Citation

```bibtex
@inproceedings{wisotzky2026xspec,
  title     = {Cross-Spectral Dense Correspondence for Multimodal Spectral Medical Imaging},
  author    = {Wisotzky, Eric L. and
               Triller, Jost and
               H{\"a}rtl, Simon W. and
               Bruns, Oliver T. and
               Eisert, Peter and
               Hilsmann, Anna},
  booktitle = {ECCV 2026 Workshop on Data Curation \& Augmentation in Medical Imaging},
  year      = {2026}
}
```

Preprint: [arXiv:2608.28341](https://arxiv.org/abs/2608.28341). Final proceedings metadata will be added when available. GitHub-readable metadata is provided in [CITATION.cff](CITATION.cff).

Please also cite **Kokaly et al., USGS Spectral Library Version 7 (2017)** when using spectral-library-derived data.

## Contact

For questions, open an issue or contact **Eric L. Wisotzky**, Fraunhofer Heinrich Hertz Institute HHI, Vision & Imaging Technologies, Berlin, Germany.
