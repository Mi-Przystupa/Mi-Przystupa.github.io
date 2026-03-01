---
title: "latexdiff: The Good, The Bad, The Ugly" 
date: 2026-03-01 
tags: ["LaTeX","latexdiff","thesis","Overleaf"]
author: ["Michael Przystupa"]
description: "An article summarizing my chat with Claude.AI on running latexdiff on my dissertation"
summary: "Latexdiff is a useful tool for creating a compiled difference between two versions of a latex file. The difficulty is the tool can be a bit trickier for fancier set-ups with nested file structure."
showToc: false
disableAnchoredHeadings: false

---

This article is a summary of my experience attempting to run `latexdiff` between versions of my PhD dissertation. I want to keep it short and simple, so for some background for those wanting to know more about relevant topics:

- [What is latexdiff](https://ctan.org/pkg/latexdiff)
- [Overleaf: Using latexdiff](https://www.overleaf.com/learn/latex/Articles/Using_Latexdiff_For_Marking_Changes_To_Tex_Documents)

## Disclaimers & Notes:


- This document is a summary of working with Claude.AI to run latexdiff on my dissertation to better facilitate the reviewing process. 
- This chat started with first downloading 2 copies of the document from Overleaf: the current version, and an older version I could get from the history. I use premium, but something like version control with git probably gives same result (you want 2 versions of the project).
- I read some where the file and folder structures should be identifical for nested use cases, so keep that in mind.
- This article was written based on my experience using MiKTeX with Ubuntu 24.04 for latex package management.  
- The commands are assuming you compile your latex in bash

## The Good

If you have a single LaTeX file you want to compare against an older version, it's straightforward:

```bash
latexdiff old_version.tex new_version.tex > diff.tex
```

This creates a LaTeX file you can compile as normal. The neat thing is you can further manually modify `diff.tex` because it's just another `.tex` file — this becomes useful for the ugly things later.

To compile, you'll want the full cycle to resolve citations and references:

```bash
pdflatex --shell-escape diff.tex
bibtex diff
pdflatex --shell-escape diff.tex
pdflatex --shell-escape diff.tex
```

The `--shell-escape` flag may be needed if your document uses packages like `auto-pst-pdf`.

## The Bad

### Multi-file projects need flattening

If your project uses `\input{}` or `\include{}` to split content across multiple files (as most theses do), you need to flatten everything into a single file first. `latexdiff` has a `--flatten` flag for this:

```bash
latexdiff --flatten old/main.tex new/main.tex > diff.tex
```

However, I found `--flatten` to be unreliable, especially with deeply nested includes. A more robust approach is to use `latexpand` to pre-flatten both versions:

```bash
# Flatten old version
cd /path/to/old_version
latexpand main.tex > ../old_flat.tex

# Flatten new version
cd /path/to/new_version
latexpand main.tex > ../new_flat.tex

# Diff the flattened files
cd ..
latexdiff old_flat.tex new_flat.tex > diff.tex
```

**Important:** compile `diff.tex` from your new version's project directory so that image paths, the `.cls` file, and `.bib` file all resolve correctly:

```bash
cp diff.tex /path/to/new_version/
cd /path/to/new_version/
pdflatex --shell-escape diff.tex
bibtex diff
pdflatex --shell-escape diff.tex
pdflatex --shell-escape diff.tex
```

## The Ugly

### Custom commands are invisible to flattening

Here's where things became messier to run the command. My dissertation template ([uAlberta Thesis Template](https://www.overleaf.com/latex/templates/university-of-alberta-latex-thesis-template/wmstjbzthypf)) defined custom include commands like:

```latex
\newcommand{\insertchapter}[1]{
    \input{"./Chapters/#1.tex"}
}
```

Neither `latexdiff --flatten` nor `latexpand` recognized `\insertchapter`. The result? A "flattened" file that was only 344 lines instead of thousands — all the chapter content was silently skipped.

**The fix:** temporarily replace custom include commands with standard `\input{}` in both versions before flattening. For example, change `\insertchapter{Introduction}` to `\input{./Chapters/Introduction.tex}`. Alternatively, do not use custom commands in your latex document. 

### Custom text commands need explicit flags

My template also used custom commands to store text content:

```latex
\newcommand\abstracttext[1]{\renewcommand\@abstracttext{#1}}
\newcommand{\addterm}[2]{\newglossaryentry{#1}{name={#1},description={#2}}}
```

By default, `latexdiff` doesn't know to diff inside these commands. For the template I was using, these included some of the following additional commands:

```bash
latexdiff \
  --append-textcmd=abstracttext,preface,dedication,acknowledgementtext,thesisquote,addterm \
  --append-safecmd=convocationdate,specialization,department,faculty,degree \
  old_flat.tex new_flat.tex > diff.tex
```

- `--append-textcmd` tells `latexdiff` to diff the text *inside* these commands.
- `--append-safecmd` handles commands whose arguments are short labels that can be diffed as units.

__Note__: I"m not convinced they made much of a difference, but it was a suggestion when working with Claude.AI that it should work. 

### Multi-line command arguments still don't diff

Even with `--append-textcmd`, I found that `latexdiff` struggled with multi-line arguments like a full abstract passed to `\abstracttext{...}` in my template. It would comment out the old version and insert the new one, but without any visual markup — so changes were invisible in the compiled PDF.

For these cases, the only reliable fix is manual editing of `diff.tex`: find the relevant section and wrap the old/new text in `\DIFdel{}`/`\DIFadd{}` yourself. Since `diff.tex` is just a `.tex` file, this is tedious but straightforward.

## My Final Workflow

For anyone using a template with custom commands, here's the full pipeline that eventually worked:

```bash
# 1. In both versions, replace custom commands (e.g., \insertchapter)  with more basic ones (e.g., \input or \include)

# 2. Flatten
cd /path/to/old_version
latexpand main.tex > ../old_flat.tex

cd /path/to/new_version
latexpand main.tex > ../new_flat.tex

# 3. Diff with custom command flags
cd ..
latexdiff \
  --append-textcmd=abstracttext,preface,dedication,acknowledgementtext,thesisquote,addterm \
  --append-safecmd=convocationdate,specialization,department,faculty,degree \
  old_flat.tex new_flat.tex > diff.tex

# 4. Copy into new version's directory and compile
cp diff.tex /path/to/new_version/
cd /path/to/new_version/
pdflatex --shell-escape diff.tex
bibtex diff
pdflatex --shell-escape diff.tex
pdflatex --shell-escape diff.tex

# 5. Manually fix any remaining sections in diff.tex (abstract, glossary, etc.)
```

## Takeaways

`latexdiff` works well for simple, single-file LaTeX projects. But if you're working with a multi-file thesis template full of custom commands — which is exactly the use case where you most need tracked changes — expect to spend some time wrestling with it. The tool assumes a level of project simplicity that most real dissertations don't have.

That said, once you understand the failure modes, the workarounds are manageable. The key insight is that `diff.tex` is just a regular `.tex` file, so anything the tool gets wrong can be fixed by hand.