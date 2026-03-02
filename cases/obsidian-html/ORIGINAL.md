Obsidian to HTML converter
This is a short Python script to convert an Obsidian vault into a vault of HTML files, with the goal of publishing them as static files. It is heavily dependent on the excellent markdown2 by trentm, but also deals with some parsing and file handling that makes it compatible with Obsidian's flavor of Markdown.

Installation
Install obsidian-html by running:

sudo pip install git+https://github.com/kmaasrud/obsidian-html.git
Or doing the same (without the sudo) as an administrator on Windows.

Admin privileges is needed to ensure that the script is in the PATH. You can easily clone this repo and install the package locally with pip install . or python setup.py develop, if you do not want to install as admin.

Usage
obsidian-html will by default convert all the Markdown documents in the folder you're running it in, and place the HTML files in a directory called html. You might not want to run it directly in your vault or place the converted files in another directory. This is specified by this syntax:

obsidian-html <path to vault> -o <path to html files>
The script will only convert the files located directly in the directory specified and never work recursively. To specify subfolders, these must be supplied to the -d flag, like in this example:

obsidian-html <vault> -d "Daily notes" "Zettels"
Templates
The output is not very exiting from the get-go. It needs some style and structure. This is done by using a HTML template. A template must have the formatters {title} and {content} present. Their value should be obvious. The template file is supplied to obsidian-html by the -t flag, like this:

obsidian-html <vault> -t template.html
Here you can add metadata, link to CSS-files and add unified headers/footers to all the pages. Here's an example of how I use the template function on my own hosted vault.

TeX support via KaTeX
By loading KaTeX in the HTML template and initializing it with $ and $$ as delimiters, you will have TeX support on the exported documents.

Add this to the bottom of you template's body
Syntax highlighting of code blocks
Using highlight.js, syntax highlighting is easily achieved.

Just add this to the bottom of you template's body
Deploying vault with GitHub Actions
Make a GitHub Actions workflow using the YAML below, and your vault will be published to GitHub Pages every time you push to the repository.

Make sure you have GitHub Pages set up in the vault, and that it has gh-pages /root as its source.

Modify the following YAML job to match your repository.

name: Deploy to GitHub Pages

on:
  push:
    branches: [ master ]
  
jobs:
  deploy:
    runs-on: ubuntu-latest

    steps:
    - uses: actions/checkout@v2

    - name: Set up Python 3.8
      uses: actions/setup-python@v2
      with:
        python-version: 3.8

  - name: Install obsidian-html
    run: |
      python -m pip install --upgrade pip
      pip install git+https://github.com/kmaasrud/obsidian-html.git
      
  - name: Generate HTML through obsidian-html
    run: obsidian-html ./vault -o ./out -t ./template.html -d daily

  - name: Deploy
    uses: s0/git-publish-subdir-action@develop
    env:
      REPO: self
      BRANCH: gh-pages
      FOLDER: out
      GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
To do
 Support local attachments
 Support the ![[]] embedding syntax (perhaps using iframe or some similar method)
 Support extra features added by the user through YAML metadata
Known issues
Links in headers lead to weird header ids, and thus malfunctioning header links from other pages.
