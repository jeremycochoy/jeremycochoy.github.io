Jeremy Cochoy
=============

Personal website. Contains blog posts, music, articles and resources
related to math and computer science and teaching.


Deployment process
------------------

**Quick deployment:**
Compile and deploy to master the development branch:
`bundle exec jgd -r development -b master`

### Website Deployment

**Repository Structure:**
- **Development branch**: `development` - where active development happens
- **Master branch**: `master` - contains compiled/built website files 
- **GitHub Pages branch**: `gh-pages` - older deployment branch (last updated Nov 2019)

**Technology Stack:**
- Jekyll static site generator with Ruby
- Uses the `jgd` gem for GitHub deployment
- Domain: `www.cochoy.fr` (configured in CNAME file)
- Remote repository: `git@github.com:jeremycochoy/techgate.git`

**Deployment Flow:**
1. Content is developed in `development` branch with source files (.md, _posts/, _layouts/, etc.)
2. Jekyll builds the static site into the `_site/` directory 
3. The built site is committed to the `master` branch using: `bundle exec jgd -r development -b master`
4. GitHub Pages serves the site from the `master` branch at www.cochoy.fr

The `master` branch contains only the compiled HTML/CSS files, while `development` contains the Jekyll source files.

### CV Deployment Process

**CV Generation Pipeline:**
The CV exists in multiple formats generated from a single source:

- **Source**: `resume/index.md` (Markdown format)
- **Generated formats**:
  - `index.html` - HTML version for web display
  - `index.pdf` - PDF version
  - `index.docx` - Word document format
  - `index.txt` - Plain text version
  - `index.tex` - LaTeX source (uses ConTeXt for PDF generation)

**Build Process:**
The CV uses ConTeXt (TeX-based) styling for professional formatting. The `.tex` file contains the styling definitions, and the build process converts the Markdown source through LaTeX to generate the final PDF and other formats.

**Web Integration:**
The CV is accessible at `/resume/` and `/cv/` paths on the main website, with the HTML version integrated into the Jekyll site structure.
