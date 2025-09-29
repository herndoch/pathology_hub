### **Comprehensive AI Guide: CCH Anki Generation Engine**

**Document Version:** 19.0 (Definitive Master Protocol - No Placeholders)
**Date:** September 28, 2025

**Objective:** To provide a complete and unambiguous specification for an AI system to process structured source materials into a set of modular, relational CSV files and a supporting shell script. This protocol uses concrete file paths and examples throughout to eliminate all ambiguity.

---
#### **1.0 Foundational Principles**

1.  **Strict Protocol Adherence:** The AI must execute every rule within this document without deviation.
2.  **Pedagogical Mandate:** The primary objective is the creation of educationally effective learning tools, enforced by the **Intelligent Cloze Generation Mandate (Rule 5.3)**.
3.  **Source-Centric Modularity:** Each input source JSON file must result in a discrete output CSV file for card fronts.
4.  **Tag as Relational Primary Key:** The `Tag` string is the primary key and MUST be the first field in every data row.
5.  **Strict Content Format Specificity:** `Gallery` content is HTML; `Markdown` content is raw Markdown.

---
#### **2.0 Input Specification**

The AI shall be provided with the following assets for processing:

1.  **Source Libraries (List of JSON files):**
    *   A list of one or more absolute file paths to JSON documents.
    *   Example Input Path: `"/mnt/chromeos/GoogleDrive/MyDrive/1-Projects/Knowledge_Pipeline/_asset_library/textbooks/Breast_Pattern/Breast_Pattern_Figures.json"`
    *   Each object in a JSON array MUST contain:
        *   `image_path` (string): The absolute path to the source image file.
        *   `description` (string): Text for cloze generation.
        *   `notebook_links` (array of objects): Contains a `heading` for tag generation.

2.  **Master Notebook (Single Markdown file):**
    *   A single absolute file path to a UTF-8 encoded Markdown document.
    *   Example Input Path: `"/mnt/chromeos/GoogleDrive/MyDrive/1-Projects/Knowledge_Pipeline/_asset_library/notebooks/Breast_Pathology_Notebook.md"`

---
#### **3.0 Computational Workflow**

The workflow is a deterministic, three-stage process.

**3.1. Stage 1: Serialization of High-Quality Front-End Data**

1.  For each source JSON file provided:
    1.  **Derive Output Path:** The output filename is derived by replacing the `.json` extension of the source file with `_Anki.csv`.
        *   **Example:** If the input path is `.../Breast_Pattern_Figures.json`, the derived output path MUST BE `.../Breast_Pattern_Figures_Anki.csv`.
    2.  Iterate through each `figure` object in the JSON file.
    3.  For each `figure`, construct a two-field record `[Tag, Front_Content]` and append it to a temporary data store.
    4.  Serialize the data store to the derived output path as a pipe-delimited CSV.

**3.2. Stage 2: Aggregation and Serialization of Back-End Data**

1.  **Global Tag Aggregation:** Scan **all** provided Source Library JSON files to compile a master set of all **unique** tags.
2.  **Data Extraction and Assembly:**
    1.  **Derive Output Path:** The output filename is derived from the Master Notebook filename.
        *   **Example:** If the input path is `.../notebooks/Breast_Pathology_Notebook.md`, the derived output path MUST BE `.../notebooks/Breast_Pathology_Notebook_Anki.csv`.
    2.  For each `unique_tag` in the master set, construct a three-field record `[unique_tag, Gallery, Markdown]` and append it to a final data store.
    3.  Serialize the final data store to the derived output path as a pipe-delimited CSV.

**3.3. Stage 3: Generation of Media Transfer Script**

1.  The AI SHALL generate a `bash` script named `process_media.sh`.
2.  **Target Path:** The script MUST be hardcoded with the specific, non-negotiable target directory: `~/.local/share/Anki2/CCH/collection.media/`
3.  **Operation:** The script MUST perform a direct `cp` (copy) operation for every image file path found in the source JSONs. No renaming will occur. The script must contain the exact, absolute source directories.
    *   **Example Script Content:** The generated script will contain lines similar to:
        `SOURCE_DIR="/mnt/chromeos/GoogleDrive/MyDrive/1-Projects/Knowledge_Pipeline/_asset_library/textbooks/Breast_Pattern/figure_images/"`
        `find "$SOURCE_DIR" ... -exec cp -v {} "$ANKI_MEDIA_DIR" \;`

---
#### **4.0 Output Asset Specification**

*   **Front-End CSV File:**
    *   Example Filename: `Breast_Pattern_Figures_Anki.csv`
    *   Schema: `Tag|Front_Content`
*   **Back-End CSV File:**
    *   Example Filename: `Breast_Pathology_Notebook_Anki.csv`
    *   Schema: `Tag|Gallery|Markdown`
*   **Media Script:**
    *   Filename: `process_media.sh`

---
#### **5.0 Core Mandates: The Rules of Generation**

**5.1. Tag Generation Mandate:**
*   **Format:** `System::Specific_Diagnosis`.
*   **Example:** Given a notebook named `Breast_Pathology_Notebook.md` and a JSON `heading` of `"Pseudoangiomatous Stromal Hyperplasia (PASH)"`, the generated tag MUST BE the exact string `Breast::Pseudoangiomatous_Stromal_Hyperplasia`. Parentheticals are stripped, and spaces are replaced with underscores.

**5.2. Simplified Image `src` Mandate**
*   **Rule:** The `src` attribute of an `<img>` tag MUST contain **only the filename component** of the `image_path`.
*   **Example:** If the input `image_path` is `"/mnt/chromeos/GoogleDrive/MyDrive/1-Projects/Knowledge_Pipeline/_asset_library/textbooks/Breast_Pattern/figure_images/Breast_Pattern_page_103_img_2.jpeg"`, the `src` attribute in the generated HTML MUST BE exactly `'Breast_Pattern_page_103_img_2.jpeg'`.

**5.3. Intelligent Cloze Generation Mandate (CRITICAL QUALITY RULE)**
*   **Rule:** The AI must synthesize a high-quality, context-rich sentence and create a cloze deletion that tests a key concept, not just the diagnosis name. Low-value, zero-context clozes are forbidden.

**5.4. Gallery Field Mandate:**
*   **Rule:** The content MUST be a single HTML string of directly concatenated `<img>` tags.

**5.5. Markdown Field Mandate:**
*   **Rule:** The content MUST be the raw, unprocessed text extracted from the notebook file. It MUST NOT be converted to HTML.

---
#### **6.0 Execution Example & Compliance Test**

**Given this exact input `figure` object:**
```json
{
  "image_path": "/mnt/chromeos/GoogleDrive/MyDrive/1-Projects/Knowledge_Pipeline/_asset_library/textbooks/Breast_Pattern/figure_images/Breast_Pattern_page_103_img_2.jpeg",
  "description": "Pseudoangiomatous stromal hyperplasia (PASH): Histologically, PASH is characterized by stromal fibroblastic cells forming complex, anastomosing slit-like spaces.",
  "notebook_links": [{"heading": "Pseudoangiomatous Stromal Hyperplasia (PASH)"}]
}
```

**And this input Master Notebook filename:**
`Breast_Pathology_Notebook.md`

**The AI MUST produce this exact, single line of output for the `Breast_Pattern_Figures_Anki.csv` file:**

`Breast::Pseudoangiomatous_Stromal_Hyperplasia|A key feature distinguishing Pseudoangiomatous Stromal Hyperplasia (PASH) from a low-grade angiosarcoma is that the slit-like spaces are lined by bland myofibroblasts and are characteristically {{c1::empty}}, lacking red blood cells.<br><br><img src='Breast_Pattern_page_103_img_2.jpeg'>`

*   **Compliance Analysis:**
    *   **Tag:** Correctly generated as `Breast::Pseudoangiomatous_Stromal_Hyperplasia` (Rule 5.1).
    *   **Cloze Sentence:** High-quality, synthesized, and tests a key distinguishing feature (Rule 5.3).
    *   **Image `src`:** Contains *only the filename* from the absolute path (Rule 5.2).
    *   **Structure:** The final string is correctly formatted as `Tag|Front_Content`.
