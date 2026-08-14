# Third-party notices and attributions

SonicSight uses two published research models and their trained weights. This
file records what came from where, and what is still unresolved.

## Sound of Pixels

- **Source:** Zhao, H., Gan, C., Rouditchenko, A., Vondrick, C., McDermott, J.,
  & Torralba, A. (2018). *The Sound of Pixels.* ECCV 2018. MIT CSAIL.
- **Used:** the published checkpoints (frame network, sound network, and
  inner-product synthesiser), unmodified. No training or fine-tuning.
- **Trained on:** the MUSIC dataset — solo and duet clips across eleven
  musical instrument classes. This is the source of the model's domain limit.
- **Licence:**
```bibtex
    @InProceedings{Zhao_2018_ECCV,
        author = {Zhao, Hang and Gan, Chuang and Rouditchenko, Andrew and Vondrick, Carl and McDermott, Josh and Torralba, Antonio},
        title = {The Sound of Pixels},
        booktitle = {The European Conference on Computer Vision (ECCV)},
        month = {September},
        year = {2018}
    }
```

## Multisensory

- **Source:** Owens, A., & Efros, A. A. (2018). *Audio-Visual Scene Analysis
  with Self-Supervised Multisensory Features.* ECCV 2018. arXiv:1804.03641.
- **Used:** the published network, ported from Python 2 to Python 3 and run
  under the TensorFlow v1 compatibility layer inside TensorFlow 2.21. The
  fork is at <https://github.com/K1ller-Whale/multisensory>.
- **Modified:** yes — porting patches, an activation-map probe, a configurable
  session, a cuDNN workspace cap, and lazy imports to keep plotting packages
  out of the server. Each is documented in the fork.
- **Pretext task trained on:** approximately 750,000 clips from AudioSet
  (Gemmeke et al., ICASSP 2017). Separation head fine-tuned on VoxCeleb
  (Nagrani et al., Interspeech 2017).
- **Licence:** 
```
@article{multisensory2018,
  title={Audio-Visual Scene Analysis with Self-Supervised Multisensory Features},
  author={Owens, Andrew and Efros, Alexei A},
  journal={arXiv preprint arXiv:1804.03641},
  year={2018}
}
```

## Why the licence findings matter

Both repositories here are public, and the fork redistributes modified
upstream code. If either upstream project carries no licence, that is worth
knowing and stating rather than assuming permission. Recording "no licence
found" alongside full attribution is the honest position and costs nothing.

## Frameworks and libraries

The backend runs on PyTorch and TensorFlow; the client on Android, CameraX,
and Kotlin coroutines; the transport on gRPC and Protocol Buffers. Each is
used under its own licence, unmodified. Full citations are in the project
report bibliography.
