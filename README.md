# A Handwritten Text Recognition Model for the Bosnian Language: Mixed Print and Cursive Scripts

## Introduction & Project Overview

## Workflow & Pipeline Structure
**1. Dataset preprocessing**
- The initial dataset consists of 156 different texts, each with an image of the full handwritten text (scanned at 300 DPI) and an XML file containing word labels (in **PascalVOC** format).
- Labels and images of individual words and lines of text are extracted, resulting in three datasets (_labels_w_, _labels_l_, and _labels_l_m_). All images are in JPG format, and all labels are stored in TXT files.
- The data is split into training (70%), validation (20%), and test (10%) partitions.
- **HDF5** files are then created from these datasets, following the previous partitioning.

## Sample Data & Formats
Three individual datasets are used, containing images and labels of:
- **words** (_labels_w_)
  
![EM_mix_002_2](https://github.com/user-attachments/assets/dcbf3189-e4e9-496b-9a72-a215f94f5a6d)

`EM_mix_002_2.jpg|2|2092|2928|462|36|654|140|ZNALI`

- **lines of text**, extracted using two different methods (_labels_l_ and _labels_l_m_)
  
![EM_mix_001_1](https://github.com/user-attachments/assets/d5551d52-665a-40d3-a14a-abd3e7c08ba4)

`EM_mix_001_1.jpg|1|2136|3012|120|196|811|391|ŠTA ĆEŠ ONDA ?`

and

`EM_mix_001_1.jpg|1|2136|3012|120|196|811|391|ŠTA ĆEŠ ONDA?`

The datasets are converted into **HDF5** format, following the instructions given in section _1.4 Kreiranje HDF5 formata dataset-a_ of the [notebook](HTR_bos3%20notebook.ipynb). These HDF5 files are used as inputs to the **HTR-flor** model.

Currently supported datasets for HDF5 file creation are the three mentioned above. In order to do the same for other datasets, additional methods would have to be added to the [reader.py](src/data/reader.py) file. For further understanding, it is recommended to analyze the existing methods and the provided [documentation](documentation/HTR_bos3%20dokumentacija.pdf).

## Model Training Process

## Evaluation Metrics & Results

## Model Comparison & Analysis

## Referenced repositories
The following repositories were used within the project:
- [handwritten-text-recognition](https://github.com/arthurflor23/handwritten-text-recognition)
- [masters-theses-htr](https://github.com/abegovac2/masters-theses-HTR)
