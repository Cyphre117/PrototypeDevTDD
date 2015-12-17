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

## Type 3 engine on windows desktop with openGL ES

Type3 engine was initially built as an SDL/OpenGL based technology with Visual Studio, since in the early stages of design it was not clear what platform we would be developing for. When the decision was made to create an android game, it was clear that we would have to switch the graphics API to OpenGL ES, and therefore it was of utmost importance to test whether or not our current code could be adapted for use with embedded systems or if we would have to rebuild everything from scratch. The decision was then made to create a quick test branch of the current code substituting GL initialisation code with GLES code.
The first difficulty encountered was that the GL extension wrangler library (GLEW) used to load the API's extensions did not offer support for GLES (probably because there is no reason anyone would want to write a desktop application based on ES, apart from testing purposes), and therefore we would have to load them manually. To do this, we used SDL's SDL_opengles2.h header, which provides all the function definitions, macros and typedefs useful for GLES v2.x, and SDL's SDL_GL_GetProcAddress() function, used to grab and assign the API function pointers.

The following is an example of the implementation:
1. suppose our code requires the use of the function glCreateProgram()
2. make sure  SDL_opengles2.h is being included in the file where we want to use the function
3. looking up the definition in  SDL_opengles2.h, identify the types for the function's return value and input parameters (in this case Gluint and void respectively)
4. create a typedef similar to this: 

```cpp
typedef GLuint(APIENTRY * GL_CreateProgram_Func)(void)
```

(APIENTRY resolves to __stdcall)

5. Create a variable with the function's name using the new typedef, making sure the scope is local to where the function has to be called

```cpp
GL_CreateProgram_Func glCreateProgram = NULL
```

6. Finally, assign the function's address to the variable using

```cpp
glCreateProgram= (GL_CreateProgram_Func)SDL_GL_GetProcAddress("glCreateProgram")
```

7. Now `glCreateProgam()` can be used as normal in that scope.

This process has to be repeated for every required function.
Since most of OpenGL's functions we were using had a GLES counterpart with the same or similar name, the engine's structure only suffered very minor changes apart from substituting GLEW includes and initialisation calls with the process described above. The only other addition worth mentioning is the need to specify a GLES context when creating the window:

```cpp
SDL_GL_SetAttribute(SDL_GL_CONTEXT_PROFILE_MASK,SDL_GL_CONTEXT_PROFILE_ES);
SDL_GL_SetAttribute(SDL_GL_CONTEXT_MAJOR_VERSION, 2);
SDL_GL_SetAttribute(SDL_GL_CONTEXT_MINOR_VERSION, 0);
```

In the end we were able to have a window with a GLES context running on Windows, and the only non reusable code ended up being the few lines of test shader written in desktop hlsl.

## Compiling SDL for Android

After confirming our technology could be adapted to the new platform, we needed to put it to practice. SDL is written in c and our engine in c++, so to compile it and run it on an android device the Native Developement Kit (NDK) has to be used.
The first step was therefore to compile the example android project provided with the SDL 2.x source. This is an Eclipse/androidSDK project that contains all the Java files required to start the application and interface with SDL's function calls and any c/c++ files the user wants to add.
For a number of unfortunate coincidences however, this proved to be a harder task than it seemed. In fact, android sdk support for Eclipse has recently been discontinued in favor of Android Studio, which means the example project had to be ported to the new IDE unless we wanted to try and find an old version of Eclipse with androidSDK and develop in an obsolete environment. Other than the minor annoyances of changing IDE however, the biggest problem was that since Android Studio is a relatively new tool, its current NDK support is not complete and anything more than compiling a single c/c++ file with native function definitions in it requires one of the following things:

Either downloading an experimental plugin for Gradle (the build automation system used by Android Studio) with NDK support, which is still being worked on in these months

Or tricking Gradle into thinking your project has no native code by changing the build script, and then running NDK build tools manually from the command line, effectively compiling your java project and c/c++ files in two separate steps.

After spending a lot of time trying to automate the whole process with the experimental plugin and struggling with its DSL (the language use to write gradle scripts) syntax  changing week by week, we decided to temporary settle for option two.
The following list describes the process with which we managed to get the SDL android project to run on Android Studio's emulator (Nexus 5):

1. Download Android Studio, the android NDK and the SDL2 source code.
2. Install the sdk platform and tools for the target platform you want to develop for(NDK supports up to 21, we used 18 in this test). This is done via the SDK-manager tool that comes with the IDE.
3. Set the environment path variable to where the ndk folder is to be able to call build commands later on (on windows 10: control panel > system and security > system > advanced system settings > environment variables).
4. Start android studio and choose "import project (eclipse, gradle,...)".
5. Select the “android-project” from the SDL source folder.
6. Copy the SDL source folder in “YourNewProject\app\src\main\jni”.
7. Create a main.cpp file in “YourNewProject\app\src\main\jni\src” and add your code(in our test: initialise SDL, create a GLES window, draw a triangle with a shader, wait for input to quit).

Then make the following changes to the following files (using api 18 for examples)
1. YourNewProject\app\src\main\jni\Android.mk:
add “APP_PLATFORM := android-18”

2. YourNewProject\app\src\main\jni\src\Android.mk:
 change “yourSourceHere.c” to “main.cpp” and make sure SDL_PATH is the correct relative (or absolute) path to your SDL source folder

3. C:\Users\<USER>\.gradle:
create a file named “gradle.properties” and add “android.useDeprecatedNdk=true” in it

4. YourNewProject\local.properties:
add “ndk.dir=<path to your ndk folder>”

5. YourNewProject\app\bulid.gradle:
change compileSdkVersion, minSdkVersion and targetSdkVersion to 18
inside defaultConfig scope add:
ndk {
moduleName 'main'
}

inside android scope add:
sourceSets.main { 
		jni.srcDirs = []
        	jniLibs.srcDir 'src/main/libs' 
}
This is where we tell gradle we don't have native code to compile by clearing the native source directories paths

6. In android studio's terminal (or in the command prompt) move to YourNewProject\app\src\main\jni and run the “ndk-build” command. This will build SDL and the cpp files previously added.

7. Finally, run the gradle build and start the  correct emulator (in our test a nexus5 with api 23).

With this process we were able to run an OpenGL ES shader on android without writing any Java code thanks to SDL. The downside, other than the build process being split in two steps, is that to prevent gradle from trying and failing to build our native code we cannot see our c/c++ files in android studio's project tree.

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
