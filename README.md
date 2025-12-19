# Furious5's Cybersecurity Research Blog

## Overview

This repository hosts a publicly accessible cybersecurity research blog available at [https://furious-05.github.io/](https://furious-05.github.io/). The blog contains a comprehensive collection of technical writeups, penetration testing walkthroughs, and security research. It documents detailed analyses of Capture The Flag (CTF) competitions, HackTheBox machine exploitations, vulnerability research, and practical security topics. Each writeup includes exploitation techniques, proof-of-concept code, and methodologies relevant to modern cybersecurity practices.

## Purpose

This blog serves multiple objectives:

- Share detailed technical walkthroughs for cybersecurity challenges with the security community
- Provide practical learning resources for penetration testing and ethical hacking
- Document vulnerability research and exploitation techniques
- Contribute to the cybersecurity community through publicly available knowledge
- Maintain a personal portfolio of security research and accomplishments

## Public Access

The blog is publicly hosted on GitHub Pages and accessible to anyone interested in cybersecurity education and research. All content is available for reading and learning purposes.

## Content Structure

The repository is organized into the following categories:

### HackTheBox Walkthroughs

Complete machine exploitation writeups covering:
- Linux and Windows target systems
- Web application vulnerabilities (IDOR, command injection, etc.)
- Privilege escalation techniques
- Active Directory exploitation
- Network services enumeration and exploitation

### CTF Challenge Solutions

Detailed writeups from various CTF competitions including:
- BlackHat Qualifications
- Themed CTF challenges
- Specific vulnerability demonstrations

### Security Research Articles

In-depth technical articles covering:
- Active Directory Certificate Services (ADCS) exploitation
- Kerberos attacks and exploitation
- Web cache deception techniques
- Linux privilege escalation
- Hash cracking and credential harvesting

### Topics Covered

Active Directory, ADCS, ACL Abuse, Command Injection, CVE Analysis, DACLABUSE, DCSYNC, DNS, DPAPI, Kerberoast, Privilege Escalation, Web Application Security, Linux Security, Lateral Movement, and more.

## Installation and Local Development

### Prerequisites

- Ruby 3.0 or higher
- Bundler package manager
- Git

### Setup Instructions

Clone the repository:

```bash
git clone https://github.com/furious-05/furious-05.github.io.git
cd furious-05.github.io
```

Install dependencies:

```bash
bundle install
```

Start the local development server:

```bash
bundle exec jekyll serve
```

The site will be available at `http://127.0.0.1:4000/`

### Auto-Regeneration

The development server includes automatic regeneration. Any changes to files will be automatically reflected in the browser.

## File Structure

```
.
├── _config.yml              # Site configuration
├── _posts/                  # Blog post markdown files
├── _tabs/                   # Static pages (About, Archives, Categories, Tags)
├── _data/                   # Data files (contact, sharing options)
├── assets/                  # Images and resources
├── _plugins/                # Custom Jekyll plugins
└── index.html               # Homepage template
```

## Creating New Posts

To create a new writeup or article, add a markdown file to the `_posts/` directory with the following naming convention:

```
YYYY-MM-DD-Post-Title.md
```

Include front matter metadata:

```yaml
---
title: Post Title
date: YYYY-MM-DD
categories: [Category]
tags: [tag1, tag2, tag3]
---
```

## Important Notes

- Code blocks containing Liquid template syntax (curly braces) should be wrapped with `{% raw %}` and `{% endraw %}` tags to prevent template processing errors
- All images and resources should be placed in the `assets/` directory with appropriate subdirectories
- Posts are organized by category and tagged for easy navigation and search

## Build and Deployment

The site is automatically deployed to GitHub Pages on every push to the main branch via GitHub Actions workflow.

To manually build the site:

```bash
bundle exec jekyll build
```

The output will be generated in the `_site/` directory.

## Author

Muneeb Nawaz (Furious5)

Email: muneebnawaz3849@gmail.com

GitHub: [furious-05](https://github.com/furious-05)
