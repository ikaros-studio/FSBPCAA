# FSBPCAA
**Facilitating Sustainable Behavior through Personalized Conversational AI Agents**
This repository contains the thesis implementation and resources investigating the effectiveness of personalized conversational AI agents in promoting sustainable behavior.

## Project Overview

The project consists of two complementary studies exploring different approaches to environmental communication through AI:

### Study 1: Text-Based Conversational Agents
- Multiple persuasion strategies (Future-oriented, Action-oriented, Moral foundation)
- Personalized vs. non-personalized variants
- Control conditions with static text

### Study 2: Interactive 3D Marine Life Agents
- Animated 3D models of marine creatures (Beluga, Jellyfish, Seahorse)
- Voice-enabled interactions
- Personalized environmental storytelling

## Getting Started

### Prerequisites
- Node.js and npm
- Vue.js 3
- OpenAI API key
- 3D models for Study 2
- Optional Google Sheet Script with API endpoint to Log Conversations

### Setup Instructions

1. **Application Setup:**
   ```bash
   # For Study 1
   cd interfaces/study1
   npm install
   npm run dev

   # For Study 2
   cd interfaces/study2
   npm install
   npm run dev
   ```

2. **Environment Configuration:**
   - Replace `OPEN AI KEY NEEDED` placeholders with your OpenAI API key
   - Required for chat functionality and text-to-speech features
   - Optional: Replace `YOUR GOOGLE SHEET URL HERE` with your Google Sheet enabled to POST conversations via AppsScript

3. **3D Models (Study 2):**
   Download and place the following models in `interfaces/study2/public/models/`:
   - [Beluga Model](https://www.fab.com/listings/da1aff52-a59b-4919-9b78-1aae5e39af45)
   - [Jellyfish Model](https://www.fab.com/listings/76399bae-fef8-4d38-8f90-6867da2d0c67)
   - [Seahorse Model](https://www.fab.com/listings/af35f093-13b9-49c8-b3b9-fbe8116999a1)

## Repository Structure

```
FSBPCAA/
├── interfaces/
│   ├── study1/         # Study 1: Text-based intervention
│   └── study2/         # Study 2: 3D model intervention
├── analysis/
│   └── stata/          # Statistical analysis scripts and commands
└── results/            # Study results and data
```

## Features

### Study 1
- Multiple persuasion strategies
- Dynamic chat interactions
- Personalization based on user responses
- Data logging and analysis capabilities

### Study 2
- Real-time 3D model animations
- Voice synthesis integration
- Interactive storytelling
- Environmental education through marine life personas

## Analysis Tools

The project includes comprehensive statistical analysis tools:
- Reliability analysis
- Descriptive statistics
- Correlation analysis
- Regression analysis
- ANOVA with post-hoc comparisons

Detailed Stata commands and procedures are documented in `analysis/stata/stataCommands.md`.

## Documentation

- Each study directory contains detailed implementation documentation
- Analysis procedures are documented in the analysis directory
- Results and visualizations are available in the results directory

## License

This project is licensed under [CC BY-NC 4.0](https://creativecommons.org/licenses/by-nc/4.0/):
- ✓ Free to share and adapt
- ✗ No commercial use
- ✓ Attribution required
- ✓ Share-alike required

## Contact

For questions, assistance, or collaboration inquiries, please contact the repository maintainer.

---

**Note:** This repository is part of academic research. Please cite appropriately when using or referencing this work.

