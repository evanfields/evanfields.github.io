---
layout: post
title: Nomai Writing (nomai-writing.com)
---

# Background

[Outer Wilds](https://www.mobiusdigitalgames.com/outer-wilds.html) is my favorite game, a masterpiece of a meditation on the fragility of life and the vastness of space and the nature of friendship and solitude and exploring alien ruins and crashing spaceships and toasting marshmallows by the fire with your friends. I strongly recommend playing it, and I strongly recommend reading as little as possible about it before you do. Don't spoil yourself! This post contains no spoilers past the first ~10 minutes of the game. In the interest of brevity I mostly avoid gushing; feel free to infer "(which is awesome, great, beautiful, cleverly crafted)" parentheticals after every reference to the game.

In Outer Wilds, you explore a miniature solar system. One of your chief objectives is to discover what happened to the Nomai, a species of friendly scientifically minded aliens who inhabited the solar system long ago but died out for unknown reasons. You discover the Nomai's history primarily by discovering fragments of their writing. Nomai writing has a beautiful branching spiral pattern, like so:

![nomai wall]({{ site.baseurl }}/images/nomai/scroll_wall_noui.png "A wall showing a Nomai conversation")

Each spiral is a statement from one Nomai, and the branching spirals are the back-and-forth of Nomai conversation. So Nomai conversations have a tree structure rather than the linear structure typical of human conversation.

In-game, translation is handled by a device you carry: point at a sample of Nomai writing, see translation on the screen. So there's no need for the developers to implement a full Nomai writing system; the writing just has to look pretty. In fact, zoom in far enough and you'll see that all writing in the game is repetitions of this motif translated, scaled, and warped:

![writing pattern]({{ site.baseurl }}/images/nomai/writing_motif.png "The repeating pattern of Nomai writing")

Outer Wilds has resonated with me since I first played it: partly because it's just an excellent game, partly because the gameplay and setting align with my personal preferences (exploration, spaceflight, etc.), and partly because the game's expression of philosphical themes helped me work through grief over a terminal illness in my family. I can think of no other work of art—in any medium—that has affected me as much. Recently I asked my wife to play through it with me (she did the exploration, I flew the ship; OW isn't a twitchy game, but flight requires some dexterity), and watching her play inspired me to engage with the Outer Wilds world via a personal project.

Despite being fictional aliens, the Nomai are deeply human and offer much to admire. I quite enjoy the aesthetics of the Nomai writing, so I decided to build [Nomai Writing](nomai-writing.com), a tool which maps any text to a unique spiral of Nomai-style writing.

# nomai-writing.com
### Main functionality
Input a message, get back a SVG encoding that message as a Nomai-style spiral. For example, here's the spiral for `Outer Wilds is an excellent game that you should definitely play`.

![Spiral example]({{ site.baseurl }}/images/nomai/example.svg "A short spiral from nomai-writing.com")

### Features
1. Arbitrary Unicode is supported, meaning messages can be in various languages or combinations thereof, include emoji, etc.
2. Distinct messages generate unique spirals.[[^1]]
3. Configurable handwriting mode simulates varying levels of imprecision as if spirals are written by hand.
4. Partial support for configurable spiral length. Adjusting the encoding base (details below) affects the output spiral.
5. Messages of up to ~paragraph length are supported. [NomaiText.jl](https://github.com/evanfields/NomaiText.jl/tree/main) supports arbitrary length if you're patient. E.g. here's the Gettysburg Address ASCII encoded:
    ![Gettysburg]({{ site.baseurl }}/images/nomai/gettysburg.svg "Nomai Gettysburg Address")

# Technical details
Still to-do.
## Typesetting spirals
### Encoding messages as a big integer
### Oracles
### Glyphs
### GlyphGrid
### Typesetting spirals
## Tools used
* Julia
* Luxor.jl
* Jot.jl
* AWS Lambda
----

[^1]: Well, _almost_ guaranteed to be unique. Very specific circumstances you almost surely won't run into in practice can cause message "collision" where two distinct messages generate identical spirals. See the [todo](https://github.com/evanfields/NomaiText.jl/blob/main/todo.md) for more details.