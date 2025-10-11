### Comprehensive AI Guide: CCH Anki Generation Engine

**Document Version: 22.0 ; October 1, 2025**

**Objective:** To provide a complete and unambiguous specification for an AI system to process structured source materials into a single, modular, relational CSV file and a supporting shell script. This protocol uses concrete file paths and examples throughout to eliminate all ambiguity.

-----

### 1.0 Foundational Principles

\-**Strict Protocol Adherence:** The AI must execute every rule within this document without deviation. All previous instructions are now obsolete.
\-**Pedagogical Mandate:** The primary objective is the creation of educationally effective learning tools, enforced by the Intelligent Cloze Generation Mandate (Rule 6.4).
\-**Unified Relational Output:** All processed source materials (lectures, textbooks) will contribute to a single, unified CSV output file, where each row represents a complete, self-contained Anki card.
\-**Tag as Relational Primary Key:** The `Tag` string is the primary key and **MUST BE** the first field in every data row.
\-**Strict Content Format Specificity:** The output is a single pipe-delimited file with the schema `Tag|Front_Content|Markdown`. `Front_Content` is a composite HTML string, and `Markdown` is a complete HTML block derived from the master notebook.

-----

### 2.0 Pre-computation: Asset Restructuring for Lecture Sources

Before the main computational workflow, a preparatory script must be executed to standardize the asset structure for all lecture-derived sources. This one-time process modifies the source JSON files in-place and relocates the associated images into a flattened directory structure.

\-**JSON `image_path` Modification:** The script iterates through all JSON files within `_content_library/lectures/` and updates the `image_path` value.
\-**Purpose:** To remove the `slide_images` subdirectory from the path and prepend the lecture's name to the image filename, creating a unique, flat asset list per lecture.
\-**Before:** `_asset_library/lectures/Breast_Lecture_Invasive/slide_images/slide_0001.jpeg`
\-**After:** `_asset_library/lectures/Breast_Lecture_Invasive/Breast_Lecture_Invasive_slide_0001.jpeg`
\-**Physical Image File Relocation:** The script moves and renames the physical image files to match the updated `image_path` in the JSON.
\-The `slide_images` directory is removed after its contents have been moved.
\-**Reference Implementation:** The following shell script is the definitive implementation for this pre-computation step.

```bash
#!/bin/bash
# Navigate to the parent directory
cd /mnt/chromeos/GoogleDrive/MyDrive/1-Projects/Knowledge_Pipeline/ || exit
# Find all JSON files in _content_library/lectures/
find _content_library/lectures/ -type f -name "*.json" | while read -r json_file; do
    echo "Processing JSON file: $json_file"

    # Use sed to update the image_path in the JSON file
    sed -i -E 's|(_asset_library/lectures/([^/]+)/)slide_images/([^"]+)|_asset_library/lectures/\2/\2_\3|g' "$json_file"

    folder_name=$(basename "$(dirname "$json_file")")

    asset_folder_path="_asset_library/lectures/$folder_name"
    slide_images_path="$asset_folder_path/slide_images"

    if [ -d "$slide_images_path" ]; then
        # Find and move all image files
        find "$slide_images_path" -type f \( -name "*.jpg" -o -name "*.jpeg" -o -name "*.png" \) | while read -r old_image_path; do
            file_name=$(basename "$old_image_path")
            new_image_path="${asset_folder_path}/${folder_name}_${file_name}"

            if [ -f "$old_image_path" ]; then
                echo "Moving $old_image_path to $new_image_path"
                mv "$old_image_path" "$new_image_path"
            fi
        done

        # Remove the now-empty slide_images directory
        if [ -z "$(ls -A "$slide_images_path")" ]; then
            echo "Removing empty directory: $slide_images_path"
            rmdir "$slide_images_path"
        else
            echo "Warning: Directory not empty, not removing: $slide_images_path"
        fi
    else
        echo "Info: No 'slide_images' directory found for $folder_name. Skipping image move."
    fi
done
echo "Update complete."
```

-----

### 3.0 Input Specification

The AI shall be provided with the following assets for processing:

\-**Source Libraries (List of JSON files):**
\-A list of one or more absolute file paths to JSON documents. The system processes both textbook and lecture-derived JSONs. Note that lecture JSONs must first be processed by the script in Section 2.0.
\-**Textbook Example Path:** `/mnt/chromeos/GoogleDrive/MyDrive/1-Projects/Knowledge_Pipeline/_asset_library/textbooks/Breast_Pattern/Breast_Pattern_Figures.json`
\-**Lecture Example Path:** `/mnt/chromeos/GoogleDrive/MyDrive/1-Projects/Knowledge_Pipeline/_content_library/lectures/Breast_Lecture_Invasive.json`
\-Each object in a JSON array **MUST** contain:
\-`image_path` (string): The absolute path to the source image file.
\-`description` (string): Text associated with the image.
\-`notebook_links` (array of objects): Contains a `heading` for tag generation.
\-**Master Notebook (Single Markdown file):**
\-A single absolute file path to a UTF-8 encoded Markdown document.
\-**Example Input Path:** `/mnt/chromeos/GoogleDrive/MyDrive/1-Projects/Knowledge_Pipeline/_asset_library/notebooks/Breast_Pathology_Notebook.md`

-----

### 4.0 Computational Workflow

The workflow is a deterministic, unified process that generates a single CSV file and a media script.

#### 4.1. Initialization

\-Create a temporary, in-memory data store (e.g., a list of lists or an array of objects) to hold the final records for the CSV.

#### 4.2. Data Processing Loop

\-Iterate through each source JSON file provided (textbook or lecture).
\-Within each file, group all image objects according to the **Image Grouping Mandate (Rule 6.3)**.

#### 4.3. Record Construction

\-For each generated group:
\-1. **Generate Tag:** Create the `Tag` string using the first item in the group according to the **Tag Generation Mandate (Rule 6.1)**.
\-2. **Assemble Front\_Content:** Construct the `Front_Content` HTML string for the entire group according to the **Front Content Assembly Mandate (Rule 6.5)**.
\-3. **Generate Markdown:** Create the `Markdown` HTML string by looking up the generated `Tag` in the Master Notebook, as specified by the **HTML Generation Mandate (Rule 6.6)**.
\-4. **Append Record:** Combine the three generated strings into a single, pipe-delimited record (`Tag|Front_Content|Markdown`) and add it to the temporary data store.

#### 4.4. Final Serialization

\-Derive the output filename from the Master Notebook filename by appending `_Anki_Master.csv`.
\-Example: `Breast_Pathology_Notebook.md` -\> `Breast_Pathology_Notebook_Anki_Master.csv`.
\-Serialize the entire temporary data store to this single output path as a pipe-delimited CSV.

#### 4.5. Generation of Media Transfer Script

\-The AI **SHALL** generate a bash script named `process_media.sh`.
\-**Target Path:** The script **MUST** be hardcoded with the specific, non-negotiable target directory: `~/.local/share/Anki2/CCH/collection.media/`.
\-**Operation:** The script must identify every unique source directory from the `image_path` fields across all input JSONs and perform a direct `cp` (copy) operation for every image file.
\-**Example Script Content:**

```bash
#!/bin/bash
ANKI_MEDIA_DIR="~/.local/share/Anki2/CCH/collection.media/"
SOURCE_DIRS=(
  "/mnt/chromeos/GoogleDrive/MyDrive/1-Projects/Knowledge_Pipeline/_asset_library/textbooks/Breast_Pattern/figure_images/"
  "/mnt/chromeos/GoogleDrive/MyDrive/1-Projects/Knowledge_Pipeline/_asset_library/lectures/Breast_Lecture_Invasive/"
  # AI will dynamically add any other unique source directories found
)
echo "Copying media files to Anki..."
mkdir -p "$ANKI_MEDIA_DIR"
for DIR in "${SOURCE_DIRS[@]}"; do
  echo "Processing source directory: $DIR"
  find "$DIR" -type f \( -name "*.jpeg" -o -name "*.jpg" -o -name "*.png" \) -exec cp -v {} "$ANKI_MEDIA_DIR" \;
done
echo "Media copy complete."
```

-----

### 5.0 Output Asset Specification

\-**CSV File:**
\-**Example Filename:** `Breast_Pathology_Notebook_Anki_Master.csv`
\-**Schema:** `Tag|Front_Content|Markdown`
\-**Media Script:**
\-**Filename:** `process_media.sh`

-----

### 6.0 Core Mandates: The Rules of Generation

#### 6.1. Tag Generation Mandate:

\-**Format:** `System::Specific_Diagnosis`.
\-**Example:** Given a notebook named `Breast_Pathology_Notebook.md` and a JSON heading of "Pseudoangiomatous Stromal Hyperplasia (PASH)", the generated tag **MUST BE** the exact string `Breast::Pseudoangiomatous_Stromal_Hyperplasia`. Parentheticals are stripped, and spaces are replaced with underscores.

#### 6.2. Simplified `src` Mandate:

\-**Rule:** The `src` attribute of an `<img>` tag **MUST** contain only the filename component of the `image_path`.
\-**Textbook Example:** If `image_path` is `.../figure_images/Breast_Pattern_page_103_img_2.jpeg`, the `src` attribute **MUST BE** `'Breast_Pattern_page_103_img_2.jpeg'`.
\-**Lecture Example (Post-Restructuring):** If the `image_path` is `.../Breast_Lecture_Invasive/Breast_Lecture_Invasive_slide_0001.jpeg`, the `src` attribute **MUST BE** `'Breast_Lecture_Invasive_slide_0001.jpeg'`.

#### 6.3. Image Grouping Mandate:

\-**Rule:** Images are grouped to form a single Anki card based on the source type.
\-**For Lectures:** All images from the same slide number (e.g., `slide_0001`) are grouped together. The grouping key is the slide identifier.
\-**For Textbooks:** Images are grouped if they satisfy **EITHER** of the following conditions:
\-1. **Shared Figure Number:** They share the same figure number (e.g., "Fig 3.14") in their `description` fields.
\-2. **Shared Context:** They are on the same page number (e.g., `_page_183_`) **AND** are linked to the exact same `heading` in their `notebook_links`.

#### 6.4. Intelligent Cloze Generation Mandate (CRITICAL QUALITY RULE):

\-**Rule:** The AI must synthesize a high-quality, context-rich sentence and create a cloze deletion that tests a key concept, not just the diagnosis name.

#### 6.5. Front Content Assembly Mandate:

\-**Rule:** The `Front_Content` is a single, continuous HTML string constructed in a precise order for each group of images.
\-1. **Cloze Sentence:** Start with the high-quality cloze sentence.
\-2. **Image Rows:** Append the image containers. All images in the group are placed into rows, where each row is a `<div class='anki-gallery'>`. A single div can contain a maximum of two `<img>` tags.
\-3. **Descriptions Block:** Append the collated descriptions from all grouped images inside a `<details><summary>Image Descriptions</summary>...</details>` block.
\-**Final Structure Example (for a group with 3 images):** `{Cloze Sentence}<div class='anki-gallery'><img><img></div><div class='anki-gallery'><img></div><details>...</details>`

#### 6.6. HTML Generation Mandate (for `Markdown` field):

\-**Rule:** The content for the `Markdown` field **MUST** be a single HTML string generated from the Master Notebook.
\-1. **Parse Topic:** From the input `Tag` (e.g., `Breast::Fat_Necrosis`), extract the title ("Fat Necrosis").
\-2. **Locate Section:** Find the corresponding `## Fat Necrosis` heading in the Markdown file.
\-3. **Convert to HTML:** Apply the specified transformations (`##` to `<h2>`, `* Term:` to `<li><b>Term:</b>`, etc.).
\-4. **Handle Missing Topics:** If a matching heading is not found, the HTML content **MUST BE** exactly `<p>Information not available in the provided document.</p>`.
