---
layout: default
title: Scenario
parent: Custom Queries
grand_parent: Spring Data
nav_order: 1
permalink: docs/data/custom-queries/scenario/
---

# Scenario
{: .no_toc }

## Table of contents
{: .no_toc .text-delta }

1. TOC
{:toc}

---

## Scenario

Customers using our API came back with feedback.  They would like to filter offices by country.  For example, given the country, `"germany"`, our repository should return all offices that belong to this country, case-insensitive.  ThoughtWorks is promoting the self-service approach and offices can modify the office details and delete offices that are not operating anymore.

## Tasks

- [ ] Return one office by id
- [ ] Return all offices in country
- [ ] Update individual office details
- [ ] Delete an office
