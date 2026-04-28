# GitHub Copilot Workshop - April 2026

Welcome to the GitHub Copilot Workshop training materials repository! This repository contains comprehensive resources to help you master GitHub Copilot across different domains and skill levels.

## 📚 Workshop Materials

This workshop includes the following training modules:

### 1. [Tips, Tricks and Best Practices for GitHub Copilot](./Tips%20Tricks%20and%20Best%20Practices%20for%20GitHub%20Copilot_v1.4.pdf)
Essential fundamentals and best practices for effective Copilot usage:
- Context strategies (open files, top-level comments, meaningful names)
- Inline Chat (Ctrl+I) and slash commands
- GitHub Copilot CLI for terminal workflows
- Copilot Chat best practices (threads, #codebase, highlighting code)
- Role prompting and AI hallucination awareness
- Plan mode vs Agent mode comparison
- MCP Server integration
- Chat participants and variables (#file, #selection, @workspace)
- "Let's think step by step" decomposition technique

**🎥 [Watch the recording](#)** <!-- TODO: Replace with recording URL -->

### 2. [GitHub Copilot Developer](./GitHub%20Copilot%20Developer_v1.4.pdf)
Comprehensive developer workshop covering coding, customization, and code quality:
- Custom Instructions and Custom Agents (hands-on exercises)
- Agent Skills, Plugins & Marketplace ecosystem
- In-file Copilot options and data generation
- Test-Driven Development (TDD) with Copilot
- Code profiling and debugging
- `.copilotignore` for blocking sensitive files
- Pull requests, knowledge bases, and Copilot Enterprise
- GitHub Copilot Workspace and Coding Agent
- Prompt files and code refactoring
- Unit test generation and `/fix`, `/tests`, `/troubleshoot` commands

**🎥 [Watch the recording](#)** <!-- TODO: Replace with recording URL -->

### 3. [GitHub Copilot DevOps](./Github%20Copilot%20DevOps_v1.4.pdf)
Leverage GitHub Copilot for DevOps workflows:
- Dockerfile generation
- Infrastructure as Code (Terraform, Bicep/ARM templates)
- CI/CD pipeline generation
- Script writing (Bash, PowerShell)
- Alerts and monitoring configuration
- Log analysis and troubleshooting
- Terminal commands with @terminal
- `/fix`, `/tests` and other slash commands
- Code conversion between languages
- Wiki and markdown documentation generation
- MCP Server integration
- `.copilotignore` for sensitive files

**🎥 [Watch the recording](#)** <!-- TODO: Replace with recording URL -->

### 4. [GitHub Copilot ML Data Ops](./Github%20Copilot%20ML%20Data%20Ops_v1.4.pdf)
Specialized training for ML and Data Operations:
- Schema generation with foreign key constraints
- Data generation for development and testing
- SQL query generation from context and comments
- Library imports and comment-driven development
- CI/CD pipeline generation for ML workflows
- Feature engineering and hyperparameter tuning
- Parallel processing optimisation
- Exploratory Data Analysis (EDA) with Copilot
- Code explanation with `/explain`
- Unit test generation for helper functions

**🎥 [Watch the recording](#)** <!-- TODO: Replace with recording URL -->

### 5. [Introduction to Prompt Engineering for GitHub Copilot](./Introduction%20to%20Prompt%20Engineering%20for%20GitHub%20Copilot_v1.4.pdf)
Advanced techniques for writing effective prompts:
- The 4S principles: Single, Specific, Short, Surround
- Zero-shot, one-shot, and few-shot prompting techniques
- Cue-based and supporting content techniques
- Role prompting for focused results
- Understanding LLMs and model selection
- Context windows and token management
- Copilot prompt processing flow (inbound and outbound)
- Prompt types: direct questions, code requests, open-ended queries, contextual prompts

**🎥 [Watch the recording](#)** <!-- TODO: Replace with recording URL -->

## 🎯 Workshop Objectives

By the end of this workshop, you will be able to:
- ✅ Write effective prompts to generate high-quality code
- ✅ Integrate GitHub Copilot into your daily development workflow
- ✅ Create custom instructions to enforce consistent coding standards
- ✅ Build custom agents and agent skills for specialized workflows
- ✅ Use Copilot for DevOps automation and infrastructure management
- ✅ Apply Copilot to ML/Data science projects
- ✅ Leverage the plugin and marketplace ecosystem
- ✅ Maximize productivity with advanced tips and tricks

## 🚀 Getting Started

1. **Download the materials**: All PDF files are included in this repository
2. **Follow the sequence**: We recommend starting with Tips, Tricks and Best Practices
3. **Practice**: Apply what you learn in your own projects
4. **Experiment**: Try different approaches and see what works best for you

## 📖 Recommended Learning Path

```
1. Tips, Tricks and Best Practices (Fundamentals)
   ↓
2. GitHub Copilot Developer
   ↓
3. Choose your focus:
   - DevOps → GitHub Copilot DevOps
   - ML/Data → GitHub Copilot ML Data Ops
   ↓
4. Introduction to Prompt Engineering (Advanced)
```

## 🧪 Hands-On Labs

### Prerequisites

- VS Code with GitHub Copilot and Copilot Chat extensions
- GitHub Copilot license active
- GitHub CLI installed
- Clone the [AccelerateDevGHCopilot](https://github.com/shin-akuma/AccelerateDevGHCopilot) project (C#/.NET Library Management System)

### Developer Labs

| # | Lab | Duration | Level |
|---|-----|----------|-------|
| 1 | [Custom Instructions and Custom Agents](./Lab1_Custom_Instructions_and_Agents.md) | 45-60 min | Beginner |
| 2 | [Test-Driven Development with GitHub Copilot](./Lab2_Test_Driven_Development_with_Copilot.md) | 60-75 min | Intermediate |
| 3 | [Code Quality, Refactoring & Debugging](./Lab3_Code_Quality_Refactoring_Debugging.md) | 50-60 min | Intermediate-Advanced |

### Microsoft Learning Labs

| # | Lab | Lab Files |
|---|-----|-----------|
| 4 | [Analyze Document Code](https://microsoftlearning.github.io/mslearn-github-copilot-dev/Instructions/Labs/LAB_AK_02_analyze_document_code.html) | [📦 Download](https://github.com/MicrosoftLearning/mslearn-github-copilot-dev/raw/refs/heads/main/DownloadableCodeProjects/Downloads/AZ2007LabAppM2.zip) |
| 5 | [Develop Code Features](https://microsoftlearning.github.io/mslearn-github-copilot-dev/Instructions/Labs/LAB_AK_03_develop_code_features.html) | [📦 Download](https://github.com/MicrosoftLearning/mslearn-github-copilot-dev/raw/refs/heads/main/DownloadableCodeProjects/Downloads/AZ2007LabAppM3.zip) |
| 6 | [Develop Unit Tests with xUnit](https://microsoftlearning.github.io/mslearn-github-copilot-dev/Instructions/Labs/LAB_AK_04_develop_unit_tests_xunit.html) | [📦 Download](https://github.com/MicrosoftLearning/mslearn-github-copilot-dev/raw/refs/heads/main/DownloadableCodeProjects/Downloads/AZ2007LabAppM4.zip) |

### Specialized Labs

| # | Lab | Duration | Focus |
|---|-----|----------|-------|
| 7 | [Copilot with Terraform](https://github.com/shin-akuma/copilot-terraform) | 30-45 min | IaC with Terraform, Azure DevOps pipeline generation |
| 8 | [Cloud Lab — Scripting](https://experience.cloudlabs.ai/#/labguidepreview/f9fd80ac-fc1a-4609-a3f1-06650aec389e) (Page 10) | 20-30 min | Scripting exercises in a provisioned cloud environment |
| 9 | [Copilot for Data Engineering](https://github.com/CleveritDemo/copilot-data-engineering/tree/main) | 30-45 min | PySpark RDD/DataFrame operations, SQL query generation, data profiling |
| 10 | [Copilot for ML & Data Ops](https://github.com/arinco-crew-community/copilot-ml-data-ops/tree/main) | 45-60 min | SQL schema generation, penguin species classification, ML Q&A |

### Additional Labs

- [Generate Documentation Challenge](https://learn.microsoft.com/en-gb/training/modules/generate-documentation-using-github-copilot-tools/6-exercise-complete-code-documentation-challenge)
- [Interactive Cloud Lab Environment](https://experience.cloudlabs.ai/#/labguidepreview/f9fd80ac-fc1a-4609-a3f1-06650aec389e)

## 💡 Additional Resources

- [GitHub Copilot Official Documentation](https://docs.github.com/en/copilot)
- [GitHub Copilot Blog](https://github.blog/tag/github-copilot/)
- [Awesome GitHub Copilot](https://github.com/github/awesome-copilot)

## 🤝 Contributing

If you have suggestions for improving these materials or find any issues, please feel free to open an issue or submit a pull request.

## 📝 License

This training material is provided for educational purposes.

---

**Workshop Date**: April 2026  
**Last Updated**: April 7, 2026
