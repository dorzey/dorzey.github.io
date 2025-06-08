---
layout: post
title:  "Continuous deployment of machine learning models"
date:   2019-05-16 12:00:00 +0200
---

<iframe width="560" height="315" src="https://www.youtube.com/embed/wnx5yYVf2hQ?si=L4bmphsTt7mpFWpf" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share" referrerpolicy="strict-origin-when-cross-origin" allowfullscreen></iframe>

This talk describes how Auto Trader launched a suite of new machine learning models with the ability serve low-latency predictions in real time. These models are automatically retrained and redeployed using continuous deployment pipelines in our existing deployment infrastructure, making use of technology including Apache Spark, Airflow, Docker and Kubernetes. Since models are deployed without manual intervention, we developed a robust testing strategy to ensure deployments will not cause a drop in model performance, including accuracy and coverage.