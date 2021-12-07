# advent2021

Each days challenge is done in an Org-mode file, in Clojure, using Babashka.

## To run
Include the following elisp code in your Emac's ini 

```
(require 'ob)

(defvar org-babel-tangle-lang-exts)
(add-to-list 'org-babel-tangle-lang-exts '("babashka" . "clj"))

(defvar org-babel-babashka-command (executable-find "bb")
  "The command to use to compile and run your babashka code.")

;; Org-babel Babashka (clojure)
(defvar org-babel-default-header-args:babashka '())
(defvar org-babel-header-args:babashka '((package . :any)))

(defun ob-babashka-escape-quotes (str-val)
  "Escape quotes for STR-VAL so that Lumo can understand."
  (replace-regexp-in-string "\"" "\\\"" str-val 'FIXEDCASE 'LITERAL))

(defun org-babel-expand-body:babashka (body params)
  "Expand BODY according to PARAMS, return the expanded body."
  (let* ((vars (org-babel--get-vars params))
         (result-params (cdr (assq :result-params params)))
         (print-level nil) (print-length nil)
         (body (ob-babashka-escape-quotes
                (org-trim
                 (if (null vars)
                     (org-trim body)
                   (concat "(let ["
                           (mapconcat
                            (lambda (var)
                              (format "%S (quote %S)" (car var) (cdr var)))
                            vars "\n      ")
                           "]\n" body ")"))))))
    (if (or (member "code" result-params)
            (member "pp" result-params))
        (format "(print (do %s))" body)
      body)))

(defun org-babel-execute:babashka (body params)
  "Execute a block of babashka code in BODY with Babel using PARAMS."
  (let ((expanded (org-babel-expand-body:babashka body params))
        result)
    (setq result
          (org-babel-trim
           (shell-command-to-string
            (concat org-babel-babashka-command " -e \"" expanded "\""))))
    (org-babel-result-cond (cdr (assoc :result-params params))
      result
      (condition-case nil (org-babel-script-escape result)
        (error result)))))

(define-derived-mode babashka-mode clojure-mode "Babashka"
  "Major mode for editing Babashka code.")

(provide 'ob-babashka)
```
