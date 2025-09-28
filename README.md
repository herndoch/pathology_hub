# The Pathology Knowledge Hub Pipeline (v3.2)

![Status](https://img.shields.io/badge/status-in_development-blue)
![Language](https://img.shields.io/badge/language-Python-brightgreen)
![License](https://img.shields.io/badge/license-MIT-lightgrey)

This repository contains the Python-based data processing pipeline for the "Pathology Knowledge Hub" project. The goal is to create a private, expert-vetted search engine for a pathology department's internal expertise by transforming disparate educational assets (videos, textbooks, presentations) into a unified, searchable knowledge base.

## The Core Problem

Academic pathology departments possess decades of invaluable educational content that is currently siloed, unstructured, and largely unsearchable. This fragmentation creates a significant operational drag, forcing faculty and trainees to rely on inefficient manual searches or unverifiable public web sources for critical information.

## The Solution: A "Golden Asset" of Structured Knowledge

This pipeline solves the problem by creating a foundational, structured data asset. Our core philosophy is **"Data-First"**: the primary output is not just a search application, but a library of highly structured, standardized **JSON files** (the "Golden Asset"). This permanent, queryable representation of our department's knowledge is designed to fuel a decade of future educational and research innovations, from building custom search engines to training secure, in-house AI assistants.

## The Technical Workflow

The project is a multi-stage data pipeline executed in a Google Colab environment, with all file paths managed by a centralized `PATHS` dictionary for robustness.

*   **Block 1: Textbook Processing (`PyMuPDF`, `Gemini 2.5 Pro & Flash`)**
    *   Uses a highly concurrent, asynchronous process (`aiohttp`) to perform text extraction and enhancement on source PDFs.
    *   A multimodal vision model (`gemini-2.5-flash`) performs advanced Optical Character Recognition (OCR) to accurately link figures to their specific captions within the text.
    *   **Output:** Creates chunk-based `_CONTENT.json` and `_FIGURES.json` files, optimized for RAG systems.

*   **Block 2: Narrated Video Processing (`Whisper`, `OpenCV`, `Gemini`)**
    *   Uses **OpenAI's Whisper** for highly accurate, time-stamped transcription.
    *   Leverages **OpenCV and the Structural Similarity Index (SSIM)** for intelligent, vision-based segmentation of lectures into discrete clips based on slide changes.
    *   Employs a robust **two-step AI enhancement process:** `gemini-2.5-pro` performs factual vision analysis on the slide image, then `gemini-2.5-flash` synthesizes this data with the transcript to generate clean titles, summaries, and metadata.
    *   **Output:** Creates a `final_ENHANCED_data.json` file for the lecture.

*   **Block 3: RAG (Retrieval-Augmented Generation) Enrichment (`Sentence-Transformers`)**
    *   Parses a user-selected markdown notebook (e.g., from the [Pathology Notebook](https://pathologynotebook.com/notebook)) into a knowledge base.
    *   Uses a `sentence-transformer` model to perform semantic search, automatically tagging each piece of content (slides, figures, text chunks) with relevant headings from the notebook in an autonomous, AI-arbitrated "God Mode" workflow.

*   **Block 4: Knowledge Hub Packaging**
    *   A "publishing" tool that compiles selected content into a single, self-contained **HTML application**.
    *   The final Hub is fully interactive, featuring **Deep Contextual Linking** where results link directly to the precise video timestamp or the exact PDF page, and it works completely offline.

## High-Value Use Case: Automated Anki Card Generation

The structured JSON "Golden Asset" enables powerful downstream applications. Our pipeline can programmatically generate sophisticated, media-rich **Anki flashcards**, combining active recall (cloze deletions) with rich, contextual answers that include annotated images, linked video segments, and relevant text—a task impossible to do manually at scale.

## Project Status

This project is in active development. As a resident-led, non-profit academic initiative developed at UCSF, we are seeking support for the essential cloud computing and API costs required to build and host this RAG pipeline as a scalable model for other academic pathology departments.

## The Team

*   **Ronald Balassanian, MD** - Principal Investigator (Professor & Director, Pathology Residency Program, UCSF)
*   **Charlie Herndon, DO** - Co-Investigator & Lead Developer (Resident Physician, UCSF)
