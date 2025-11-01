# 📰 Project: Automated News Transcript Segmentation

## 1. **Project Overview**

This project focuses on **automatically segmenting broadcast news transcripts** into distinct news items, using semantic and temporal analysis. The transcripts are either in **French** or **Arabic** and include timestamps approximately every 10 seconds.

**Objective:**

* Detect topic changes in a broadcast.
* Aggregate consecutive transcript blocks talking about the same news item.
* Produce structured outputs with segment metadata: start time, end time, aggregated text, optional subject label, sentiment, and named entities.

**Use Cases:**

* Summarization and content indexing of news broadcasts.
* Searchable archives of news items.
* Alignment of transcript segments with audio/video for visualization or automated clipping.
* Analysis of news coverage trends or topic frequency.

---

## 2. **Input Data**

The raw input is a timestamped transcript:

```
"[0.00s] Bonsoir et bienvenue dans cette édition ...\n[12.56s] juste après ce journal d'une programmation spéciale match Maroc-Niger...\n[27.44s] sa majesté le roi Mohamed VI Amir al-Muminin a présidé une veillée religieuse...\n[43.88s] prince Moulay-Abdallah ouvre ses portes ..."
```

### Characteristics:

* Multi-lingual: French or Arabic (no mixing per document).
* Contains timestamps marking blocks (~10 seconds).
* Blocks vary in length depending on speech.
* May include transitional phrases or filler text (“Bonsoir”, “Merci de nous suivre”).

---

## 3. **Expected Output**

The output is a **structured JSON** with segmented news items:

```json
{
  "segments": [
    {
      "start": 0.0,
      "end": 27.44,
      "subject": "Annonce Spéciale",
      "text": "[0.00s] Bonsoir et bienvenue ...\n[12.56s] juste après ce journal ...",
      "sentiment_analytic": null,
      "entities": null
    },
    {
      "start": 27.44,
      "end": 84.84,
      "subject": "Veillée religieuse",
      "text": "[27.44s] sa majesté le roi ...\n[43.88s] prince Moulay-Abdallah ouvre ...",
      "sentiment_analytic": null,
      "entities": null
    }
  ]
}
```

Metadata:

* `start`/`end`: timestamps in seconds.
* `text`: concatenated block texts for the segment.
* `subject`: optional topic label.
* `sentiment_analytic`: optional sentiment analysis per segment.
* `entities`: optional extracted entities (people, organizations, locations).

---

## 4. **System Architecture / Pipeline**

### 4.1 Block Parsing

**Task:** Convert raw transcript string into structured blocks.

**Steps:**

1. Extract timestamps and text using regex.
2. Generate `start` and `end` for each block.
3. Output list of dictionaries, ordered by start time.

**Why:**
Blocks are the atomic units for semantic comparison and similarity computation.

---

### 4.2 Text Preprocessing

**Operations:**

* Remove extraneous characters, normalize punctuation.
* Lowercase (for French), normalize diacritics (for Arabic).
* Remove stopwords if using keyword-based labeling.

**Why:**
Clean input improves embedding accuracy, keyword extraction, and similarity computation.

---

### 4.3 Semantic Representation

**Task:** Convert each block into a dense vector embedding.

**Methods:**

* **Sentence embeddings:** Use transformer models (e.g., `SBERT`, `LaBSE`) suitable for French or Arabic.
* **Optional:** Topic distribution vectors via LDA or BERTopic.

**Why:**
Embeddings capture semantic meaning beyond exact words. Two blocks about the same story will have similar embeddings.

---

### 4.4 Similarity Analysis

**Task:** Measure semantic similarity between adjacent blocks.

* Compute cosine similarity between embeddings of consecutive blocks.
* Optionally, smooth similarity over a sliding window of 2–3 blocks to reduce noise.

**Why:**
A drop in similarity indicates a topic transition.
Research shows embedding-based similarity outperforms simple lexical overlap in multilingual or morphologically complex languages (Arabic, French).

---

### 4.5 Boundary Detection

**Task:** Identify start points of new news items.

* Threshold-based detection: similarity below a tuned threshold → new segment.
* Optional heuristics:

  * Minimum segment length (e.g., ≥ 20 seconds).
  * Cue phrases (“À suivre”, “Et maintenant”, Arabic equivalents).
  * Large time gaps between blocks (e.g., due to commercials).

**Why:**
Combines semantic change detection with domain-specific heuristics for broadcast news.

---

### 4.6 Segment Aggregation

**Task:** Group blocks between detected boundaries into segments.

* Determine `start` and `end` timestamps.
* Concatenate block texts into one string.
* Maintain chronological order.

**Why:**
Produces cohesive news item units suitable for labeling and analytics.

---

### 4.7 Topic/Subject Labeling

**Task:** Assign a short descriptive subject to each segment.

**Methods:**

1. **Keyword extraction:** e.g., `YAKE`, `KeyBERT`.
2. **Rule-based mapping:** Map keywords to predefined subjects.
3. **Classification:** Train a simple supervised classifier if labeled examples are available.

**Why:**
Enhances interpretability and usability of segments.

---

### 4.8 Additional Metadata Extraction (Optional)

* **Sentiment analysis:** Positive, neutral, negative.
* **Named entity recognition:** Extract entities (persons, organizations, locations).
* **Language detection:** Confirm block language for multilingual support.

---

### 4.9 Output Assembly

**Task:** Save structured segments as JSON.

* Ensure full coverage (start = 0.0, end = last block).
* Include all metadata fields.
* Optionally, generate a table of contents for overview.

---

## 5. **Evaluation & Validation**

**Methods:**

* **Manual validation:** Sample segmented transcripts to check alignment with news items.
* **Segmentation metrics:** Pk, WindowDiff, commonly used in topic segmentation research.
* **Heuristic validation:** Ensure minimum segment length, no overlapping segments, and continuity of timestamps.

---

## 6. **Technical Stack / Libraries**

| Task                   | Libraries / Tools                       |
| ---------------------- | --------------------------------------- |
| Transcript parsing     | `re`, `pandas`                          |
| Embeddings             | `sentence-transformers`, `transformers` |
| Similarity computation | `numpy`, `scipy`                        |
| Change-point detection | `ruptures`, custom thresholding         |
| Keyword extraction     | `yake`, `keybert`                       |
| Sentiment / NER        | `spacy`, `transformers`                 |
| Output handling        | `json`, `pandas`                        |
| Visualization          | `matplotlib`, `plotly` (optional)       |

---

## 7. **Research Basis & References**

* Hearst, 1997: **TextTiling** – lexical cohesion for topic segmentation.
* Reimers & Gurevych, 2019: **SBERT embeddings** for semantic similarity.
* “Advancing Topic Segmentation of Broadcasted Speech with Multilingual Semantic Embeddings”, 2024.
* Modern BERTopic / embedding-based segmentation techniques.

**Key insight:** Modern embeddings + similarity analysis + domain heuristics give high accuracy, outperforming classical lexical or TF-IDF-based segmentation.

---

## 8. **Project Deliverables**

1. **Python module** for segmenting raw transcripts.
2. **JSON output** with structured segments, including metadata.

