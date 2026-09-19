# Research Document
## Visual Inspection & Defect Root-Cause Assistant

### NeuraX Hackathon 3.0 · Domain 2: AI in Industry and Automation

---

## Abstract

High-throughput manufacturing lines rarely fail for a single reason. Quality, capacity and economics interact continuously: a subtle defect signature can indicate process drift, a starved or blocked station can reduce throughput, and a quality improvement can have an unexpected effect on operating cost or margin.

This project proposes a software-only, AI-driven decision-support system that connects three traditionally separate functions: automated quality inspection, production-flow and bottleneck analysis, and profitability estimation.

The system uses organizer-provided inspection, production and economic datasets to classify acceptable and defective units, localize defects where the data supports it, identify uncertain or novel defect signatures, detect production bottlenecks, estimate throughput and cost impact, simulate profitability under alternative operating conditions, and generate evidence-linked recommendations.

The system does not control manufacturing equipment. All interventions and financial projections are simulated or advisory, allowing the prototype to demonstrate decision support without requiring cameras, PLCs, robots, or live production-line access.

---

# 1. Introduction and Problem Statement

Modern manufacturing lines operate as interconnected systems rather than isolated processes. Product variants, inspection conditions, process parameters, station capacity, downtime, changeovers, defects and operating costs can all vary over time.

A quality engineer may know that the defect rate has increased, while a production engineer may see a station becoming constrained. However, these observations are often analyzed separately.

This creates an important gap:

> A defect is not fully understood by knowing only its label, and a bottleneck is not fully understood by knowing only its utilization.

The proposed system therefore connects:

**Defect → Evidence → Process/Batch Pattern → Bottleneck → Throughput/Loss → Economic Impact → Investigation Recommendation**

The primary user is a **quality or process engineer** who needs a single view of product quality, production-flow health and economic impact.

### Core objectives

The system aims to answer:

1. What is wrong with the inspected unit?
2. Where is the defect located?
3. How confident is the system in its prediction?
4. Is the observation similar to known defects or potentially novel?
5. Which process or batch conditions are associated with the defect?
6. Where is the production flow constrained?
7. How does the constraint affect throughput?
8. What are the estimated scrap, rework and downtime losses?
9. How could a simulated process change affect profitability?
10. What should the engineer investigate next?

---

# 2. Why This Problem Is Challenging

### 2.1 Mixed product variants

Different products can have different geometries, materials, tolerances and visual characteristics. A model that performs well on one product variant may not automatically generalize to another.

### 2.2 Changing inspection conditions

Lighting, camera angle, calibration and other inspection conditions can change the input distribution. A model therefore needs to be evaluated under changing conditions rather than only on randomly shuffled samples.

### 2.3 Recurring and ambiguous defects

Different defect families may have visually similar signatures. A system that always produces a class label can therefore convert uncertainty into incorrect certainty.

### 2.4 Batch-to-batch process drift

Defect rates and process conditions can change between production batches. Monitoring these changes can reveal relationships that are not visible in a single aggregate statistic.

### 2.5 Dynamic production constraints

A production line does not necessarily have one permanent bottleneck. Cycle time, downtime, WIP, changeovers and utilization can change the active constraint over time.

### 2.6 Economic interaction

A reduction in defects may reduce scrap and rework, while a process intervention may affect throughput or operating cost. Therefore quality and production decisions should also be examined from an economic perspective.

---

# 3. Research and Conceptual Foundations

The proposed architecture combines established approaches from computer vision, uncertainty estimation, industrial engineering, statistics and operations research.

## 3.1 Visual Quality Inspection

Modern visual inspection commonly uses supervised computer-vision models to classify products and defects.

A pretrained CNN can be fine-tuned on the available inspection dataset when sufficient labels exist. Candidate architectures include established CNN families such as ResNet or EfficientNet.

Where spatial annotations are unavailable, explainability techniques such as **Grad-CAM** can provide a visual indication of the image regions contributing to a classification.

If defect labels are limited, feature-space or anomaly-based methods can complement supervised classification.

---

## 3.2 Uncertainty and Novelty Detection

A manufacturing system should not assume that every future defect belongs to a previously known category.

The proposed system therefore separates three outcomes:

- **Classified** — sufficiently confident known defect
- **Review Required** — prediction confidence is insufficient
- **Potential Novel Defect** — observation differs substantially from known defect patterns

Confidence calibration can be performed using **temperature scaling** on a validation set.

Novelty can additionally be estimated using distances between learned feature embeddings and known-class representations.

This creates a safer decision pathway:

**Known + confident → classify**

**Known but uncertain → human review**

**Unfamiliar → potential novel defect**

---

## 3.3 Process-Condition Association

Defect patterns can be compared with available process variables such as batch, station, shift, temperature, operating condition or other organizer-provided parameters.

Possible statistical tools include:

- Chi-square tests for categorical relationships
- ANOVA for differences between groups
- Logistic regression for defect probability
- Effect-size estimation
- Confidence intervals
- Multiple-testing correction using the Benjamini-Hochberg procedure

The system explicitly distinguishes **association from causation**.

A process variable associated with a higher defect rate is presented as a condition worth investigating, not as a proven root cause.

---

## 3.4 Bottleneck and Flow Analysis

The production line can be represented as a sequence of stations connected by buffers.

Station-level information such as:

- cycle time
- utilization
- WIP
- downtime
- changeover duration
- throughput

can be used to identify constrained stations.

The approach follows **Theory of Constraints** principles and can be complemented by lightweight queueing or discrete-event simulation.

Overall Equipment Effectiveness can also be decomposed into:

**OEE = Availability × Performance × Quality**

This helps separate losses caused by downtime, operating speed and defective output.

---

## 3.5 Cost and Profitability Analysis

Quality and production losses can be translated into economic terms using available cost information.

Relevant categories may include:

- scrap cost
- rework cost
- downtime cost
- operating cost
- product price
- product mix
- throughput

The system then provides scenario analysis such as:

> "If the defect rate for this family decreases by an assumed amount, what is the estimated change in scrap, throughput and margin?"

These are **what-if simulations**, not guaranteed financial predictions.

---

# 4. Proposed System

The proposed system contains five major layers:

```text
Organizer Data
      │
      ▼
Data Validation & Preprocessing
      │
      ├──────────────┬───────────────┐
      ▼              ▼               ▼
 Quality Engine   Flow Engine   Economics Engine
      │              │               │
      └──────────────┴───────────────┘
                     │
                     ▼
          Evidence & Insight Fusion
                     │
                     ▼
           Recommendation Engine
                     │
                     ▼
             Unified Dashboard