# Website update checklist and resources

- [x] Change the font
- [x] Frontmatter: Update publications
  - [x] Check formatting
  - [x] Add featured images

- [ ] CV: Update CV; update PDF of CV
- [ ] Posts: Update tutorial post
- [ ] Update 'Currently'
- [ ] Landing page: posts, then publications, then projects. Can posts include images?



### Publications

1. Make a .bib file for any new publications

2. Use the academic CLI (installed from here https://github.com/GetRD/academic-file-converter) to convert any new bib files to publications e.g. `academic import weller_publications.bib hannahiweller/content/publication/ --compact`
3. For each new folder, go through and:
   - Change from publication_types: "article-journal" to publication_types: ["2"] (otherwise the site won't build)
   - Add a link to the PDF of the paper under `url_pdf: "static/media/pdf/"` e.g. `url_pdf: "static/media/pdf/Weller2020_catfish.pdf"`
   - Double check author names, formatting, etc.
   - Add an image called "featured.png"



### Formatting

Font: themes -> starter-academic -> data -> fonts -> my_font_set.toml

To update a font: Go to https://fonts.google.com/ and pick one -> get font -> get embed code -> copy the section of embed code between "family" and "&display=swap" and add it to google_fonts in the my_font_set.toml 

IMPORTANT: If you're building the site locally this won't show up until you stop the server and rebuild/serve the site. So after updating a font:

`blogdown::stop_server(); blogdown::build_site(); blogdown::serve_site()`

