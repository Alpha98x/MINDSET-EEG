# EEG Conversion to BIDS

This repository provides a Python script to convert EEG data in BrainVision format (`.vhdr`, `.eeg`, `.vmrk`) into the [BIDS (Brain Imaging Data Structure)](https://bids.neuroimaging.io/) format.

---

## Overview

### Input Directory
- A folder (**`input_dir`**) containing your `.vhdr`, `.eeg`, and `.vmrk` files.
- Each EEG recording typically has these three files with the same base name, for example:
R078_Day1_25012018_Rest2.vhdr R078_Day1_25012018_Rest2.eeg R078_Day1_25012018_Rest2.vmrk
