---
description: Repository Information Overview
alwaysApply: true
---

# Repository Information Overview

## Repository Summary
This repository serves as the personal GitHub profile and workspace for João Francisco Gomes Aschenbrenner, a Full-Stack Developer specialized in AI-driven development. It contains personal portfolio information, assets, and a significant subproject, **Devicon**, which is a comprehensive collection of programming-related icons.

## Repository Structure
- **.github/**: Contains GitHub Actions workflows (e.g., `Cobrinha.yml` for profile animations).
- **assets/**: Stores project logos and branding materials.
- **devicon-master/**: The core subproject containing a vast collection of icons in SVG and font formats.
- **.cursor/ / .zencoder/ / .zenflow/**: Configuration directories for AI-assisted development tools and workflows.

### Main Repository Components
- **Personal Profile**: Represented by the root `README.md`, detailing skills, experience, and AI-driven methodology.
- **Devicon Project**: A specialized toolset for developers providing icons as SVG or web fonts.

## Projects

### Devicon (Icon Collection)
**Configuration File**: `devicon-master/package.json`

#### Language & Runtime
**Language**: JavaScript (Gulp tasks), Python (Automation scripts)  
**Version**: Node.js (via package.json), Python 3.x  
**Build System**: Gulp  
**Package Manager**: npm

#### Dependencies
**Main Dependencies**:
- `gulp`: Task runner for building CSS and optimizing SVGs.
- `sass`: CSS preprocessor.
- `yargs`: Command-line argument parsing.

**Development Dependencies**:
- `gulp-sass`, `gulp-svgmin`, `gulp-footer`: Gulp plugins for asset processing.

#### Build & Installation
```bash
# Install dependencies
cd devicon-master
npm install

# Build CSS and minify
npm run build-css

# Optimize SVG files
npm run optimize-svg
```

#### Main Files & Resources
- **devicon.json**: The master list of all available icons and their versions.
- **icons/**: Directory containing raw SVG files for all logos.
- **fonts/**: Generated web font files.
- **devicon.min.css**: The main stylesheet for using icons in web projects.

#### Testing & Validation
**Framework**: Custom Python scripts using Selenium (`icomoon_build.py`, `icomoon_peek.py`).  
**Test Location**: `devicon-master/.github/scripts/`  
**Run Command**:
```bash
# Run build tests (requires geckodriver)
npm run build-test

# Peek test for PRs
npm run peek-test
```

### Personal Portfolio (Profile)
**Type**: Personal documentation and resource repository.

#### Key Resources
- **README.md**: Main profile page with professional history and technical skills.
- **assets/logos/**: Visual assets used in the profile and associated projects.

#### Usage & Operations
The root repository is primarily used for GitHub Profile presentation. Changes to the profile are reflected by editing the root `README.md`.
