# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Repository Overview

This is a documentation repository for **The Cozm** containing:
1. **Cozm Travels API Documentation** - Complete API reference for A1, COC, and Immigration certificate applications
2. **Email Campaign Playbook** - Internal guide for producing email outreach campaigns

This is a documentation-only repository with no build process, tests, or application code.

## Repository Structure

```
cozm-docs/
├── cozm-travels/
│   └── README.md          # API documentation for Cozm Travels compliance API
├── playbooks/
│   └── email-campaign.html # Email campaign production guide
└── README.md              # Root repository description
```

## Content Architecture

### Cozm Travels API Documentation (`cozm-travels/README.md`)

Complete API reference covering:
- **Authentication**: Bearer token-based auth using `id_token`
- **Form Fields Endpoint**: Dynamic field retrieval based on country and compliance type
- **File Upload**: Pre-signed S3 URLs for document uploads
- **Application Submission**: Creating A1/COC/ETA/BV applications
- **Application Listing**: Paginated retrieval of submitted applications

Key compliance types supported:
- `A1` - EU social security coverage (single-country)
- `MSW-A1` - Multi-State Worker A1 (multi-country)
- `COC` - Certificate of Coverage (non-EU)
- `ETA` - Electronic Travel Authorization
- `BV` - Business Visa

**API Base URL**: `https://api.development.cozmtravels.democozm.com/`

The documentation uses markdown format and includes:
- Field attribute explanations (name, type, group, persona, conditional_fields, etc.)
- Data type reference with validation requirements
- Conditional field logic for dynamic forms
- cURL examples for all endpoints

### Email Campaign Playbook (`playbooks/email-campaign.html`)

Internal team guide documenting the 8-step workflow for email campaigns:
1. Research (Gemini Deep Research)
2. Build contact list (Google Sheets)
3. Validate emails (Emailable)
4. Find images (Pixabay)
5. Host images (Cloudinary)
6. Design HTML email
7. Test send
8. Full send (SendGrid)

Uses The Cozm brand colors throughout (teal `#44919c`, red `#bd4040`, gold `#bd8941`, etc.)

Includes the Christmas 2025 campaign as a reference example with embedded email content.

## Editing Documentation

### When updating API docs (`cozm-travels/README.md`):
- Maintain the existing markdown table format for consistency
- Keep cURL examples updated with actual endpoint URLs
- Preserve the numbered section structure (1-8)
- Document all field attributes in the field reference tables
- Include example request/response payloads for new endpoints

### When updating playbooks (`playbooks/*.html`):
- Use inline CSS only (email clients strip `<style>` tags)
- Maintain The Cozm brand color variables in `:root`
- Keep table-based layouts for email templates
- Include working examples of emails sent
- Update version and date at the top of playbooks

### Brand Colors (The Cozm)
```css
--cozm-teal: #44919c;
--cozm-teal-light: #c7e5e9;
--cozm-red: #bd4040;
--cozm-red-light: #f1cfcf;
--cozm-purple: #ac40bd;
--cozm-gold: #bd8941;
--cozm-gold-light: #f7e6ce;
--cozm-black: #121212;
--cozm-white: #FFFFFF;
--cozm-grey: #f3f3f3;
```

## Git Workflow

This repository uses standard git version control:
- Main branch: `main`
- Commits should describe documentation changes clearly
- Use conventional commit messages (e.g., "Add BV endpoint documentation", "Update email campaign playbook")

## Important Notes

- **No code execution**: This repository contains only documentation (markdown and HTML)
- **No dependencies**: No package.json, no build process, no tests
- **API credentials**: All sensitive credentials in the docs are marked as `**redacted**`
- **Cloudinary**: Images are hosted on The Cozm Cloudinary account (cloud name: `dtrayn0x8`)
- **SendGrid**: Email sending uses authenticated domain `thecozm.com`
