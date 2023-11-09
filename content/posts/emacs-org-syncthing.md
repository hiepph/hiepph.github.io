---
title: "Emacs + Org-mode + Syncthing = Perfect combo"
date: 2017-11-24
---

Taking notes is a part of my life and work: jot down ideas, manage tasks and track personal projects.
But soon came a problem, what tool is best for supporting this simple task.

There are 3 must-have features I searched for:

+ Markdown (or relevant) editor

+ Auto-sync between machines

+ Less but not least, affordable price

## Searching journey

As for me, todo note or shopping list is too trivial. And editing markdown on smart phone sucks and inconvenient. So I just focus on PC and laptop station here.

I found [Boostnote](https://boostnote.io/) first. Free as it's a FOSS (Free and Open Source Software). But this fancy editor is (as expected) built on [Electron](https://electronjs.org/), a RAM-hungry framework. No built-in sync feature, but enabled through third party like Google Drive or Dropbox.

Since I didn't like the roundabout syncing, so [Inkdrop](https://www.inkdrop.info/) is my second stop. Also built on Electron, but came with built-in sync service through [CouchDB](https://couchdb.apache.org/), and charge you less than $5 per month. This was pretty suitable for me, and I felt comfortable in 2-month trial. The creator also wrote a [post](https://blog.inkdrop.info/how-i-built-a-markdown-editor-earning-1300-mo-profit-inkdrop-ddf6ad702c42) about process of building his product, which is very interesting to know about.

But I don't really like the idea of a RAM-killer app for just a simple task like saving texts. There should be another ways for rubbing my itch.

## The unicorn came to the rescue

Vim was my main editor, until I knew about [Org-mode](http://orgmode.org/), which is only fully available on Emacs.

![org-mode](https://upload.wikimedia.org/wikipedia/commons/thumb/a/a6/Org-mode-unicorn.svg/1200px-Org-mode-unicorn.svg.png)

Vim was good, but I'm somewhat not in the mood of learning Vimscript for long-term scaling my `.vimrc`. In contrast, I love Lisp - the good old legend survived through time.
So `Emacs Lisp` + `Org-mode` are 2-main reasons made me make a switch to a new life-long partner: Emacs.

So after grabbing the basics and setting a somewhat decent `.emacs.d`, fully integrating Org-mode, life is damn good.

![demo](/org/org.png)

```lisp
;; [...]
;; simple setup

(require-package 'org)

(add-hook 'org-mode-hook 'turn-on-font-lock)

(global-set-key "\C-cl" 'org-store-link)
(global-set-key "\C-ca" 'org-agenda)
(global-set-key "\C-cc" 'org-capture)
(global-set-key "\C-cb" 'org-iswitchb)

(setq org-log-done 'time)

;; Turn off auto-fold
(setq org-startup-folded nil)

;; [...]
```

My dotfiles's [.emacs.d](https://github.com/hiepph/dotfiles/tree/master/emacs/.emacs.d) for your information. And a good setup [guide](http://manenko.com/2016/08/03/setup-emacs-for-rust-development.html) for Emacs in my opinion.

> Org mode is for keeping notes, maintaining TODO lists, planning projects, and authoring documents with a fast and effective plain-text system.
> - [org-mode](http://orgmode.org/)

Talk is cheap, actually use this with powerful key combination really broads my horizon. For example:

+ Add a TODO list with `M-shift-RET`

![todo](/org/todo.png)

+ Mark as completed with a single combination `C-c C-t`.

![todo_complete](/org/todo_complete.png)

Besides, learning Org syntax is not so hard, from a perspective of a Markdown fan-boy like me.

## Syncthing - the lost ingredient to perfect combo

![combo](https://vignette.wikia.nocookie.net/streetfighter/images/1/1c/Kens_kick_combo.gif/revision/latest?cb=20100605074359)

So Emacs + Org-mode is totally free, check. Wonderful Markdown/Org editor, check. It occurred to me that I knew about [Syncthing](https://syncthing.net/) in a Golang blog post:

> Syncthing is an open-source cross platform peer-to-peer continuous file synchronization application.
> - [Eight years of Go](https://blog.golang.org/8years)

Basically, Syncthing decentralizes your shared data to all connected nodes identified with cryptographic certificates.

I need at least a live node for no-downtime syncing (a central hub). A low cost private cloud server is a good choice, but in my case I used my 99.99% uptime Raspberry Pi 3 (which mainly ran [Pi-hole](https://pi-hole.net/) for my home).

```sh
# Add GPG key
wget -O - https://syncthing.net/release-key.txt | sudo apt-key add -

# Add repository
echo "deb http://apt.syncthing.net/ syncthing release" | sudo tee -a /etc/apt/sources.list.d/syncthing-release.list

# Update and install
sudo apt-get update
sudo apt-get install syncthing -y
```

Re-bind configuration from local loopback `127.0.0.1` to `0.0.0.0` (`/home/pi/.config/syncthing/config.xml`):

```xml
<gui enabled="true" tls="true">
 <address>0.0.0.0:8384</address>
 <apikey>...</apikey>
</gui>
```

This will enable you to access `http://<pi's IP>:8384` Web Interface of Syncthing.

Finally just start `syncthing` service:

```sh
sudo systemctl start syncthing@pi.service

# auto start at start-up
sudo systemctl enable syncthing@pi.service
```

Access `http://<pi's IP>:8384`:

![syncthing](/org/syncthing.png)

Here you can see I shared my `/home/pi/org` directory which contains all my `.org` notes to 2 nodes: online `dell` + offline `thinkpad`.

Now if I make a change from my `/home/dell/org`, it will syncs back to `/home/pi/org`. And when my `thinkpad` goes online, `/home/thinkpad/org` auto updates all its notes, too!

Et Voilà! Work like a charm. Plus, both Emacs and Syncthing use just a little of RAM and CPU resource.


## Conclusion

[Inkdrop](https://www.inkdrop.info/) solo, (Vim + Git) traditional way, or (Emacs + Org-mode + Syncthing) combo with hands-free syncing? Use the right tool for the job, and pick what suits you the best.

For now, Emacs is my everyday editor, including programming stuff and note-taking. Long live (Emacs) Lisp.
