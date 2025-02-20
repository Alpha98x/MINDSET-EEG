# MINDSET-EEG
This repository provides a Python script for converting EEG data (in BrainVision format: .vhdr, .eeg, .vmrk) into the BIDS (Brain Imaging Data Structure) format.

Overview
Input Directory
A folder (input_dir) containing subfolders/files of BrainVision EEG recordings. Each recording is typically composed of three files sharing the same base name:
<recording>.vhdr (header file)
<recording>.eeg (continuous data)
<recording>.vmrk (marker events)
Output Directory
A target folder (output_dir) where the script will produce a BIDS-compliant directory structure. The final structure will look like:

lua
Copy
Edit
bids_output/
└── sub-XXXX/
    └── ses-YYYY/
        └── eeg/
            ├── sub-XXXX_ses-YYYY_task-TASK_run-01_eeg.eeg
            ├── sub-XXXX_ses-YYYY_task-TASK_run-01_eeg.vhdr
            ├── sub-XXXX_ses-YYYY_task-TASK_run-01_eeg.vmrk
            └── sub-XXXX_ses-YYYY_task-TASK_run-01_eeg.json
Excel Sheets
Excel Sheet #1: Maps a short file code (e.g., R078) to the MR ID.

Must have columns:
FileName (e.g., R078)
MR ID (e.g., MR0034)
Excel Sheet #2: Maps the MR ID (e.g., MR0034) to the Subject ID (the patient or participant ID, e.g., SUB017).

Must have columns:
MR ID
Subject ID
Note: In BIDS, we often label participants as sub-XXXX. In this script, the Subject ID becomes the subject in BIDS, and the MR ID becomes the session.

Script Explanation
Below is a brief explanation of key parts of the EEG-to-BIDS conversion script:

File Discovery
The script walks through the input_dir and collects every .vhdr file. For each .vhdr, it will look for the companion .eeg and .vmrk files.

Filename Parsing
The script extracts a file-specific code (e.g., R078) from the .vhdr file name. An example might be R078_Day1_25012018_Rest2.vhdr.

Here, the script can parse R078 as the file code, Day1 as extra info (e.g., for run or ignoring it), and Rest2 as a task.
Excel Lookup

Step 1: Use the file code (R078) to look up the MR ID in the first Excel sheet (File-MR_ID.xlsx).
Step 2: Use that MR ID to look up the Subject ID in the second Excel sheet (MR_ID-subject.xlsx).
These lookups ensure that each EEG file is assigned to the proper participant (Subject ID) and session (MR ID).
BIDS Entities
The script uses:

Subject ID → sub-<subject> in BIDS
MR ID → ses-<mr_id> in BIDS
The last part of the file name (e.g. Rest2, Reward, Startle) → task-<task> in BIDS
Optionally, Day1/Day2 can be used for run-01 / run-02 if you have multiple runs.
Conversion

MNE: Reads the .vhdr file using MNE-Python.
mne-bids: Calls write_raw_bids to generate a BIDS-valid folder structure with BrainVision files.
Sidecar JSON Update

After writing to BIDS, the script opens the newly created .json sidecar and supplements it with additional metadata:
Number of EEG channels
Sampling frequency
Power line frequency
EEG reference
Filter settings (if any) from the header
This ensures the sidecar is as complete as possible according to the BIDS specification.
Multiple Experiments or Runs
If you record multiple EEG tasks or multiple visits for the same participant, you can specify the run or task accordingly. For instance, if Rest1 and Rest2 are separate recordings, the script can assign them as run-01 and run-02 (or separate tasks), ensuring unique naming within the same subject and session.

Quick Usage
Install Dependencies
Make sure you have Python 3, plus the packages pandas, mne, and mne-bids installed. For example:

bash
Copy
Edit
pip install pandas mne mne-bids
Adjust Paths
In the script, edit the following variables:

python
Copy
Edit
input_dir       = r'/path/to/eeg/input'
output_dir      = r'/path/to/bids/output'
file_map_excel  = r'/path/to/File-MR_ID.xlsx'
sub_map_excel   = r'/path/to/MR_ID-subject.xlsx'
Run the Script

bash
Copy
Edit
python convert_eeg_to_bids.py
(or whatever name you gave the script)

Check Output

A new directory named something like sub-..., ses-..., task-..., run-... will be created within output_dir.
Verify that your EEG data (in .vhdr, .eeg, .vmrk format) and a JSON sidecar are present under the eeg/ subfolder.
Notes and Troubleshooting
Task Names: If your filenames do not include a task keyword at the end, you may need to modify the extract_metadata function to set an appropriate default.
Missing Rows in Excel: If you get warnings like “FileName not found in Excel,” ensure that the first chunk of your filename (e.g. R078) exactly matches the FileName column in your spreadsheet.
Multiple Runs: If you have multiple runs (e.g. Day1, Day2), decide if you want them as separate sessions (ses-Day1, ses-Day2) or separate runs (run-01, run-02) within the same session. Update the script accordingly.
Power Line Frequency: The script sets raw.info['line_freq'] to 50 Hz. If your recordings used 60 Hz, modify that line.
That’s it! For more details on BIDS, see https://bids.neuroimaging.io/. For help with MNE-BIDS, see the official docs.

Feel free to open issues or pull requests if you have suggestions or improvements.

Happy converting!
