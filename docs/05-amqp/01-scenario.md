---
layout: default
title: Scenario
parent: Advanced Message Queuing Protocol
nav_order: 1
permalink: docs/amqp/scenario/
---

# Scenario
{: .no_toc }

## Table of contents
{: .no_toc .text-delta }

1. TOC
{:toc}

---

## Scenario

The business team are very happy with our _Contact Us_ application and its usage and would like to add a new feature to streamline events and catering.

Offices organise lots of events where interested individual can register and attend.  These events also include catering.  The attendees submit their food preference on registration which is then compiled in a list and provided to the food supplier.  This is a manual and elaborate process which needs to be automated.

Mistakes happen and some attendees may not get the food they selected.  This can be very frustrating, especially for individuals who are selective with their diet.  An ideal solution would be one where we can link the prepared food to the attendee without disclosing personal information.

![AMQP-Registration-Flow.png]({{ '/assets/images/AMQP-Registration-Flow.png' | absolute_url }})

The food caterer provides an [application](https://github.com/albertattard/event-food-demo) that handles food ordering for events.  The food supplier is also willing to improve the existing application to improve its accessibility.

## Objectives

- [ ] Provide an endpoint from where attendees can register to an event
- [ ] Submit the food preference to the food supplier as part of the registration process
- [ ] Create an application that receives the attendees' food preferences
