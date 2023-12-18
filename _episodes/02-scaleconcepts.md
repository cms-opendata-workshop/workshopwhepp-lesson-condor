---
title: "Logistics of an Open Data analysis"
teaching: 20
exercises: 0
questions:
- "Can you log in to TIFR to use condor?"
objectives:
- "Learn the workshop login commands for TIFR and test your access."
keypoints:
- "Logging in to the TIFR cluster is easy for workshop participants!"
---

Go over the concepts needed to process an analysis:

- Use local resources and these lessons to build out analysis scripts. Example: run POET, merge the trees, run an analysis script (ROOT or Coffea) to produce some histogram stored in a ROOT file.
- Identify all the datasets you want to run
- Determine which analysis steps to group into a single "job"
- Submit parallel jobs to a computing resource:
  - Cloud computing: share our previous lessons on Google Cloud
  - HTCondor
- Collect the job output and continue analysis


{% include links.md %}

