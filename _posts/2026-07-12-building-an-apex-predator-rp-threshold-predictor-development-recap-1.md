---
layout: post
title: 'Building an Apex Predator RP Threshold Predictor: Development Recap #1'
description: An overview of the project's motivation, data sources, and the pipeline
  used to collect, clean, and store historical Apex Predator RP threshold data.
categories:
- Personal Project
tags:
- ML
- Apex Legends
- Web Development
toc: true
comments: true
pin: false
date: 2026-07-12 20:49 -0700
---
## Introduction

In this post, I will introduce the background of my [Apex Predator RP Threshold Predictor](https://pred.lucien2714.com) and explain how the project collects and cleans its historical data.

## Background

As an enthusiastic Apex Legends player, I have always found the process of grinding for Predator, the top 750 rank, exciting. However, it is also extremely time-consuming.

Some players earn a large amount of RP during the middle of a season, stop playing ranked, and hope that their RP will remain above the Predator cutoff until the season ends. Past experience has shown that this strategy can be risky. Players may find themselves falling out of the top 750 near the end of the season and then be forced to continue grinding when competition is at its highest.

Late-season ranked matches can also contain more cheaters and boosters because many players are desperate to gain RP before the season ends.

I wanted to build a tool that could provide players with a more accurate estimate of the final Predator RP threshold. This could help them decide how much RP they need and reduce the chance of unexpectedly falling out of Predator near the end of a season.

## Data Sources

[Apex Legends Status](https://apexlegendsstatus.com/), commonly referred to as ALS, is an important third-party statistics website for Apex Legends players. It provides a wide range of ranked statistics, including historical Predator RP thresholds across multiple seasons.

ALS also provides an API for accessing the current Predator RP threshold. This project uses data provided by ALS as its primary data source.

Special thanks to Hugo for building and maintaining this useful website.

## Historical Data Collection and Cleaning

According to the [ALS API documentation](https://apexlegendsapi.com/), there is no documented API endpoint for retrieving the complete history of Predator RP thresholds. The official `/predator` endpoint only provides the current threshold.

However, historical data is displayed on the ALS [Points for Predator](https://apexlegendsstatus.com/points-for-predator) page. After inspecting the page's network requests, I found that the charts load their historical data from publicly accessible JSON resources used internally by the website.

Because these resources are undocumented and are not part of the official public API, I will not disclose their exact endpoints. The project used them only once to retrieve historical data; all new data will be collected through the official `/predator` endpoint.

The threshold dataset follows a structure similar to the format used by Chart.js:

```text
Date        → ["05/01", "05/02", ...]
PC          → [31200, 31500, ...]
Playstation → [...]
Xbox        → [...]
Switch      → [...]
```

The project also retrieves the historical Masters and Predator population data used by the page's charts.

The date labels only include the month and day, so the program must reconstruct the full date for each record. It begins with the newest available date and assigns it to the most recent applicable calendar year. It then processes the remaining dates in reverse chronological order and decreases the year whenever it crosses a year boundary.

Both datasets are normalized into a long-format table:

```text
platform | date       | metric        | value
PC       | 2026-06-01 | rp_cutoff     | 31200
PC       | 2026-06-01 | masters_preds | 85000
```

During the cleaning process, the program removes missing, malformed, and otherwise invalid RP values. It then detects ranked split resets by identifying unusually large decreases in the Predator RP threshold.

Each group of records between two detected resets is assigned a split label, such as `split_31`. The cleaned and labeled records are then stored in a SQLite database.

This historical dataset serves as the foundation for feature engineering and model training, which I will discuss in the next development post.
