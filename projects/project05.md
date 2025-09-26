Freie Universität Berlin, Robert-Koch Institute

Practical course: SARS-2 Bioinformatics & Data Science

Max von Kleist, Daniela Börnigen


# Project 5

## Cross-Immunization

**Deadline**: 10.10.2025

**a) The presentation should be uploaded via whiteboard as `Project5.pdf`, no later than the above stated deadline.**

An exemplary structure of the talk: 
*	Title including group composition (possibly a link to the code repository)
*	Short background (what is this good for?) & description of task
*	Methods: A step-by-step explanation of what has been done; troubleshooting
*	Results: A summary of the outcomes and analysis
*	Discussion of results; aspects that were unclear; how the workflow could be improved 
*	Outlook

**b) Codes for conducting the project tasks should be zipped into `Project5.zip` and uploaded either via whiteboard or made accessible via GitHub or GitLab.** In the latter case, a link to the repo should be provided on the slides and the repo must be accessible to the lecturers.

**Detailed Tasks:**

1) Get a GISAID user account: [GISAID] https://gisaid.org/register/
This will take a few days!

2) Write a small R script to download mutation profiles from [outbreak.info](https://outbreak.info/) API for the following SARS-CoV-2 variants using your gisaid account:

* JN.1
* JN.2
* JN.3
* KP.3
* XBB.1.5

Use the methods described in the Hands-on to obtain the mutation profiles of these variants and compute their cross-reactivity 
Plot the the result in a cross-reactivit map

## 3. Reproduce the Relative fitness Trends of 5 of the variants (of your choice) shown in the Figure. 3 of Manuscript below: 
Manuscript: (SARS-CoV2 Evolution on a dynamic Immune landscape)[https://www.nature.com/articles/s41586-024-08477-8]
It does not need to look exactly the same, but you should be able to reproduce the trends

**NB:** the appendage `.X` refers to all daughter variants with the parent included, and the trend is re-normalized to their respective proportions (see Hints: `Project5_hints.ipynb` found [here](https://github.com/AlexiaNomena/SC2_VASIL))

4) Analyze results.

5) Prepare a talk to present the results & analysis
