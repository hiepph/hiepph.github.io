---
title: "use-package for a tidy .emacs.d"
date: 2018-11-07
---

[//]: # (problem)

My `~/.emacs.d` configuration used to be a mess.

For example, here are 2 old configurations for [IDO](https://www.emacswiki.org/emacs/InteractivelyDoThings) and sidebar [neotree](https://github.com/jaypei/emacs-neotree).

```lisp
;; ### IDO #####
(require 'ido)

;; enable IDO
(ido-mode 1)
(ido-everywhere 1)
(ido-ubiquitous-mode 1)

(setq ido-enable-flex-matching t)
(setq ido-use-filename-at-point nil)
(setq ido-auto-merge-work-directories-length -1)
(setq ido-use-virtual-buffers t)

;; Shows a list of buffers
(global-set-key (kbd "C-x C-b") 'ibuffer)
```

```lisp
;; #### Neo Tree #####
(require-package 'neotree)

;; Bind F8 to show the tree
(global-set-key [f8] 'neotree-toggle)

;; slow rendering
(setq inhibit-compacting-font-caches t)

;; set icons theme
(setq neo-theme (if (display-graphic-p) 'icons 'arrow))

;; Every time when the neotree window is opened, let it find current file and jump to node
(setq neo-smart-open t)

;; When running ‘projectile-switch-project’ (C-c p p), ‘neotree’ will change root automatically
(setq projectile-switch-project-action 'neotree-projectile-action)

;; show hidden files
(setq-default neo-show-hidden-files t)
```

Both are identical for some configurations:

+ Install a package, typically from [MELPA](https://melpa.org/packages/) or [Marmalade](https://www.emacswiki.org/emacs/MarmaladeRepo).
And you have to do it *manually* by default. I used a little trick for auto installation if the required package is not yet installed.

```lisp
(require 'package)

(defun require-package (package)
  "Install given PACKAGE if it was not installed before."
  (if (package-installed-p package)
      t
    (progn
      (unless (assoc package package-archive-contents)
	(package-refresh-contents))
      (package-install package))))
```

Replace `(require 'some-package)` with `(require-package 'some-package)` does the trick.


+ Require package:

| IDO  |   neotree  |
| ------- | ------ |
| `(require 'ido)` | `(require-package 'neotree)` |

+ Enable mode: (IDO) `(ido-mode 1)`

+ Setup:

| IDO  |   neotree  |
| ------- | ------ |
| `(setq ido-enable-flex-matching t)` | `(setq inhibit-compacting-font-caches t)` |

+ Bind keys:

| IDO  |   neotree  |
| ------- | ------ |
| `(global-set-key (kbd "C-x C-b") 'ibuffer)` | `(global-set-key [f8] 'neotree-toggle)`


They all have the same indent level and repeat many times, thus trouble for productivity and readability. I want a more compact way of tidying my configuration.


[//]: # (solve with use-package)

## Enter [`use-package`](https://github.com/jwiegley/use-package)

It simply combines all those processes above into a single block of code.

Setup is simple:

```lisp
;; Basic setup for package archives
(require 'package)

(setq package-enable-at-startup nil)
(add-to-list 'package-archives '("melpa" . "http://melpa.org/packages/"))
(add-to-list 'package-archives '("marmalade" . "http://marmalade-repo.org/packages/"))
(add-to-list 'package-archives '("gnu" . "http://elpa.gnu.org/packages/"))
(package-initialize)

;; use-package setup
;; install if not yet installed
(unless (package-installed-p 'use-package)
  (package-refresh-contents)
  (package-install 'use-package))

(eval-when-compile
  (require 'use-package))

(use-package diminish
  :ensure t
  :defer t)
(use-package bind-key
  :ensure t
  :defer t)
```

And now the best part of `use-package` really shines. Enter my new setup for `IDO` and `neotree`.

```lisp
;; IDO
(use-package ido
  :diminish ido-mode
  :bind ("C-x C-b" . 'ibuffer)   ;; Shows a list of buffers
  :init
  (ido-everywhere 1)
  (ido-ubiquitous-mode 1)
  (setq ido-enable-flex-matching t)
  (setq ido-use-filename-at-point nil)
  (setq ido-auto-merge-work-directories-length -1)
  (setq ido-use-virtual-buffers t))
```

```lisp
(use-package neotree
  :ensure t
  :bind ("<f8>" . 'neotree-toggle)
  :init
  ;; slow rendering
  (setq inhibit-compacting-font-caches t)

  ;; set icons theme
  (setq neo-theme (if (display-graphic-p) 'icons 'arrow))

  ;; Every time when the neotree window is opened, let it find current file and jump to node
  (setq neo-smart-open t)

  ;; When running ‘projectile-switch-project’ (C-c p p), ‘neotree’ will change root automatically
  (setq projectile-switch-project-action 'neotree-projectile-action)

  ;; show hidden files
  (setq-default neo-show-hidden-files t))
```

Neat! As you see, each package comes with its own territory.
It automatically installs and requires the package (`:ensure`), enable the mode (`:diminish`) and then loads the configuration (`:init`).

Such task as adding packages, add new features (`:init` or `:config`), bind keys (`:bind`), etc. now becomes very easy and productive.

[//]: # (future and conclusion)

You can check out more cool features of `use-package` on its [README](https://github.com/jwiegley/use-package).
