---
title: An anecdote about irradiance
author: Hannah Weller
date: '2024-05-30'
slug: an-anecdote-about-irradiance
categories:
  - misc
tags:
  - color
  - light
  - risd
  - art
subtitle: ''
summary: 'A good reminder about why color scientists care so much about light sources.'
authors: [Hannah Weller]
lastmod: '2024-05-03T16:39:52+03:00'
featured: no
image: 
  caption: 'This might count as giving away the whole plot in the trailer.'
  focal_point: ''
  preview_only: no
projects: []
---

# The anecdote 

A couple of years ago, I got an email from my friend Anna Rose, at the time a textiles curator at the Rhode Island School of Design (RISD), asking me if I knew anything about infrared photography. She wanted to know if she could use it to uncover some lost history, which was such an exciting idea my mind immediately rejected it a fantasy that I was going to have to shoot down.

The context here was that a team of researchers from [Fa'asamoa Arts](https://www.faasamoaarts.com/) (Su'a Uilisone Fitiao, Reggie Meredith Fitiao, and Mary Anne Bordonaro) were cataloging the painted barkcloths (siapo) from American Samoa in RISD's textile collections. I could not be less qualified to describe the history or cultural significance of the siapo tradition in American Samoa, or to speak on the appropriateness of RISD having these particular examples (although the Fa'asamoa Arts site and [this book by Mary J. Pritchard](https://www.loc.gov/item/84073325/) seem like excellent places to start learning more, and I've put the full explanation for the project that I was sent at the end of this post). Here's an example they had out when I went to visit:

![A circular siapo with a central checkerboard pattern, a border of leaves, and a scalloped design around the edge](images/IMG_1343.JPG)

The reason she had contacted me, though, was because o'a--[a brown dye](https://www.faasamoaarts.com/post/brown-dye-american-samoa-art) used in creating siapo, made from a tree of the same name--oxidizes to near-black over time. So rather than intricate geometric patterns, some of the siapo the research team were cataloging looked like this:

![A painted barkcloth (siapo) that appears completely black due to oxidized dye](images/IMG_1328.JPG)

But RISD also had, although apparently nobody knew how to use it, a professionally converted full-spectrum camera capable of imaging in ultraviolet and infrared. For imaging nerds: it was specifically a MaxMax-modified Canon UV-VIS-IR camera with an X-Nite 1000 infrared lens. IR photography has been [plenty often](https://mci.si.edu/infrared-and-ultraviolet-imaging) in art conservation, so they thought it was worth trying to see if we could uncover the hidden siapo designs using this camera. So one afternoon I walked the ten or so minutes from Brown's campus to RISD's collections to help out with this.

I will be honest: I thought I would show them how to use the camera, and then be the wet blanket explaining why it didn't reveal the design. I think when we photograph things in UV or IR, we're always hoping to reveal a hidden world, the hyperspectral equivalent of the numbers on the [Ishihara colorblindness test cards](https://en.wikipedia.org/wiki/Ishihara_test). And as a scientist I think I'm inclined to be skeptical of anything I want badly to be true. In this case, for the design to be visible, there would need to be a difference in infrared absorption between the two dyes, which seemed unlikely since lama (the black dye) is a combination of o'a (the brown dye) and burned candlenut kernels. Differences in IR absorption would probably be minor and hard to see without an intense source of infrared light.

And guess what! Yeah, we saw nothing. I didn't take a picture, but fortunately, I can recreate for you almost exactly what it looked like when we popped the IR lens on the camera and pointed it at the siapo, because it was literally just black:

![A black rectangle.](images/black.JPG)

At this point I said something like well, haha, that's usually how it goes, sorry, although if you want to try something else, I guess you could get a light source with a lot of infrared light instead of photographing it just under your presumably fluorescent ceiling lights, but I wouldn't get your hopes up.

And Anna Rose asked me "Well, what's the cheapest source of infrared light?" and I said, "THE SUN, I guess, but it's not like you can take this 150-year-old barkcloth outside," and she said oh we totally can, as long as it stays on museum grounds, and she and her intern promptly wheeled it through the museum cafe and onto the patio where people were having their lunch. 

It was fine, but this move felt so illegal to me at the time that I honestly feel like writing about it now is incriminating. I don't know the statute of limitations on taking a piece of preserved cultural heritage onto a museum cafe patio. On the other hand it worked.

![The solid black siapo from before, this time with the underlying geometric design revealed in a purple-tinted infrared photo.](images/IMG_1325.JPG)

I distinctly remember that her intern pulled over a chair from an empty table, stood on top of it, flipped open the camera with the IR lens still on, and actually gasped out loud. And then she showed me, and I also gasped out loud.

![A DSLR camera pointed at a barkcloth, with the infrared preview showing the revealed design.](images/IMG_0946.JPG)

I'm not even sure how to describe how satisfying this moment was. It was like a scenario I would make up in a daydream about all my esoteric color knowledge suddenly feeling useful. A lot of science is working for months or years on something you find interesting only to show it to other people and have most of them say at most, "oh, nice," and this took about fifteen minutes to provide something tangible that everyone found exciting. This is how science works in like, forensic TV shows.

# The irradiance

The first image we took with the IR lens was indoors with a standard fluorescent ceiling light. Fluorescent lights are meant to more or less emulate the way that daylight appears to our eyes by providing peaks of emitted light around the peak wavelength sensitivities of our cone cells (around 430, 540, and 570 nm; image from [Waveform Lighting](https://www.waveformlighting.com)):

![Irradiance spectrum for a typical fluorescent light, from https://www.waveformlighting.com](images/waveformlighting_fluorescent.png)

Those peaks do a great job of illuminating objects for human vision so that they appear more or less as they would under natural sunlight. But they provide very little light outside of those peaks, especially for red and infrared wavelengths (>650nm). Compare that to the amount of light above 650nm in sunlight:

![Irradiance spectrum for sunlight, from https://www.waveformlighting.com](images/waveformlighting_daylight.png)

People can't see infrared light, so this doesn't make a big practical difference in how things look indoors vs. outdoors. But the IR lens on the camera filters out any non-infrared light, which under a fluorescent lightsource means it's just filtering out ALL of the available environmental light. Even if a surface differs in how much it absorbs and reflects IR wavelengths--leading to a difference in infrared "color"--we weren't able to see that when there was no available infrared light to absorb or reflect. 

I like this example because it's just enough outside the normal human sensory experience to feel non-obvious, but it's a phenomenon that happens all the time with imaging without us ever clocking it. We think of color as an inherent property of an object (e.g. "my dress *is* green, it just looks black with the bad lighting in the hallway"), but it's not purely academic to point out that color is an emergent property of how a surface interacts with available environmental light. It can have practical consequences!

A much more succinct, intuitive example is Olfaur Eliasson's "Room for One Colour", which he very neatly demonstrated in about ten seconds of a [Netflix trailer](https://www.youtube.com/watch?v=fDE7A68hVaw): 

![Olafur Eliasson turning on a lightbulb in a room lit with yellow light.](images/monochromatic1.png)
![Turning on the white light reveals a rainbow.](images/monochromatic2.png)

### Information about the siapo project

Here's the info that RISD sent me when I asked. Any questions about the project should certainly go to the research team or possibly the museum, not me.

> On June 7th, 2022, researchers froman indigenous arts nonprofit in American Samoa came to the RISD Museum to view Samoan barkcloth, called siapo. Their organization, known as Folauga o le Tatau Malaga Aganu’u Fa’asamoa (a.k.a. Fa’asamoa Arts), seeks to educate others about the indigenous arts of American Samoa and promote the integration of these ancient art forms into modern lives.Their travel was funded by the Amerika Samoa Humanities Council, and will culminate in a new creative nonfiction book: *U’A: Ancient Art through Contemporary Eyes*. The research team consists of three people. Su’a Uilisone Fitiao is a Tufuga Ta Tatau (a traditional tattoo master), a siapo maker, and woodcarver. Reggie Meredith Fitiao (MFA) is a siapo maker and art educator. Mary Anne Bordonaro is a grant writer, short story writer, and will be the author of the book being funded by this grant.
