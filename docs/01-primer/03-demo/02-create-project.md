---
layout: default
title: Create project
parent: Demo
grand_parent: Primer
nav_order: 2
permalink: docs/primer/demo/create/
---

# Create project
{: .no_toc }

## Table of contents
{: .no_toc .text-delta }

1. TOC
{:toc}

---

## Spring initializr

1. Access [Spring initializr https://start.spring.io/](https://start.spring.io/)

1. Configure the application

   ![Spring initializr]({{ '/assets/images/Spring-Initializr.png' | absolute_url }})

   | Option      | Selection      |
   | ----------- | -------------- |
   | Project     | Gradle Project |
   | Language    | Java           |
   | Spring Boot | 2.3.1          |

   **Project Metadata**

   | Option       | Selection       |
   | ------------ | --------------- |
   | Group        | demo            |
   | Artifact     | contact-us      |
   | Name         | contact-us      |
   | Description  | Contact Us Demo |
   | Package name | demo.boot       |
   | Packaging    | jar             |
   | Java         | 14              |

   **Dependencies**

   | Dependencies |
   | ------------ |
   | Lombok       |
   | Spring Web   |

1. (_Optional_) Explore the project

   Click _EXPLORE_ to view the project

   ![Spring initializr]({{ '/assets/images/Spring-Initializr.png' | absolute_url }})

1. Download (or generate) the project

   Click _DOWNLOAD_ (or _GENERATE_) to download the zip file

   ![Spring initializr Explore]({{ '/assets/images/Spring-Initializr-Explore.png' | absolute_url }})

## Download preconfigured project

The application can also downloaded from [contact-us.zip]({{ '/assets/demo/01-primer/contact-us.zip' | absolute_url }})
