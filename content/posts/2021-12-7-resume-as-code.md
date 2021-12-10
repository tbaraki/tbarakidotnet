---
date: "2021-12-07"
draft: false
slug: resume-as-code
title: Building and deploying a "resume-as-code"
---

I recently decided to test out the job market and look for a new challenge. My plan was to:

1. Update my resume to capture the last nine years of experience
2. Build a simple and performant personal landing page to host my resume
3. Keep a better inventory of my work history and projects for future updates

## Concept

My previous resume had been created using [LaTeX](https://www.latex-project.org/). I enjoyed this approach as it meant that the content could be decoupled from the styling and layout of the document. I was looking for something similar but more approachable. A quick search brought me to the [JSON Resume](https://jsonresume.org/) and [yaml-resume](https://yaml-resume.com/) projects. I was intrigued by the idea of keeping work history and personal details in a structured data file, YAML in particular. I've used Static Site Generators in the past, such as [Hugo](https://gohugo.io) or [Gatsby](https://www.gatsbyjs.com/), which accomplish something similar. Page _content_ can be written in simple Markdown and the resulting pages are styled at build time. It seemed natural to combine these two ideas and create a static resume website based off structured data files.

## Breakthrough

Fortunately for me, the [Resume A4](https://mertbakir.gitlab.io/projects/resume-a4/) theme already exists for Hugo. This particular implementation constrains the web layout to an ISO A4 sized page for easy convertion to PDF or for print. I made a few styling tweaks and converted the page sizing to US Letter (8.5x11") to match my locale.

```css
@media print {
  @page {
    size: 8.5in 11in;
    margin: 0;
  }

.paper {
  width: 215.9mm;
  height: 279.4mm; 
```

Now I simply had to record my work history and personal info in YAML files:

```yaml
- company: Spreckels Union School District
  roles: 
    - role: Information Services Director
      date: "July 2015 - Present"
```
## Deployment

With the content created, I needed a place to host the finished product. I decided to deploy to my Azure subscription using Azure Static Web Apps. The CI/CD workflow is simple yet powerful. It triggers on main branch pushes _or_ pull requests. This means that a pull request from a feature branch will automatically generate a live staging environment in Azure to preview the changes. Very cool!

```yaml
name: Azure Static Web Apps CI/CD

on:
  push:
    branches:
      - main
  pull_request:
    types: [opened, synchronize, reopened, closed]
    branches:
      - main
```

## Benefits

I see a few major benefits from building a resume this way:

* Structured data allows me to cleanly and concisely document my history and experiences.
* Source control (git) makes it easy to version and implement changes.
* CI/CD pipelines make deploying changes effortless.
* A live resume website provides a positive web presence and acts as a portfolio piece.

I'm thrilled with [the end result](https://resume.tbaraki.net) and love that I can continuously deploy tweaks and updates in the future.

