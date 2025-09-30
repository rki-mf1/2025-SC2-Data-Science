Freie Universität Berlin, Robert-Koch Institute

Practical course: SARS-CoV-2 Bioinformatics & Data Science

Max von Kleist, Martin Hölzer

# Project 6

## Outbreak detection for SARS-CoV-2

**Deadline**: 10.10.2025

*The project should be worked out in groups of two or three students. Students should document their progress and present their work on Friday; 10.10.2025 to the lecturers and students. The talk should be about 30-40 min, and allowing for 10-15min of questions.*

**a) The presentation should be uploaded via whiteboard as `Project6.pdf`, no later than the above stated deadline.**

An exemplary structure of the talk: 
*	Title including group composition (possibly a link to the code repository)
*	Short background (what is this good for?) & description of task
*	Methods: A step-by-step explanation of what has been done; troubleshooting
*	Results: A summary of the outcomes and analysis
*	Discussion of results; aspects that were unclear; how the workflow could be improved 
*	Outlook

**b) Codes for conducting the project tasks should be zipped into `Project5.zip` and uploaded either via whiteboard or made accessible via GitHub or GitLab.** In the latter case, a link to the repo should be provided on the slides and the repo must be accessible to the lecturers.

**Detailed Tasks:**

1) Install `Breakfast` [https://github.com/rki-mf1/breakfast](https://github.com/rki-mf1/breakfast)
2) Download [SARS-CoV-2 sequences](https://osf.io/de3v7) and corresponding [meta data](https://osf.io/rsv39)
3) Perform basic statistics:
  * Number of ambiguous nucleotides per sequence
  * Number of mutations
  * Project and cluster sequences (for example PCA, kmeans)
4) Configure & run `Breakfast`
  * Visualize cluster sizes and number distributions
  * Compute within cluster _average_ mutation edit distance
5) Based on the meta data: compute, visualize, and analyze:
  * sampling date distribution within clusters vs. in the entire dataset
  * Within-cluster compositions based on zip-code (Federal state proximity, e.g. use the SEQUENCING_LAB_PC as a proxy for the zip-code)
