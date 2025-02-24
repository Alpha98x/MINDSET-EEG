# EEG Conversion to BIDS

This repository provides a Python script to convert EEG data in BrainVision format (`.vhdr`, `.eeg`, `.vmrk`) into the [BIDS (Brain Imaging Data Structure)](https://bids.neuroimaging.io/) format.

---

## Overview

### Input Directory
- A folder (**`input_dir`**) containing your `.vhdr`, `.eeg`, and `.vmrk` files.
- Each EEG recording typically has these three files with the same base name, for example:
  
R078_Day1_25012018_Rest2.vhdr R078_Day1_25012018_Rest2.eeg R078_Day1_25012018_Rest2.vmrk


### Output Directory
- A folder (**`output_dir`**) where the script will generate a BIDS-compliant directory structure.  
- After the conversion, you’ll have a structure like:

output_dir/

└── sub-<SubjectID>/

  └── ses-<MRID>/
  
    └── eeg/ 
    
      ├── sub-<SubjectID>_ses-<MRID>_task-<Task>_run-01_eeg.vhdr 
      
      ├── sub-<SubjectID>_ses-<MRID>_task-<Task>_run-01_eeg.eeg 
      
      ├── sub-<SubjectID>_ses-<MRID>_task-<Task>_run-01_eeg.vmrk
      
      └── sub-<SubjectID>_ses-<MRID>_task-<Task>_run-01_eeg.json


### Excel Sheets

**1. File-to-MR Mapping**  
- First Excel file (e.g., `File-MR_ID.xlsx`) maps a short code (like `R078`) to an **MR ID**.  
- Columns expected:
  - **FileName** (e.g., `R078`)
  - **MR ID** (e.g., `MR0034`)

**2. MR-to-Subject Mapping**  
- Second Excel file (e.g., `MR_ID-subject.xlsx`) maps the **MR ID** (e.g., `MR0034`) to a **Subject ID** (the patient’s ID).  
- Columns expected:
  - **MR ID** (e.g., `MR0034`)
  - **Subject ID** (e.g., `SUB017`)

> **Subject ID** = the Patient ID (in BIDS: `sub-<SubjectID>`)  
> **Session ID** = the MR ID (in BIDS: `ses-<MRID>`)

---

## Why Task and Run?

- **Task**: Derived from the filename (e.g., `Rest2`, `Reward`, `Startle`). It appears as `task-<Task>` in the final BIDS filenames.
- **Run**: If you have **multiple experiments** or repeated runs for the same subject (e.g., `Day1` and `Day2`), you can use the `run` number to differentiate them (e.g., `run-01`, `run-02`).

---

## How the Script Works

1. **Parse Filenames**  
 The script splits each `.vhdr` filename to extract:
 - A **file code** (e.g., `R078`) used to look up the MR ID in Excel.
 - A **task** (e.g., `Rest2`) from the last part of the file name.
 - A fixed **run** (e.g., `01`) or something determined by other parts of the file name if desired.

2. **Excel Lookups**  
 - **Step 1**: `file_code` → **MR ID** (from `File-MR_ID.xlsx`).  
 - **Step 2**: **MR ID** → **Subject ID** (from `MR_ID-subject.xlsx`).  
 This way, each file is associated with the correct patient (Subject ID) and session (MR ID).

3. **MNE-BIDS Conversion**  
 - [MNE](https://mne.tools/) is used to read each `.vhdr` file.  
 - [mne-bids](https://mne.tools/mne-bids/stable/) exports the data into a BIDS-valid folder structure.  
 - A JSON “sidecar” file is generated/updated with additional metadata like sampling frequency, channel counts, and filter settings.

---

## Dependencies

Make sure you have the following Python packages installed:
```bash
pip install mne mne-bids pandas
```
---
 
mne: Reading .vhdr (BrainVision) files.
mne-bids: Writing BIDS-compliant structures.
pandas: Reading Excel spreadsheets.

## Usage
### important 
this is the example for the RACLO study. ABP studies and FMZ Scripts are sligltly different from this script. If Subject data and MR ID is available ABP script can be used.

1.Download the files from r'/home/projects/multimodal/rlochana/RACLO'.

2.Update the script with your paths:

---
In convert_to_bids.py (example)

input_dir       = r'/home/MR/mr_user/TRIMODAL_Team_Data/PET_MR_EEG_Studies/RACLO'

output_dir      = r'/home/projects/mindset/RACLO'

file_map_excel  = r'/home/projects/multimodal/rlochana/test_output_dir/File-MR_ID.xlsx'

sub_map_excel   = r'/home/projects/multimodal/rlochana/test_output_dir/MR_ID-subject.xlsx'

3.Run the script
python convert_to_bids.py

4.Check your output_dir to see the BIDS-formatted files.

Multiple Runs or Sessions
If you have multiple EEG recordings (like Day1 and Day2) for the same subject, you can tweak:

run logic in the script to assign run-01, run-02 based on “Day1”/“Day2”.
session if “Day1” is actually a different session from the MR ID.
By default, we keep:

subject = Subject ID (Patient ID)
session = MR ID
run = "01" (fixed, but you can easily adapt).

