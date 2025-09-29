Capiche. I understand perfectly. This is a major, and brilliant, evolution of the workflow.

You are moving from a generic batch-processing model to a **source-centric, modular system**. This is far superior for organization and tracking progress. Creating separate CSVs for each source (`BST_Horvai_Anki.csv`, etc.) makes the entire pipeline transparent.

Furthermore, separating the **Gallery** into its own dedicated field is a significant upgrade for card design and flexibility in Anki. It allows you to style and position the image gallery independently from the main text content.

This requires a new definitive SOP. All previous versions are now obsolete.

---
---

### **The CCH Anki Generation Project: Master SOP and User Implementation Guide**

**Document Version:** 14.0 (Definitive Master Version)
**Date:** September 28, 2025

**Objective:** To establish a modular, source-centric, and repeatable process for generating advanced Anki flashcards. This workflow creates separate "front" files for each source and a single "back" file from a master notebook, designed for a 4-field Anki note type (`Front`, `Markdown`, `Gallery`, `Tags`).

---
### **Part I: Standard Operating Procedure for AI Execution**
---

#### **1.0 Governing Principles**

1.  **Strict Adherence:** The AI must follow every rule in this SOP (v14.0) without deviation.
2.  **Source-Separated Outputs:** The AI MUST generate a distinct "fronts" CSV for each source JSON file provided (e.g., `BST_Horvai_Figures.json` -> `BST_Horvai_Anki.csv`).
3.  **Four-Field Card Structure:** The entire process is designed to populate a four-field Anki note. The `Gallery` is now a distinct data field, separate from the notebook `Markdown`.
4.  **Tag-First Mandate:** In all generated CSVs, the `Tag` field MUST be the first column. This ensures it acts as the primary key for all data merging.
5.  **Content Integrity:** The `Markdown` field for the back of the card must be extracted from the notebook and provided *as raw, unprocessed Markdown*. HTML conversion will be handled by Anki's editor. The `Gallery` field, however, MUST be formatted as clean HTML.

#### **2.0 Input Specification**

1.  **A list of one or more Figure Library JSON files** (e.g., `BST_Horvai_Figures.json`, `BST_Lecture_SoftTissue1.json`). These are the sources for the **front** of the cards.
2.  **A single Notebook Markdown file** (e.g., `BST Pathology Notebook.md`). This is the **sole source for the back text** of the cards.

#### **3.0 Core Processing Logic (The Two-Stage Method)**

**3.1. Stage 1: Generate Card Fronts (Source by Source)**

1.  For **each** source JSON file provided by the user:
    *   Determine the output filename based on the input (e.g., `BST_Horvai_Figures.json` -> `BST_Horvai_Anki.csv`).
    *   Initialize an empty list for this specific source's card data.
    *   Iterate through each figure object in the *current* JSON file.
    *   For each figure:
        *   Generate the Anki Tag (e.g., `BST::Leiomyoma`).
        *   Generate the HTML for the Front field (cloze sentence + `<br><br>` + prefixed `<img>` tag).
        *   Store the `Tag` and `Front` content.
    *   **Output File:** Write the list to a pipe-delimited CSV file named `[Source_JSON]_Anki.csv`. The structure MUST be: `[Tag]|[Front Content]`.
2.  Repeat this process for all provided JSON files, resulting in multiple `*_Anki.csv` output files.

**3.2. Stage 2: Generate Master Backs (Text and Gallery)**

1.  First, scan **all** provided source JSON files to compile a list of every unique tag that will be generated across the entire project.
2.  Determine the output filename from the notebook file (e.g., `BST_Pathology_Notebook.md` -> `BST_Pathology_Notebook_Anki.csv`).
3.  For **each** unique tag identified in step 1:
    *   **A. Build the Gallery Field:** Scan **all** source JSON files again. Find *every* figure object that resolves to the current unique tag. Collect their sanitized, prefixed image paths and compile them into a single string of HTML `<img>` tags. This string is the complete content for the `Gallery` field.
    *   **B. Extract the Markdown Field:** Find the heading in the `notebook.md` file that exactly matches the `Specific_Diagnosis` part of the tag. Extract **all** raw Markdown text under that heading until the next heading of the same or higher level. This raw text is the complete content for the `Markdown` field.
    *   **C. Store the Data:** Create a record containing the `Tag`, the `Gallery` HTML string, and the raw `Markdown` text.
4.  **Output File:** Write all records to a single, pipe-delimited CSV file named `[Notebook_Filename]_Anki.csv`. The structure MUST be: `[Tag]|[Gallery]|[Markdown]`.

---
### **Part II: End-User Implementation Guide (Colab Workflow)**
---

**Objective:** To correctly set up Anki, then use the multiple AI-generated CSV files to merge and import your new, powerful 4-field flashcards.

#### **0.0 Phase 0: One-Time Anki Setup (CRITICAL FIRST STEP)**

1.  Open the Anki desktop application.
2.  Go to **Tools > Manage Note Types**.
3.  Click **Add** and choose to add a new type based on `Cloze`.
4.  Name the new note type `Pathology Cloze (4-Field)`.
5.  With the new type selected, click **Fields...**.
6.  Add two new fields. You should have a total of four fields. Rename them to be exactly:
    *   `Text` (This will hold the front/cloze content)
    *   `Extra` (This will hold the Markdown notebook text)
    *   `Gallery` (This will hold the image gallery)
    *   `Tags` (This will be populated by the import process)
7.  Click **Save**. Your Anki is now ready.

#### **1.0 Phase 1: Generate and Save AI Files**

1.  Provide the AI with the paths to your source JSON files and your notebook file.
2.  The AI will provide multiple files:
    *   Several `[Source]_Anki.csv` files (e.g., `BST_Horvai_Anki.csv`, `BST_Lecture_Anki.csv`).
    *   One `[Notebook]_Anki.csv` file.
    *   One `process_media.sh` script.
3.  Save all these files into the same folder (e.g., `~/Knowledge_Pipeline/anki_exports/`).

#### **2.0 Phase 2: Process Media Files**
1.  Open your Terminal and run the `process_media.sh` script as before. This will copy and prefix all necessary images.

#### **3.0 Phase 3: Merge CSV Files in Google Colab**

1.  Open a new Colab notebook and mount your Google Drive.
2.  Copy and paste the following Python script. **This script is new and more powerful.** It will automatically find all your "fronts" files and merge them with your single "backs" file.

    ```python
    import pandas as pd
    import glob
    import os

    # --- 1. DEFINE PATHS & FILENAMES ---
    # The folder where all your AI-generated CSVs are saved.
    export_folder = '/content/drive/MyDrive/1-Projects/Knowledge_Pipeline/anki_exports/'
    # The filename of your single "backs" file.
    notebook_csv_name = 'BST_Pathology_Notebook_Anki.csv'
    # The desired name for the final merged file.
    output_file_path = os.path.join(export_folder, 'anki_import_final.csv')

    # --- 2. LOAD AND CONCATENATE ALL "FRONTS" FILES ---
    # Find all CSV files in the folder that are NOT the notebook CSV.
    fronts_files = glob.glob(os.path.join(export_folder, '*_Anki.csv'))
    fronts_files = [f for f in fronts_files if os.path.basename(f) != notebook_csv_name]

    print(f"Found {len(fronts_files)} source files to merge.")
    
    # Load and combine them into a single DataFrame.
    df_list = [pd.read_csv(f, sep='|', header=None, names=['Tags', 'Text']) for f in fronts_files]
    fronts_df = pd.concat(df_list, ignore_index=True)

    # --- 3. LOAD THE "BACKS" FILE ---
    backs_file_path = os.path.join(export_folder, notebook_csv_name)
    backs_df = pd.read_csv(backs_file_path, sep='|', header=None, names=['Tags', 'Gallery', 'Extra'])

    # --- 4. MERGE THE DATA ---
    # Merge the combined fronts with the master backs using the 'Tags' column.
    merged_df = pd.merge(fronts_df, backs_df, on='Tags')

    # --- 5. ENSURE CORRECT FINAL COLUMN ORDER FOR ANKI ---
    # This order must match the import mapping in the next phase.
    final_df = merged_df[['Text', 'Extra', 'Gallery', 'Tags']]

    # --- 6. SAVE THE FINAL CSV ---
    final_df.to_csv(output_file_path, sep='|', header=False, index=False)

    print(f"Merge successful! {len(final_df)} cards were processed.")
    print(f"Final file saved to: {output_file_path}")
    ```
3.  Run the cell to create your `anki_import_final.csv`.

#### **4.0 Phase 4: Import into Anki**

1.  Open Anki, go to **File > Import...** and select `anki_import_final.csv`.
2.  Configure the Import Dialog Box:
    *   Note Type: **`Pathology Cloze (4-Field)`** (the new type you created).
    *   Deck: Your destination deck.
    *   Fields separated by: **Pipe (`|`)**.
    *   **Allow HTML in fields:** Ensure this is **checked**.
    *   **Field Mapping (CRITICAL):**
        *   `Field 1` -> **Text** (Front/Cloze)
        *   `Field 2` -> **Extra** (Markdown text)
        *   `Field 3` -> **Gallery**
        *   `Field 4` -> **Tags**
3.  Click **Import**.
