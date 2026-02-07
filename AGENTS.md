# Console Whisperer Deployment

## Project Overview
`console-whisperer-deploy` is the landing page and deployment repository for **Console Whisperer**, an AI-powered mixing console setup assistant. The tool helps live sound engineers configure channel names and colors on mixing consoles (specifically targeting **Mixing Station**) using natural language.

The project is a static website designed to be hosted on GitHub Pages, providing a professional landing page for marketing and a dedicated download portal for the desktop application installers.

### Key Technologies
- **Frontend:** HTML5, Tailwind CSS (via CDN), Google Fonts (Space Grotesk).
- **Hosting:** GitHub Pages.
- **Packaging:** [Hydraulic Conveyor](https://hydraulic.dev) is used to generate the download page and package the desktop application for Windows and macOS.

## Architecture and Key Files
- `index.html`: The main landing page featuring the product value proposition, "Buy Now" CTA (linking to Polar.sh), and links to the download page.
- `download.html`: A dynamically generated download page that detects the user's OS (Windows/macOS) and provides the appropriate installer links from GitHub Releases.
- `icon.svg`: The official logo for Console Whisperer.
- `CNAME`: Configures the custom domain for the GitHub Pages site.
- `AGENTS.md`: Instruction file for AI agents (currently empty).

## Building and Running

### Local Development
Since this is a static project, you can run it locally without any build steps:
1. Open `index.html` in any modern web browser.
2. Alternatively, use a simple HTTP server:
   ```bash
   # Using Node.js
   npx serve .
   
   # Using Python
   python3 -m http.server
   ```

### Deployment
Deployment is handled via Git. Pushing changes to the main branch will update the live site if GitHub Pages is configured to track that branch.
- **Target URL:** `https://livenathan.github.io/console-whisperer-deploy/` (or the custom domain specified in `CNAME`).

## Development Conventions
- **Styling:** Utility-first CSS using Tailwind. Avoid adding custom CSS files unless necessary; use the `<script>` config block in `index.html` for theme extensions.
- **Assets:** Use SVG for icons (like `icon.svg`) to ensure scalability and small file sizes.
- **Links:** Ensure download links in `download.html` point to the correct GitHub Release tags or `latest`.

## Usage
This directory serves as the public-facing entry point for users to:
1. Learn about Console Whisperer's features.
2. Purchase the application via Polar.sh.
3. Download the desktop application for their specific platform.
