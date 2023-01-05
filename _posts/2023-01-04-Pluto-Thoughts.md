---
layout: post
title: Thoughts on Pluto.jl
---
This post is about [Pluto.jl](https://plutojl.org/), a reactive notebook environment for the [Julia](https://julialang.org) programming language. Spoilers: I recommend it!

For years, I was stridently anti-notebook (as in Jupyter, not the bundle of papers you write in). [Here's a fun talk](https://youtu.be/7jiPeIFXb6U)—not me—laying out some problems with notebooks. Then I watched the [JuliaCon talk inroducing Pluto.jl](https://youtu.be/IAF8DjrQSSk?t=112) and thought gee, that seems to solve almost all my complaints. Maybe I like notebooks after all? But I never spent much time with Pluto until recently, when I used it for a few days for a small self-contained project at work. Here are my thoughts a few days in; your mileage may vary.

# Good

### Reactivity

My biggest gripe with traditional Jupyter notebooks is that they're _rife_ with hidden state. Cells can be executed in arbitrary order, variables can be defined and overwritten elsewhere, inconsistency between current state and "state that generated some output I'm looking at" is common, etc. In contrast, Pluto is reactive, which as a non-expert user I take to mean:
* Each variable can only be defined in one cell, e.g. you can't have multiple cells defining `x` (in global scope; of course multiple cells can use the same variable name in separate local scopes).
* Pluto knows which cells depend on which other cells. If cell 1 defines `x = 5` and cell 2 defines `y = x + 42`, then Pluto knows cell 2 depends on cell 1. This dependency is not based on the position of cells in the notebook.
* Whenever you update a cell, all dependent cells are recomputed, and their dependents are recomputed, etc.

Therefore
* The entire notebook always has a consistent state. (Except while cells are evaluating)
* Any change you make will propogate throughout the notebook. This enables reactive use cases, e.g. modifying a variable and watching a plot update.
* Cell ordering is irrelevant, so you can always reorganize safely.

These are incredible features! When exploring data or algorithms, it's super helpful to make a change and see how some results or plots update. Fast feedback loops enable rapid iteration. This is especially powerful with Pluto's interactive UI ([example](https://user-images.githubusercontent.com/6933510/134824521-7cefa38a-7102-4767-bee4-777caf30ba47.mp4)), which brings us to:

### Interactive UI

Pluto comes with [PlutoUI](https://juliapluto.github.io/sample-notebook-previews/PlutoUI.jl.html), a package providing easy to use UI tools like sliders and drop-down menus. Tweak a parameter by sliding a slider, see a plot update nearly instantly, very slick. The core UI tools provided out of the box are also delightful straightforward to use, e.g. if you want to have a slider controling some variable `alpha` you can do `@bind alpha Slider(0:100)`.

### Grab bag

* Pluto has good...vibes? The colors are pleasing, the descriptions are upbeat, the whole thing puts me in a nice mood.
* The Live Docs window updates really quickly as you type and is especially helpful for reminding you of how to call functions. For example, I wanted to construct a range using the `range` function but didn't remember the exact syntax. When I had typed `xs = range(` the live docs were showing me the docstring for `range`, which helpfully starts by listing the methods available. Super convenient!
* Pluto makes me appreciate Julia's speed, because even though compilation can at times be slow, being ultimately very fast makes reactivity much more pleasant.
* Apparently Pluto notebooks are git friendly and can be used as ordinary Julia `.jl` files. And ordinary `.jl`s can be notebooks? These features should fix several of my Jupyter complaints, but I haven't tried them out.


# Bad

### User interface
* Creating and deleting cells requires a fair amount of clicks. Deleting in particular requires clicking a small `...` button and clicking again to delete. Keyboard shortcuts are non-obvious (see below).
* No way to move multiple cells at once.
* No way to enable/disable multiple cells at once.
* Pluto is in light mode on my personal computer and dark mode on my work computer and I don't know why. I prefer light mode but don't know how to switch it.
* While writing this blog post I discovered you can click and drag to select multiple cells, but half the time I tried this I ended up accidentally copying a bunch of text into a cell rather than selecting multiple cells.

### Missing or non-obvious documentation
* When I started this project with Pluto, I carefully made a normal Julia environment with Pluto as well as the packages I anticipated needing: Gadfly, DataFrames, Distributions, etc. It took me a day to realize Pluto could affect my environment and _another_ day to realize the environment being modified wasn't the environment that launched Pluto but rather notebook-specific. This is a cool feature but not super obvious. In a regular Julia REPL, package commands are prefaced with the current environment and it's clear what environment you're modifying: `(@v1.8) pkg>`
    * I guess `Pkg.` commands can be used for more precise control over a notebook's environment, and use of such commands disables the default notebook-local environment, but that's pretty hidden.
    * Is there a way to use the package REPL mode with the automatic notebook environment? `]st` in a cell doesn't work. But won't use any `Pkg.` commands disable the automatic notebook environment?
    * Is it possible to use my own packages (not in the general registry) in the automatic environment?
    * For the curious reader: as of this writing, the docs on Pluto and package management are [here](https://github.com/fonsp/Pluto.jl/wiki/%F0%9F%8E%81-Package-management)
* There are limited keyboard shortcuts, but you have to look hard to find them! I got to a list by clicking the FAQ link in the bottom left of the notebook => UI => keyboard shortcuts. That list is itself confusing because it seems to list not-yet-finalized or working shortcuts? I just learned while writing I could have also hit `F1` to get shortcuts: cool, not obvious.
* It's not obvious to me how I should combine reactivity with file IO. For example, what if I want to interactively+reactively build a plot, then save it as an image once I'm satisfied with the result? If I have a cell which saves the plot, I'll end up saving a new plot every time I interactively change something. Is the intended workflow to only create such a cell once the plot is set, then delete it? Or to wrap the saving in an `if` block and use a PlutoUI element to control the condition on the if block? I looked for this in the docs but didn't find anything.
* What's the difference between "disable cell" and "disable in file"?
* Julia is a powerful and expressive language, so I assume it's possible to "outsmart" Pluto's analysis of which cells depend on which other cells. I didn't find any guidelines on particular kinds of dependencies between cells that aren't supported by the reactive model. This wasn't an issue for me, everything I tried to do worked fine.

### Miscellaneous
* My work project used Gadfly to create plots to help with interactive algorithm development. Overall this worked well, but:
    * Showing a Gadfly plot from a regular Julia REPL opens a browser window with a fully interactive plot. Not all interactive features worked in the Pluto rendering of the plot.
    * I wanted to make the plot bigger, but it wasn't clear how to do that without zooming the entire notebook webpage.
* The requirement that each cell can only define one variable isn't obvious, especially because there seems to be no downside to defining multiple variables within a cell using a `begin...end` block? Many times I tried to define multiple variables in a cell and had to wrap in a `begin...end` block.
* At one point I relaunched my notebook which used several heavy packages (Gadfly, DataFrames). The `using Gadfly, DataFrames` cell took a _long_ time to execute and the log icon didn't show anything, unlike in the [docs](https://github.com/fonsp/Pluto.jl/wiki/%F0%9F%8E%81-Package-management#logs).
* PlutoUI is convenient and easy to use, but its defaults seem not great for building intuitive interactive notebooks. For example, if you define `x` to be controlled by a slider via `@bind x Slider(1:10)`, the cell output is _just_ the interactive slider and it's not clear from the cell output what the slider actually controls! You can look at the code itself, which isn't very pretty. I ended up having my slider cells return a tuple like `("X", @bind x Slider(1:10))` and then hiding the cell source. This worked okay, though once you drag the slider the tuple unhelpfully expands into a multi-line representation.
    * The PlutoUI example notebook does have a pretty example of multiple sliders with labels in the "Combine" section. But, it's pretty high friction: mixing markdown and code, anonymous functions, you get back a named tuple instead of directly defining variables in global scope, etc. None of those are that tricky to deal with, but combined it was enough friction that I thought "meh, I'll just stick with my tuple hack."


# Overall

I'm pretty pleased with my Pluto experience and encourage folks to try it out. Thank you to the Pluto team for making a great tool!
The set of projects/situations where Pluto shines seems pretty narrow, roughly the intersection of:
* Exploratory work where reactivity is helpful, or where you need notebook features like mixed cell types.
* Not so compute-heavy that accidentally kicking off a chain of dependent cell evaluations is painfully slow.
* Not a tiny case where maximally fast start-up time matters (e.g. using Julia as a calculator).
* Small to moderate amounts of code needed; at a certain code volume, missing out on the ergonomics of your preferred editing setup is painful. Probably one strategy here is to write most of your code as a package in your editor of choice, then use Pluto as a user interface.

But that intersection is _far_ from non-empty, and next time I find myself therein, I think I'll again reach for Pluto.