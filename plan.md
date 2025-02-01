<!-- ## Overview
The project focuses on automating surgical feedback generation, reframing it as a "next-action suggestion" task. The goal is to go beyond assessing surgical action quality and instead suggest specific next steps, which aligns with how trainers often provide feedback and supports higher-level planning. To achieve this, the model would generate action triplets [instrument][action][tissue structure] (paper). For example, [scissors][cut][blood-vessel] refers to feedback like "Cut the blood-vessel with the scissors".
Using these action triplets to represent feedback has advantages:
- Better evaluation: Evaluation can focus on the correctness of action triplets instead of word-to-word comparisons, reducing variations.
- Separation of "needed" from "arbitrary" feedback: "Arbitrary" feedback (e.g., "alright", "what's that?") complicates video and feedback mapping.
- Feedback delivery style invariant: Feedback styles vary by surgeon, and this approach normalizes representation, facilitating learning.

To form a dataset of action triplets, we would filter out "arbitrary" feedback using an LLM and prompt it to generate action triplets for each "needed" feedback instance.

On the technical side, we propose utilizing a hierarchical knowledge augmentation approach inspired by paper, "Procedure-Aware Surgical Video-Language Pretraining with Hierarchical Knowledge Augmentation". This method addresses domain-specific challenges in aligning textual and visual representations by enriching and refining hierarchical text inputs using an LLM. Similarly, we aim to correct, summarize, and enrich feedback representations to support action triplet generation and alignment with surgical video data.

Below is a revised version of your plan that integrates the key discussion points from your Slack messages and refines the initial overview into a more detailed roadmap. You can use this as your updated **plan.md**: -->

---

# Project Plan: Automated Surgical Feedback Generation

## 1. Overview
The project focuses on automating surgical feedback by reframing it as a “next-action suggestion” task. Instead of simply assessing surgical quality, our goal is to generate actionable, context-aware next steps. We represent feedback as **action triplets**:  
**[instrument] [action] [tissue structure]**  
For example, the triplet `[scissors][cut][blood-vessel]` corresponds to feedback like “Cut the blood-vessel with the scissors.”

### Advantages of the Action Triplet Representation
- **Evaluation Simplification:** Focusing on triplets allows evaluation to concentrate on key actionable elements rather than word-for-word matching.
- **Filtering Arbitrary Feedback:** By separating directly actionable feedback from “arbitrary” or non-informative comments (e.g., “alright”, “what’s that?”), we can better align video cues with verbal feedback.
- **Style Invariance:** Abstracting away individual trainer styles leads to a standardized representation that facilitates learning and comparison.

## 2. Motivation and Background
- **Feedback Variability:** Live surgical feedback can include both explicit instructions (actions, instruments, tissue cues) and arbitrary or ambiguous remarks.  
- **Existing Work:** Our approach is inspired by recent research (e.g., “Procedure-Aware Surgical Video-Language Pretraining with Hierarchical Knowledge Augmentation”), which demonstrates the value of hierarchical knowledge and multi-modal cues in surgical environments.
- **Challenge:** Integrating visual cues (e.g., instrument presence, tissue localization) with verbal feedback to robustly generate action triplets.

## 3. Technical Approach

### 3.1. Action Triplet Extraction
- **Action Extraction:**  
  - Use LLMs (e.g., GPT-4) with custom prompts to extract actionable phrases.  
  - Interpret phrases like “you’re gonna…”, “I want you to…” in the imperative form (e.g., “cut”, “clamp”, “retract”).  
  - If no explicit action is present, mark as `[NONE]`.

- **Instrument Identification:**  
  - Explicitly extract any mentions of surgical tools (e.g., robotic arms, graspers, scissors, electrocautery).  
  - Also account for implied references based on context.  
  - If no instrument is mentioned, use `[NONE]` (noting that instruments can sometimes be inferred from video).

- **Tissue Structure Recognition:**  
  - Detect explicit mentions (or context-driven references like “that”, “this”) of tissue structures.  
  - As with instruments, if no explicit reference is provided, mark as `[NONE]` (to be potentially resolved via visual cues).

### 3.2. Integration of Visual Cues
- **Visual Augmentation:**  
  - Utilize surgical video frames along with methods such as SAM2 or SurgicalSAM to extract additional visual cues.  
  - Combine OCR outputs and visual context to “fill in the gaps” when verbal feedback is incomplete.

### 3.3. Filtering Arbitrary vs. Needed Feedback
- **Action-Based Filtering:**  
  - If the extracted action is `[NONE]`, consider the feedback as likely arbitrary.  
  - Develop heuristics (and eventually annotations) to differentiate between directly actionable feedback and arbitrary commentary.
- **Multi-Modal Cross-Validation:**  
  - Use video information to reinforce or clarify ambiguous verbal instructions.

## 4. Data Collection and Annotation
- **Data Sources:**  
  - Surgical videos and corresponding live feedback transcripts.
- **Annotation Pipeline:**  
  - Use LLM-generated action triplets as a starting point.
  - Manually verify and refine these triplets, especially in ambiguous cases.
  - Annotate additional metadata (e.g., surgical phase, feedback context) to support hierarchical modeling.

## 5. Model Development and Evaluation

### 5.1. Baseline Models
- **Direct Generation:** Use GPT-4 (or similar) to generate feedback directly from video inputs.
- **Video Captioning Models:** Train standard captioning systems as a point of comparison.
- **Hierarchical Model:** Our proposed approach that extracts action triplets from multi-modal inputs and then generates feedback.

### 5.2. Evaluation Metrics
- **Generation Metrics:** Word Error Rate, BLEU, ROUGE to measure language generation quality.
- **Action Triplet Accuracy:** Evaluate the correctness and completeness of extracted triplets.
- **Practical Relevance:** Compare how well generated next-action suggestions align with expert surgical feedback.

## 6. Implementation Roadmap
1. **Prototype Prompt Development:**  
   - Create and test LLM prompts for extracting action, instrument, and tissue structure components.
   - Validate against historical feedback transcripts.

2. **Model Integration:**  
   - Combine the textual extraction with visual analysis modules (e.g., SAM2/SurgicalSAM).
   - Experiment with different fusion strategies to balance explicit and implied feedback components.

3. **Iterative Evaluation and Refinement:**  
   - Run experiments comparing baseline and proposed models.
   - Use both standard metrics and a custom surgical-action triplet evaluation.
   - Adjust the heuristics for filtering arbitrary feedback based on initial results.

4. **Future Directions:**  
   - Explore real-time feedback integration.
   - Extend the system to capture more complex, multi-step instructions.
   - Investigate additional modalities (e.g., instrument tracking) to further enhance triplet extraction.

## 7. Discussion and Challenges
- **Ambiguity in Feedback:**  
  - Differentiating actionable instructions from arbitrary comments remains a challenge.  
  - Rely on both context and visual cues for improved resolution.
  
- **Visual-Textual Alignment:**  
  - Bridging the gap between what is said and what is visible is non-trivial and requires robust multi-modal integration.
  
- **Domain-Specific Variability:**  
  - The nuances of surgical language and trainer styles must be managed to ensure the model’s robustness across different contexts.

## 8. Conclusion
By representing surgical feedback as action triplets and leveraging hierarchical, multi-modal data, our approach aims to generate precise, actionable feedback for surgical training. This methodology not only simplifies evaluation but also offers a structured way to integrate diverse data sources, ultimately supporting more effective and context-aware surgical coaching.
