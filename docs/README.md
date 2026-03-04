# Documentation Structure

This directory contains the product documentation for Arambh AI by Arambh Labs.

## File Organization

```
docs/
├── index.md                 # Documentation homepage
├── deployment-guide.md      # Deployment and installation guide
├── integrations-guide.md    # Third-party integrations guide
└── user-guide.md           # End-user documentation
```

## Documentation Pages

### 1. Deployment Guide (`deployment-guide.md`)
Comprehensive deployment documentation covering:
- Prerequisites and system requirements
- Installation steps
- Configuration options
- Multiple deployment methods (Docker, Kubernetes, Cloud)
- Post-deployment verification
- FAQ and troubleshooting

### 2. Integrations Guide (`integrations-guide.md`)
Integration documentation including:
- Authentication setup
- Pre-built integrations (Slack, Teams, Jira, GitHub, GitLab, etc.)
- Database connections
- Cloud storage integrations
- Monitoring and analytics tools
- Custom integrations via API
- Webhooks
- FAQ and troubleshooting

### 3. User Guide (`user-guide.md`)
End-user documentation covering:
- Getting started and onboarding
- Dashboard overview
- Core features and functionality
- Account and security management
- Settings and preferences
- Best practices
- FAQ and troubleshooting

## Editing Documentation

### Front Matter
Each documentation page includes Jekyll front matter:

```yaml
---
layout: page
title: Page Title
permalink: /docs/page-name/
nav_order: 1
---
```

### Structure
All guides follow a consistent structure:
1. Overview/Introduction
2. Table of Contents
3. Main Content Sections
4. FAQ Section
5. Troubleshooting Section

### Style Guidelines
- Use clear, concise language
- Include code examples where applicable
- Add screenshots for UI-related instructions
- Keep troubleshooting solutions actionable
- Update "Last Updated" dates when making changes

## Building Locally

To preview documentation changes locally:

```bash
bundle exec jekyll serve
```

Then visit `http://localhost:4000/docs/`

## Contributing

When adding or updating documentation:
1. Follow the existing structure and style
2. Test all code examples
3. Update the table of contents if adding new sections
4. Update cross-references between documents
5. Run a local build to verify formatting

## Support

For documentation questions or issues:
- Open an issue in the repository
- Contact the documentation team
- Submit a pull request with improvements
