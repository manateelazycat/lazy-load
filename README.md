With the increasing number of plugins for Emacs, Emacs will get more and more slow.

Emacs startup slow have two main reasons:
1. The threshold for garbage collection at load time is too small, which causes garbage collection to be triggered when the plugin is loaded, slowing down the startup speed.
2. By default, Emacs loads too many plugins, which wastes too much time to load the plugin when it starts.

The solution to the first problem is as follows:

```elisp
(let (;; temporarily increase `gc-cons-threshold' when loading to speed up startup.
      (gc-cons-threshold most-positive-fixnum)
      ;; Empty to avoid analyzing files when loading remote files.
      (file-name-handler-alist nil))

    ;; Emacs configuration file content is written below.

)
```

For the purpose of the above code, temporarily set the value of gc-cons-threshold to the maximum before Emacs loads any plugins, to avoid triggering garbage collection when Emacs starts.
After the Emacs configuration file is loaded, the variable gc-cons-threshold is automatically restored to the default value, avoiding Emacs taking up too much memory at runtime.

As for the solution to the second problem, I will use the lazy-load.el plugin I wrote before.

During the two years from 2007 to 2008, I almost played all the Emacs plug-ins at the time. When you load hundreds of plug-ins all by default, Emacs startup time can be extended from seconds to minutes.
At that time, I was thinking about how to optimize the startup time of Emacs to the second, without reducing the use of any plug-ins. Finally, I developed the lazy-load.el plug-in. Even if I use the 300+ plug-in, the Emacs startup speed is still in one second.

### Principle

The principle of lazy-load.el is simple:
1. Let's define our keymap first, and write the function corresponding to each key.
2. Tell the Emacs function the filename of the plugin so that when the function is called, Emacs knows where to load the plugin.
3. Emacs only loads very few plugins by default. After the loading is complete, Emacs will dynamically load the plugin and corresponding functions when we first press some key.

Because 99% of Emacs plugins can be loaded in 1s when loaded separately, so 99% of the plugins are loaded at runtime.
This greatly reduces the number of plug-ins that Emacs needs to load at startup, which ultimately improves Emacs startup time.


### installation method
1. Download lazy-load.el from [lazy-load](https://github.com/manateelazycat/lazy-load) to the ~/elisp directory
2. Add the following configuration to ~/.emacs

```
(add-to-list 'load-path (expand-file-name "~/elisp"))
(require 'lazy-load)
```

### Instructions
The following code means that the first time you press Alt + g, Emacs goes to the load-path directory and finds the goto-line-preview.el file, loads the plugin and executes the goto-line-preview function.

```elisp
(lazy-load-global-keys
 '(("M-g" . goto-line-preview))
 "goto-line-preview")
```

The following code means that the first time you press Ctrl + c t in ruby ​​mode, Emacs goes to the load-path directory and finds the ruby-extension.el file, loads the plugin and executes ruby-hash-syntax-toggle function.

```elisp
(lazy-load-local-keys
 '(("C-c t" . ruby-hash-syntax-toggle))
 Ruby-mode-map
 "ruby-extension")
```

Many global keystroke are already occupied by Emacs by default. You must unload them before you can re-bind these global keystroke, such as Ctrl + x. The following code is to uninstall the default global keystroke with lazy-load-unset-keys:

```elisp
(lazy-load-unset-keys '("C-x C-f" "C-z" "C-q" "s-W" "s-z" "M-h" "C-x C-c" "C-\\" "s-c" "s-x" "s-v"))
```

### Advanced usage
Sometimes, we will use a prefix keystroke to take different functions in one plugin. For example, the different functions of my sdcv.el plugin can use the Ctrl + z keystroke as the prefix key in the following code.
Press Ctrl + z and press p to trigger the sdcv-search-pointer function.

```elisp
(lazy-load-global-keys
 '(("p" . sdcv-search-pointer)
   ("y" . sdcv-search-pointer+)
   ("i" . sdcv-search-input)
   (";" . sdcv-search-input+))
 "init-sdcv"
 "C-z")
 ```

The corresponding lazy-load-local-keys also supports the last parameter passing the prefix keystroke, but lazy-load-local-keys use local keymap, not global keymap.

If Emacs loads a plugin by default, instead of dynamically loading it at runtime, you can also use the lazy-load-set-keys function to do a separate key binding operation, without having to manually write a line of repeated writes (define- Key keymap key) configuration

```elisp
(lazy-load-set-keys
 '(("M-;" . comment-dwim-with-haskell-style))
 Haskell-mode-map)
```

### Example
There are a lot of [examples](https://github.com/manateelazycat/lazycat-emacs/blob/master/site-lisp/config/init-key.el) of lazy-load usage
