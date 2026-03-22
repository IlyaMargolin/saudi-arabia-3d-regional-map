# Saudi Arabia 3D Regional Prototype Map

## Overview

This repository contains an open prototype of a 3D regional map of Saudi Arabia developed as part of a separate project task based on a provided dataset.

The project demonstrates a reusable workflow for regional data visualization through 3D geometry, category-based coloring, and interactive tooltips. The current version is published as a presentation and development prototype that can be reviewed, adapted, and extended for analytical, communication, or exploratory purposes.

## What the map shows

The map visualizes the administrative regions of Saudi Arabia in a 3D format.

In the current implementation:

- region color reflects its assigned category within the selected comparison logic;
- extrusion height reflects the relative value of the `score` field;
- interactive tooltips display the region name and the related values included in the input dataset.

The visualization should therefore be interpreted as an analytical display layer built on top of a supplied dataset and a selected mapping logic.

## About the score

The `score` field in this prototype is a comparative summary value used to support visual differentiation between regions.

It should be interpreted as a visualization-oriented composite indicator within the current project version. Its meaning depends on the structure of the input dataset and the adopted aggregation logic. If users replace the source values or apply a different methodology, the score and the resulting visual hierarchy may change accordingly.

## Project status

This repository is published as an open prototype.

It is intended to serve as:

- a demonstration of the implemented visualization workflow;
- a reusable technical base for further development;
- a template for replacing or updating the underlying regional data.

The current version is suitable for presentation, prototyping, adaptation, and internal analytical discussion.

## Limitations

This repository does not represent an official governmental cartographic release, a normative statistical publication, or a legally authoritative administrative reference.

The map is a project-based visualization generated from a supplied dataset and a selected display logic. As with any prototype built on configurable inputs, results may vary when source values, matching rules, category thresholds, or aggregation methods are updated.

Users are encouraged to review, replace, or refine the data model depending on their own analytical needs and validation requirements.

## Suggested repository structure

```text
assets/
  saudi_regions_13.geojson
data/
  saudi_regions_data.csv
output/
  saudi_3d_development_map.html
notebooks/
  saudi_map_colab.ipynb
README.md
LICENSE
.gitignore
```

## Reuse and adaptation

The repository is intentionally structured in a way that allows further modification.

Users may:

- update the input data;
- revise the score construction logic;
- modify thresholds and category labels;
- refine the visual design;
- adapt the project for other territories or regional datasets.

## Notes

This publication is intended to make the implementation transparent, reviewable, and reusable. It reflects one specific project-stage version of the map and should be interpreted accordingly.
