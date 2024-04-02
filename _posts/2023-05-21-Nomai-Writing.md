---
layout: post
title: Nomai Writing (nomai-writing.com)
---
![header spiral]({{ site.baseurl }}/images/nomai/pretty_spiral.svg "Example Nomai spiral")

## Background

[Outer Wilds](https://www.mobiusdigitalgames.com/outer-wilds.html) is my favorite game, a masterpiece of a meditation on the fragility of life and the vastness of space and the nature of friendship and solitude and exploring alien ruins and crashing spaceships and toasting marshmallows by the fire with your friends. I strongly recommend playing it, and I strongly recommend reading as little as possible about it before you do. Don't spoil yourself! This post contains no spoilers past the first ~10 minutes of the game. In the interest of brevity I mostly avoid gushing; feel free to infer "(which is awesome, great, beautiful, cleverly crafted)" parentheticals after every reference to the game.

In Outer Wilds, you explore a miniature solar system. One of your chief objectives is to discover what happened to the Nomai, a species of friendly scientifically minded aliens who inhabited the solar system long ago but died out for unknown reasons. You discover the Nomai's history primarily by discovering fragments of their writing. Nomai writing has a beautiful branching spiral pattern, like so:

![nomai wall]({{ site.baseurl }}/images/nomai/scroll_wall_noui.png "A wall showing a Nomai conversation")

Each spiral is a statement from one Nomai, and the branching spirals are the back-and-forth of Nomai conversation. So Nomai conversations have a tree structure rather than the linear structure typical of human conversation.

In-game, translation is handled by a device you carry: point at a sample of Nomai writing, see translation on the screen. So there's no need for the developers to implement a full Nomai writing system; the writing just has to look pretty. In fact, zoom in far enough and you'll see that all writing in the game is repetitions of this motif translated, scaled, and warped:

![writing pattern]({{ site.baseurl }}/images/nomai/writing_motif.png "The repeating pattern of Nomai writing")

Outer Wilds has resonated with me since I first played it: partly because it's just an excellent game, partly because the gameplay and setting align with my personal preferences (exploration, spaceflight, etc.), and partly because the game's expression of philosphical themes helped me work through grief over a terminal illness in my family. I can think of no other work of art—in any medium—that has affected me as much. Recently I asked my wife to play through it with me (she did the exploration, I flew the ship; OW isn't a twitchy game, but flight requires some dexterity), and watching her play inspired me to engage with the Outer Wilds world via a personal project.

Despite being fictional aliens, the Nomai are deeply human and offer much to admire. I quite enjoy the aesthetics of the Nomai writing, so I decided to build [Nomai Writing](https://nomai-writing.com), a tool which maps any text to a unique spiral of Nomai-style writing.

## nomai-writing.com
### Main functionality
Input a message, get back a SVG encoding that message as a Nomai-style spiral. For example, here's the spiral for `Outer Wilds is an excellent game that you should definitely play`.

![Spiral example]({{ site.baseurl }}/images/nomai/example.svg "A short spiral from nomai-writing.com")

### Features
1. Arbitrary Unicode is supported, meaning messages can be in various languages or combinations thereof, include emoji, etc.
2. Distinct messages generate unique spirals.[^1]
3. Configurable handwriting mode simulates varying levels of imprecision as if spirals are written by hand.
4. Partial support for configurable spiral length. Adjusting the encoding base (details below) affects the output spiral.
5. Messages of up to ~paragraph length are supported. [NomaiText.jl](https://github.com/evanfields/NomaiText.jl/tree/main) supports arbitrary length if you're patient. E.g. here's the Gettysburg Address ASCII encoded:
    ![Gettysburg]({{ site.baseurl }}/images/nomai/gettysburg.svg "Nomai Gettysburg Address")

## Technical details
tldr: A message is converted to a (very) large integer. We then create a two dimensional grid of "glyphs" (basically letters). The number-encoded message is used to decide (a) which glyphs to draw; (b) where to draw them; (c) how to connect the glyphs. Optionally the rectangular grid of glyphs is "typeset" by wrapping into a spiral. Read on for details...

### Oracles
Each Unicode character is associated with a codepoint integer, so a Unicode string can be converted to a list of integers. A list of integers is converted to a single integer by treating the list as digits of a (high) base number. For example, the characters in `"hey"` have codepoints `104, 101, 121` respectively. If we choose 256 as our base, we can represent `"hey"` as `104 * 256^0 + 101 * 256^1 + 121 * 256^2 = 7955816`.

Note that the base we choose should be larger than the largest codepoint we might see in a message; otherwise distinct messages can collide. If we encoded `"hey"` in base 10 as `104 * 10^0 + 101 * 10^1 + 121 * 10^2 = 13214` we'd have the same final result as encoding `"rxw"` which has codepoints `114, 120, 119` and `13214 = 114 + 120 * 10 + 119 * 100`. Therefore a base of 256 works for ASCII messages and ~200,000 for the Unicode you're likely to encounter.

The resulting integers encoding entire messages can be quite large! Using a base of 200,000, the integer encoding of `"This sentence has moderate length."` is over 10^176. For comparison, there are ~10^80 atoms in the universe.

Once a message has been converted to a big integer (Julia's `BigInt` type), we wrap the integer in an `Oracle` which can answer "which-of-`k`" questions. If an `Oracle` has current state `x` and is asked to pick one item from a set of `k` items, it picks the `(x mod k + 1)`th item and updates its state to `x ÷ k` since picking from `k` items "uses up" a factor of `k`. E.g. if an `Oracle` has state 37 and you ask it to pick between 5 items, it'll pick the 3rd (`1 + 37 mod 5 = 3`) and update its state to `37 ÷ 5 = 7`. When an `Oracle`'s state reaches 0, it is exhausted: all information contained therein has been used up to answer questions. `Oracle`s reset their state to its original value once exhausted (in case a few extra questions are needed) and internally track whether they've been exhausted.

So as long a we can phrase all our text-to-Nomai questions as "which-of-`k`", an `Oracle` wrapped around a message integer tells us how to use the information in a message to build a Nomai writing sample.

### Glyphs
A typical question to ask an `Oracle` is "which glyph should I draw next?" My Nomai writing system has an alphabet of 33 glyphs:

![Nomai alphabet]({{ site.baseurl }}/images/nomai/glyphs.svg "A short spiral from nomai-writing.com")

These are all hand-generated based on looking at the in-game writing samples. The first 16 glyphs are "base glpyhs" and the remaining 17 glyphs are base glyphs with an annotation (e.g. a small square, a pair of "horns", etc.). The base-vs-annotated distinction has no semantic meaning and is just vestigial from an early attempt at capturing the Nomai spirit.

Drawing a message as Nomai writing is basically a simple interative process:
```julia
function message_to_nomai(message; base = 200_000)
    o = Oracle(message; base)
    glyphs = Glyph[]
    while !(iscomplete(o))
        # oracle chooses a glyph
        push!(glyphs, ask!(o, KNOWN_GLYPHS))
    end
    return glyphs
end
```

### GlyphGrid
The above mini-algorithm returns a simple list of glyphs and skips important decisions: where glyphs are located relative to each other and how glyphs should connect. These are handled by the `GlyphGrid` type and associated functions. A `GlyphGrid` contains all the "abstract" information of a Nomai writing sample: which glyphs to draw, how the glyphs relate spatially, how the glyphs are connected. It doesn't include any typesetting details like how to put glyphs on a canvas, how big the glyphs should be, etc.

A `GlyphGrid` is a _grid_ because internally it contains a two dimensional grid of possible glyph locations. Each location can contain a glyph or nothing, and glyphs are connected to each other, like so for the string `"Obi-Wan Kenobi"` encoded in base 200,000:

![Glyph grid]({{ site.baseurl }}/images/nomai/glyphgrid.svg "Abstract grid of glyph locations and connections")

There are two "paths" of glyphs through the grid. The paths can overlap at a grid location, in which case only a single glyph is drawn there. Both paths always start at the middle row on the far left, and as an `Oracle` is transformed into a `GlyphGrid` the paths extend to the right one column at a time. The `Oracle` is used to pick the grid locations within each column that have glyphs.

Finally, glyphs along a path in the grid must connect to each other. When the abstract path through the grid indicates that two glyphs should connect, we connect the closest vertices of the two glyphs. If there are multiple pairs of vertices (one from each glyph) at equivalent distances, we let the `Oracle` pick the pair used.

### Typesetting spirals
A `GlyphGrid` can be drawn directly, which produces something like the linear writing occasionally seen in-game. Here's `"Obi-Wan Kenobi"` from above drawn that way:

![Nomai linear]({{ site.baseurl }}/images/nomai/obiwan.svg "Linear Obi-Wan")

Typesetting a spiral instead "just" requires mapping the horizontal axis of the `GlyphGrid` to a spiral; the vertical grid axis becomes perpendicular offsets to the spiral. The potential glyph locations around a spiral look like so; notice how the glyphs get bigger and farther apart as you move along the spiral:

![Spiral vis]({{ site.baseurl }}/images/nomai/spiral_vis.svg "Spiral typesetting")

`NomaiText.jl` does a few tweaks to make typeset spirals look prettier and more authentic:
* As is the case in-game, glyphs get bigger along the spiral. The first glyph in the densely twisted center of the spiral is half the size of the last glyph at the tail.
* Accordingly, glyphs are not evenly spaced along the spiral since the larger tail glyphs need more space than the small core glyphs.
* The spiral period (i.e. how many times the spiral twists) is chosen to approximately match the needed length.
* The spiral is rotated so that (except for very short spirals) the tail ends midway up the left side of the resulting image.

Here's Obi-Wan once more, now as a spiral:

![Obi-Wan spiral]({{ site.baseurl }}/images/nomai/obiwan_spiral.svg "Spiral Obi-Wan")

### Tools used
#### Julia
All of the logic described here was implemented in Julia, now wrapped up in [NomaiText.jl](https://github.com/evanfields/NomaiText.jl). I decided to write in Julia because it's my preferred language and I don't get to write it much at my day job, but it turned out to be a _great_ choice: expressive and succint enough to make iteration and refactoring easy, but performant enough to save my sanity. I've spent barely any time on performance optimization but tried to write in a reasonably idiomatic (and type stable) way. On my [aging] desktop, typesetting a few sentences takes a second or two: noticeable, but not so long as to be super painful to wait for. Having to wait a few _tens_ of seconds per message would have made developing in a slower language much less pleasant.

#### Luxor.jl
Luxor.jl is a Julia vector graphics package ("Cairo for tourists"), and this project would have been _much_ harder without it. Luxor handles all the graphics work of canvases and rendering things on screen and generating SVGs etc. Luxor _also_ provides a ton of out-of-the-box geometry tools. For example, the spirals used are straight from Luxor's `spiral` function; I just had to find some shape parameters that looked about right.

While Luxor was a huge enabler for NomaiText overall, at times it was also a significant stumbling block. Luxor enforces a fairly specific mental model of the world: complicated drawings are done by transforming a global drawing _state_ and then drawing locally near the origin. This works well for something like typesetting English characters: translate the global state so that the origin is at the bottom-left corner of the next character to be drawn, draw that character at the origin, repeat. But this paradigm became tricky in NomaiText, especially for spiral typesetting: adjacent glyphs in a `GlyphGrid` will correspond to different locations along the spiral which in turn means the glyphs have different transforms of the global coordinates (location, rotation, and scale). But glyph connections means we have to draw a line between two glyphs with different coordinate systems. This all ended up fine but took me a while to get working properly. Luxor author Cormullion was a huge help here—thanks!

#### Jot.jl and AWS Lambda
[nomai-writing.com](nomai-writing.com) isn't visited frequently—and mostly by me—so it's neither cost nor compute efficient to have a server always running. Instead, the frontend calls AWS's API Gateway which in turn hits a Lambda function that maps `message string => svg string`. Julia isn't a first class supported language on AWS Lambda, but fortunately the Jot.jl package takes care of almost all the complexity of wrapping your Julia code into a package-compiled binary, wrapping that into a Docker container that implements the Lambda spec, and pushing that container to your AWS account.

## Closing thoughts
Trying to reverse engineer the look of Nomai writing was a surprisingly difficult task; I went through a lot of prototypes that didn't feel anywhere near right. The current version is still far from perfect (and the [to-do list](https://github.com/evanfields/NomaiText.jl/blob/main/todo.md) remains long), but is at least at a point where I can comfortably share the results. Perhaps sometime years hence I'll be listening to the Outer Wilds soundtrack and the bug will bite me and I'll return to continue improving `NomaiWriting.jl`. In the meantime, pull requests are most welcome.

----

[^1]: Well, _almost_ guaranteed to be unique. Very specific circumstances you almost surely won't run into in practice can cause message "collision" where two distinct messages generate identical spirals. See the [to-do](https://github.com/evanfields/NomaiText.jl/blob/main/todo.md) for more details.
