---
layout: post
title: "Blender commands"
date: 2018-07-12
---

I'm teaching myself [Blender](https://www.blender.org/) to improve my 3D Asset Generation skills. I'm using Blender's own [tutorials](https://cloud.blender.org/p/game-asset-creation/56041550044a2a00d0d7e069), and these notes are my record of the various commands and techniques mentioned in the videos.

## Basics

Event |Meaning
:----------|:----------
`Shift s`|`Snap` menu (e.g. cursor to center)
`Tab` | Toggle: edit Mode / object Mode
`g`| grab
`s` | scale
`r` | rotate
`Ctrl z` | undo
`Shift Ctrl -z` | redo
`Ctrl Alt u`| User preferences

## selection

Event |Meaning
:----------|:----------
`RMB` | click select
`Shift RMB` | multiple select
`b`| border select (drag rect)
`a`| toggle: select all / deselect all

## Camera movement
The directions in the following table describe the movement of the camera, not objects in the scene

Event |Meaning |`Ctrl+...` |`Shift+...`
:----------|:----------|:----------|:----------
`MMB`|rotate cam |zoom| pan freely
`Scroll MMB`|zoom|pan left/right|pan up/down
`Numpad 1`|front view | back | -
`Numpad 3`|right view | left view | right view
`Numpad 7`|top view | bottom view | -
`Numpad 2`|orbit down | pan down  | -
`Numpad 4`|orbit left | pan left  | roll left
`Numpad 6`|orbit right | pan right | roll right
`Numpad 8`|orbit up | pan up | -
`Numpad 9`|orbit right 3.142 | - | -
`Numpad .`| View selected | ?? | ??
`Numpad 5`| perspective/orthographic view| - | -
`Numpad 0`| user/camera view| - | -
`c` | ?? | ?? | center
`Home` | view all | ??view all | -

Other useful view controls

Event |Meaning
:----------|:----------
`Shift f`|Fly mode (WASD, arrows, etc)
`Ctrl Alt q`| quad view / single view

## Object visualisation

Event | Meaning
----------|----------
`z`| solid view / wireframe
`Alt z`| solid view / textured

`Properties sidebar`->`shading` tab has additional controls
* GLSL
* backface culling
* MatCap (e.g. red, mirrored, etc)


## layers

Event | Meaning
----------|----------
`m`|Move (selected object) to layer

The object tab of the properties window has a layers control that achieves a similar effect.


## Layout customization

Event |Meaning
:----------|:----------
`t`|Tool shelf sidebar
`n`|Properties sidebar
`Ctrl MMB`|Cycle through tabs
`Ctrl left/right`|Cycle through windows (up/down)

LMB and drag in window triangle to split or join
RMB on window boundary to split or join


## Configuring blender

Here are the non-default configuration settings that I'm currently using.

Interface tab 
* Auto Depth
* Auto Perspective

Addons
* Layer management - creates toolshelf tab for giving layers descriptive names
* Rigify


## Text editing

To edit the text of a text object, `tab` to enter edit mode, `tab` again when
