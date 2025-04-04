---
title: "Publish org-roam notes to personal wiki using ox-hugo and Quartz"
draft: false
---

[quartz]({{< relref "2025-01-07-222940-quartz.md" >}})

[org roam]({{< relref "20230220220100-org_roam.md" >}})

[ox-hugo]({{< relref "20230304173453-ox_hugo.md" >}})


## [Zotero]({{< relref "20230517103056-zotero.md" >}}) {#zotero--20230517103056-zotero-dot-md}

Zotero is configured to automatically [export]({{< relref "20230517103056-zotero.md#export" >}}) a BibLaTeX bibliography, which is placed in ~/notes/references.bib. This file is read by org-cite and citar to resolve citation keys.

[Citar](https://github.com/emacs-citar/citar) provides auto-completion for entries from the bibliography. I use it together with [vertico](https://github.com/minad/vertico) and [marginalia](https://github.com/minad/marginalia) for my auto-completion setup. Additionally, citar provides functionality that integrates with org-mode and org-cite. I use the following customization options:

```emacs-lisp
;; All-the-icons (for pretty symbols in citar)
(use-package all-the-icons
  :straight t
  :if (display-graphic-p))

;; Org-cite and Citar configuration using straight.el
(use-package citar
  :straight t
  :after oc
  :custom
  (org-cite-global-bibliography '("~/org/library.bib"))
  (org-cite-insert-processor 'citar)
  (org-cite-follow-processor 'citar)
  (org-cite-activate-processor 'citar)
  (citar-bibliography '("~/org/library.bib"))
  (citar-notes-paths '("~/org/org-roam/references/"))
  (citar-symbols
   `((file ,(all-the-icons-faicon "file-pdf-o" :face 'all-the-icons-green :v-adjust -0.1) . " ")
     (note ,(all-the-icons-material "speaker_notes" :face 'all-the-icons-blue :v-adjust -0.3) . " ")
     (link ,(all-the-icons-octicon "link" :face 'all-the-icons-orange :v-adjust 0.01) . " ")))
  (citar-symbol-separator "  "))

```

This configures org-cite to rely on citar for inserting, viewing, and following citations. It also makes citar’s auto-completion prettier, using fancy icons.

org-cite is a minimal frontend for handling citations in Org files and is included with org-mode. It introduces **cite:** links, which are citations. By using the **#+print_bibliography:** instruction, a bibliography will be included when exporting from Org to other formats. Citations will also be formatted according to the configured citation style. Citar builds on org-cite and expands on the possible actions on citations, such as jumping to the corresponding literature notes in org-roam or opening the referenced content.

To actually take literature notes, I use the **citar-open** function and select the relevant bibliography entry. This will present me with multiple options, one of which is to open or create a note. Because I am using [citar-org-roam](https://github.com/emacs-citar/citar-org-roam) and have enabled the **citar-org-roam-mode** minor mode, this will create a new org-roam note (instead of an ordinary Org file). The new note will have the **ROAM_REFS** property populated with the citation key, allowing org-roam to connect cite: links with the literature notes, tracking backlinks as it does for other notes. See org-roam’s manual for more information about **ROAM_REFS**.

To revisit these notes later, I can use **citar-open** again or follow **cite:** links in other notes that reference the content. Both of these also allow me to open the file (such as a PDF) corresponding to a key. In theory, I should configure citar-library-paths which lets citar look up files from citation keys. However, the exported BibTeX file contains absolute paths to these files, which citar apparently can use.

Literature notes go in the ~/notes/references subdirectory. This behavior is controlled by the citar-org-roam-subdir variable, which defaults to "references".

I want to include BibTeX entries with my literature notes so that notes published on my website are self-contained. Unfortunately, citar-org-roam doesn’t allow customizing the capture template being used. Instead, I redefine citar-org-roam--create-capture-note. I also copy citar--insert-bibtex and modify it to return a string instead of inserting into a buffer.

```emacs-lisp
;; citar-org-roam only offers the citar-org-roam-note-title-template variable
;; for customizing the contents of a new note and no way to specify a custom
;; capture template. And the title template uses citar's own format, which means
;; we can't run arbitrary functions in it.
;;
;; Left with no other options, we override the
;; citar-org-roam--create-capture-note function and use our own template in it.
(defun dh/citar-org-roam--create-capture-note (citekey entry)
    "Open or create org-roam node for CITEKEY and ENTRY."
    ;; adapted from https://jethrokuan.github.io/org-roam-guide/#orgc48eb0d
    (let ((title (citar-format--entry
                  citar-org-roam-note-title-template entry)))
      (org-roam-capture-
       :templates
       '(("r" "reference" plain "%?" :if-new
          (file+head
           "%(concat
 (when citar-org-roam-subdir (concat citar-org-roam-subdir \"/\")) \"${citekey}.org\")"
           "#+title: ${title}\n\n#+begin_src bibtex\n%(dh/citar-get-bibtex citekey)\n#+end_src\n")
          :immediate-finish t
          :unnarrowed t))
       :info (list :citekey citekey)
       :node (org-roam-node-create :title title)
       :props '(:finalize find-file))
      (org-roam-ref-add (concat "@" citekey))))

;; citar has a function for inserting bibtex entries into a buffer, but none for
;; returning a string. We could insert into a temporary buffer, but that seems
;; silly. Plus, we'd have to deal with trailing newlines that the function
;; inserts. Instead, we do a little copying and implement our own function.
(defun dh/citar-get-bibtex (citekey)
    (let* ((bibtex-files
            (citar--bibliography-files))
           (entry
            (with-temp-buffer
              (bibtex-set-dialect)
              (dolist (bib-file bibtex-files)
                (insert-file-contents bib-file))
              (bibtex-search-entry citekey)
              (let ((beg (bibtex-beginning-of-entry))
                    (end (bibtex-end-of-entry)))
                (buffer-substring-no-properties beg end)))))
      entry))

(advice-add #'citar-org-roam--create-capture-note :override #'dh/citar-org-roam--create-capture-note)
```


## Reference List {#reference-list}

1.  <https://www.asterhu.com/post/20240220-publish-org-roam-with-quartz-oxhugo/>
2.  <https://honnef.co/articles/my-org-roam-workflows-for-taking-notes-and-writing-articles/>
