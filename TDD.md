% The Enemy Within: Technical Design Document

# Overview
## Team
* David
* Davide
* Thomas

## feature outline
The game is about cells growing and changing state on a hex grid. The player has to manage the growth of cells, cells will move autonomously if the player doesn't take action. Each cell has internal state that changes based on adjacent cells.
* Hex grid based
* touch input
* nodes preform checks on adjacent nodes
* nodes change their state + spawn based on user input
* 'infinite' grid
* images: load sprite sheets, transparency, scale, rotate, tint
* Audio: play once, loop, pitch variation, random selection
* Physics: minimal path finding

## platform
* Android Tablet

## Technology
* C++
* OpenGLES
* SDL2
* SDL2_mixer
* SDL2_image

## Risks
* not having animation
* simplified audio
* harder to debug
* might not build for large platform
* not enough time to polish
* 'infinite' grid required memory allocation / deallocation = crashes

# Research
## Feasability
* A team last year did it
## Current Progress
* david is working on building an engine for OpenGL
* davide is building openGLES projects for android
* Thomas is iterating on prototypes
## Challenges identified
* easy to use audio / image pipeline that doesn't require coder intervention

# Implementation

## Source Control
* Git

## Features
* graphics manager
* audio manager
* game states
* Grid

## Grid detail
* split the grid into chunks
* each chunk is basically an array of pointers to nodes, or nullptr if node is empty
* when a cell tries to grow off the edge of a a chunk
* * if it exists in the chunk list add it in there
* * if the chunk doesn't exist allocate the memory and connect it to the previous chunks

There are games that already use this approach

## Testing Tool
While the engine is under development it would be very valuable to have some kind of instant feedback tool for the other members of the team. Rather than having to wait to have a programmer integrate some new asset into the game they could test and iterate quickly on designs in their own time. The testing tool could show previews of animations under certain game conditions or how the hand made art would look alongside some procedural art or shader. It could also allow for the testing of audio assets, randomised events, or asset pools. 

## Infinite grid
While it is impossible to have a truly infinite grid without infinite memory it is possible to create a game world so large that no user will conceivably reach the boundary. Game with such worlds already exist, the most popular of which is Minecraft. Minecraft's world is three dimensional, and infinite in two dimensions. The game is divided into 'chunks', each chunk being dynamically created as required and unloaded when no longer needed.

Will likely use STL containers since they are robust and well documented, the reference below makes use of a `std::map` to store chunks of the grid. Each chunk can be sorted within it's container for easy lookup. When checking the contents of some grid coordinate, first look in the container to see if the chunk with that coord exists. If it does search within the chunk to find the coord, if not then allocate the chunk.

References:  

* https://github.com/gummikana/infinite_grid.cpp

# Feature List
## Gameplay
* splash screen
* main menu
* optons menu
* Touch buttons
* Drag navigation
* Pinch to zoom
* "infinite" grid

## Art
* loading png images
* static images
* rotated / scaled images
* animation from sprite sheets
* randomised image selection 
* randomised animation
* transparency
* colour tinting

## Audio
* loading wav files
* loading mp3s
* play audio once
* volume control
* loop audio
* randomly select sound
* pitch shifting
* time shifting

# Schedule
* first pass engine
* android test
* * gets touch events
* * loads images
* * plays audio
