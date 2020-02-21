---
title: 'Spleeter: a fast and state-of-the art music source separation tool with pre-trained models'
tags:
  - Python
  - musical signal processing
  - source separation
  - vocal isolation
authors:
  - name:  Romain Hennequin
    orcid: 0000-0001-8158-5562
    affiliation: 1
  - name: Anis Khlif
    affiliation: 1
  - name: Felix Voituret
    affiliation: 1
  - name: Manuel Moussallam
    orcid: 0000-0003-0886-5423
    affiliation: 1
affiliations:
 - name: Deezer Research, Paris
   index: 1
date: 20 February 2020
bibliography: paper.bib

---

# Summary

We present and release a new tool for music source separation with pre-trained models called Spleeter. Spleeter was designed with ease of use, separation performance and speed in mind. Spleeter is based on Tensorflow [@tensorflow2015-whitepaper] and makes it possible to:

* separate audio files into $2$, $4$ or $5$ stems with a single command line using pre-trained models.
* train source separation models or fine-tune pre-trained ones with Tensorflow (provided you have a dataset of isolated sources).

The performance of the pre-trained models are very close to the published state of the art and is, to the authors knowledge, the best performing $4$ stems separation model on the common musdb18 benchmark [@musdb18] to be publicly released. Spleeter is also very fast as it can separate a mix audio file into $4$ stems $100$ times faster than real-time (We note, though, that the model cannot be applied in real-time as it needs buffering) on a single Graphics Processing Unit (GPU) using the pre-trained $4$-stems model. Spleeter is packaged within Docker which makes it usable as is on various platforms.


# Purpose
We release Spleeter with pre-trained state-of-the-art models in order to help the Music Information Retrieval (MIR) community leverage the power of source separation in various Music Information Retrieval (MIR) tasks, such as vocal lyrics analysis from audio (audio/lyrics alignement, lyrics transcription...), music transcription (chord transcription, drums transcription, bass transcription, chord estimation, beat tracking), singer identification, any type of multilabel classification (mood/genre...) or vocal melody extraction.
We believe that source separation has reached a level of maturity that makes it worth of consideration for these tasks and that specific features computed from isolated vocals, drums or bass may help increase performances, especially in low data availability scenarios (small datasets, limited annotation availability) for which supervised learning might be difficult.
Spleeter also makes it possible to fine tune the provided state-of-the-art models in order to adapt the system to a specific use-case.
Finally, having an available source separation tool such as Spleeter will allow researchers to compare performances of their new models to a state-of-the-art one on their own private datasets instead of musdb18, which is usually the only used dataset for reporting separation performances for unreleased models.
Note that we cannot release the training data for copyright reasons, and thus, sharing pre-trained models were the only way to make these results available to the community.


# Implementation details
Spleeter contains pre-trained models for:

  * vocals/accompaniment separation.
  * $4$ stems separation as in SiSec [@SISEC18]  (vocals, bass, drums and other).
  * $5$ stems separation with an extra piano stem (vocals, bass, drums, piano and other). It is, to the authors knowledge, the first released model to perform such a separation.

The pre-trained models are U-nets [@unet2017] and follows similar specifications as in [@deezerICASSP2019]. The U-net is a encoder/decoder Convolutional Neural Network (CNN) architecture with skip connections. We used $12$-layer U-nets ($6$ layers for the encoder and $6$ for the decoder). A U-net is used for estimating a soft mask for each source (stem). Training loss is a $L_1$-norm between masked input mix spectrograms and source target spectrograms. The models were trained on Deezer internal datasets (noteworthily the Bean dataset that was used in [@deezerICASSP2019]) using Adam [@Adam]. Training time took approximately a full week on a single GPU. Separation is then done from estimated source spectrograms using soft masking or multi-channel Wiener filtering.

Training and inference is implemented in Tensorflow which makes it possible to run the code on Central Processing Unit (CPU) or GPU.



# Speed
As the whole separation pipeline can be run on a GPU and the model is based on a CNN (which makes computation parallelization very efficient), model inference is very fast. For instance, Spleeter is able to separate the whole musdb18 test dataset (about $3$ hours and $27$ minutes of audio) into $4$ stems in less than $2$ minutes, including model loading time (about $15$ seconds), and audio wav files export, using a single GeForce RTX 2080 GPU, and a double Intel Xeon Gold 6134 CPU @ 3.20GHz (CPU is used for mix files loading and stem files export only). Spleeter is then able to separate into $4$ stems $100$ seconds of stereo audio in less than $1$ second, which makes it very useful for efficiently processing large datasets.


# Separation performances
The models compete with the state of the art on the standard musdb18  dataset [@musdb18] while it was not trained, validated or optimized in any way with musdb18 data. We report results in terms of standard source separation metrics [@separation_metrics], namely Signal to Distorition Ration (SDR), Signal to Artifacts Ratio (SAR), Signal to Interference Ration (SIR) and source Image to Spatial distortion Ratio (ISR), are presented in the following table compared to Open-Unmix [@Open-Unmix] which is, to the authors knowledge, the only released system that performs near state-of-the-art performances.
We present results for soft masking and for multi-channel Wiener filtering (applied using Norbert [@Norbert]). As can be seen, for most metrics Spleeter is competitive with Open-Unmix and especially on SDR for all instruments.


|           |Spleeter Mask  |Spleeter MWF   |Open-Unmix |
|-----------|---------------|---------------|-----------|
|-----------|---------------|---------------|-----------|
| Vocals SDR|6.55           |__6.86__       |6.32       |
| Vocals SIR|15.19          |**15.86**      |13.33      |
| Vocals SAR|6.44           |**6.99**       |6.52       |
| Vocals ISR|**12.01**      |11.95          |11.93      |
| Bass SDR  |5.10           |**5.51**       |5.23       |
| Bass SIR  |10.01          |10.30          |**10.93**  |
| Bass SAR  |5.15           |5.96           |**6.34**   |
| Bass ISR  |9.18           |**9.61**       |9.23       |
| Drums SDR |5.93           |**6.71**       |5.73       |
| Drums SIR |12.24          |**13.67**      |11.12      |
| Drums SAR |5.78           |**6.54**       |6.02       |
| Drums ISR |10.50          |**10.69**      |10.51      |
| Other SDR |4.24           |**4.55**       |4.02       |
| Other SIR |7.86           |**8.16**       |6.59       |
| Other SAR |4.63           |**4.88**       |4.74       |
| Other ISR |9.83           |**9.87**       |9.31       |


Spleeter is available at <https://www.github.com/deezer/spleeter> with a MIT license. This repository will be possibly used for releasing other models with improved performances or models separating into more than $5$ stems in the future.


# Acknowledgements

We acknowledge contributions from Laure Pretet that trained first models and write the first piece of code that lead to Spleeter.

# References
