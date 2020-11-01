# A Condensed Guide on Compiling Quake Maps from TrenchBroom

*TrenchBroom's built-in compile dialog is a powerful tool for setting up and executing builds for processing .map files, the Quake source map format.  Rather than being tailored to a specific purpose, the dialog can set up builds for a wide variety of games and utilize any non-interactive command line program.  However, its power and general-purpose construction gives it a complexity that has proven to be an obstacle to new and experienced Quake mappers alike, who would simply like to convert their source maps into playable Quake levels.  While the official TrenchBroom manual provides a complete description of the tool, its coverage of the TrenchBroom "expression language", which lends the compile dialog much of its power, is not highly accessible.  From my observation of my own compile setup, it has occurred to me that a perfectly suitable setup for building Quake maps can be constructed with the exclusion of a large portion of the expression language; in fact, very little of the language need be used for a versatile setup.  Therefore, I am writing this guide to bridge the gap for users who would like to be able to specify their own compile configurations.  For the sake of simplicity, this guide will only cover setting up a compile configuration tailored specifically for Quake maps, using ericw's wonderful compile tools.*

## Requirements

* TrenchBroom 2020.2
* ericw's compile tools (qbsp, vis, and light)

## Concepts

Before we actually build a compile configuration, there are a few concepts to cover.  A compile configuration is just a list of tasks to carry out in order, not unlike a *script* or *batch file*.  There are three tasks we will cover: Export, Run Tool, and Copy Files.

### Export

Export writes the current state of your map in TrenchBroom to a separate file than what you are working with.  The file is postfixed with `-compile` so that if you have a map named `mymap.map`, export creates a file named `mymap-compile.map`.  One reason for having a separate exported map file from the file you do work on is that the exported map will exclude any layers with "Omit From Export" checked from the Layers pane.

### Run Tool

Run Tool is the work horse of the compile process; this is what runs the individual compile tools.  Any\* program or script that can be run from the command line can be executed, so long as it doesn't require user intervention.  We'll only be  focusing on three programs, our compile tools: qbsp, vis, and light.  There are two fields you must fill in: a path to your tool, and a list of arguments.  The tool path is easy; just click on the file picker and navigate to the tool you want to run.  The arguments are a little more sophisticated, and vary from tool to tool.  The arguments specify what files we want to read, where we want to write our output file, and what options we want to use.

> \*Some command line interpreters or *shells* have built-in commands.  These aren't true programs, and cannot be executed directly.  An example of a built-in command is `CD` in the Windows "cmd.exe" program.  (TODO: verify)

### Copy Files

Copy Files does what it says on the box.  You just need to provide *source* and *destination* paths.  The purpose of this is to move your compiled assets from the folder you are working in to the game folder.

When using any of these tasks, you can take advantage of TrenchBroom's *expression language*.  Though the language itself is quite sophisticated, you need only be concerned with only a small subset of features of the lanuage: *substitution*, *variables*, and *subscripting*; and then, subscripting is used in a very limited manner.

### Substitution

Substitution is the process by which an *expression* is replaced with text.  Substitution takes anything starting with a dollar sign followed by an opening curly brace (`${`) followed by an expression, followed by a closing curly brace (`}`) and writes in its place what the expression *evaluates* to.

### Variables

Variables stand in for values.  Most of the variables you'll use contain straight text, while one variable, `MODS`, contains a list of values.  Variables themselves make-up valid expressions, which when evaluated, produce the value that they contain.  For example, if I have `${MAP_BASE_NAME}` in any of the fields of my tasks, it will be replaced with *the name of my map*.  These are all the variables we'll be using:

* `MAP_BASE_NAME`: The name of your maps's file, minus the .map extension
* `MAP_DIR_PATH`: The folder path where your .map file resides
* `WORK_DIR_PATH`: The "working directory" from which tools are run. This is where relative paths are searched from, such as those not starting with a drive letter (`C:`) in Windows or forward slash in Linux.  For our purposes, this is set to the value of `${MAP_DIR_PATH}`
* `GAME_DIR_PATH`: The path to your Quake engine installation, or the folder in which your "id" folder sits
* `MODS`: A list of mods that you have active, plus the game folder.  This list always starts with the game folder, or "id1" for Quake.  Effectively, Quake can only have one mod used at a time, so you can think of this list as either consisting only of "id1" or being "id1" followed by the mod you're using, e.g. "copper".

### Subscripting

Subscripting is applied at the end of a variable, and evaluates to one item of a list of items.  The subscript is a set of square brackets, `[]`, that contain an integer or another expression that evaluates to an integer.  This integer is called an *index* and it starts with the first item counting from 0\*, up to N-1, where N is the length of the list.  In addition, the integer can also be -1, which will select the *last* item in the list.  **ALL YOU NEED REALLY TO KNOW** for this guide is that `${MODS[-1]}` will be substituted with the mod you have active in TrenchBroom, or "id1" if you don't have an active mod.

> Why count from 0 instead of 1?  The long answer has something to do with memory offsets and historical conventions, but the short answer is the programmers are weird.

### Putting it All Together

Now, let's put all of this together!  Let's say we have a field filled out with the following:

`${GAME_DIR_PATH}/${MODS[-1]}/maps`

Each of the segments starting with `${` and ending with `}` get substituted, with everything in-between getting glued in.  So if my `GAME_DIR_PATH` is `/home/4lt/.quakespasm` and I don't have any mods active, then the resulting field becomes `/home/4lt/.quakespasm/id1/maps`.

## Building a Compile Configuration (Or Three)

Here we'll apply what we've learned, and actually build a compile setup using my most commonly used options.  This is based on my own setup, and actually consists of three compile "profiles": 1 "Base" compile that runs fairly fast, 1 for compiling without lighting, and 1 high-quality "Release" compile.

To get started, open a map in TrenchBroom and go to `Run > Compile Map...` in the menu bar.

### Creating a Profile

When the compile dialog opens, you should see two panes at the top: one headed "Profiles", and one headed "Details".  At the bottom of the Profiles pane there are two buttons marked with a "+" and "-" for adding and removing profiles. Click the "+" button.

Now you should see your new profile, "unnamed", highlighted on the left, and two fields at the top of the details pane.  Set the Name field to `Base`, and leave the Working Directory as `${MAP_DIR_PATH}`.  Now click the "+" at the bottom of the *Details* pane and select "Export Map".  You'll see a new task created with `${WORK_DIR_PATH}/${MAP_BASE_NAME}-compile.map` set in the target field; leave it as is.

### Adding Compile Steps

With the task you just created still highlighted, create a new task by clicking the "+" at the bottom of the Details pane, but this time select "Run Tool".  For the Tool field, select the button labeled "..." and find your qbsp executable.  For the Parameters field, enter `${MAP_BASE_NAME}-compile.map ${MAP_BASE_NAME}.bsp`. As you can see, there are two arguments (AKA parameters) separated by a space.  The first argument is the filename of the map we compile, and the second argument is the .bsp file we want to output.  Notice now that this time we don't have `${WORK_DIR_PATH}` before the file name.  This is because tools are run *from* the working directory, so we can use relative paths, or the name of the map file since the map is in the same directory as the working directory.

> *If Run Tool doesn't require specifying the full path, why do I need it for Export?*
>
> As of writing, there is an issue open on the TrenchBroom GitHub to make all tasks consistent and work with only a relative path.

With the qbsp task highlighted, create a new Run Tool task, and point it to your vis executable.  Set the Parameters field to `${MAP_BASE_NAME}.bsp`.  Since vis and light both read and write the same file, they only require the one argument.

Follow the same steps you used to create the vis task, but this time make sure the vis task is highlighted, and Tool is pointed at your light executable.

> Note:
>
> Since spaces separate parameters, paths and filenames that include spaces will introduce problems. E.g. if our map's name is "john's map.map" and we try to run qbsp with `${MAP_BASE_NAME}-compile.map ${MAP_BASE_NAME}.bsp`, qbsp will actually see four arguments: "john's", "map-compile.map", "john's", and "map.bsp", causing qbsp to fail with an error.  In cases where you can avoid adding spaces, don't use them; in fact, avoid quotation marks, dollar signs, and percent signs as well. Heck, just stick to letters, numbers, and underscores if you want to keep it simple.  In cases where it's unavoidable, you can wrap the argument in quotes, so instead of `${GAME_DIR_PATH}/${MODS[-1]}` you'd write `"{GAME_DIR_PATH}/${MODS[-1]}"`.


### Copying Files

Now add a Copy Files task to the end of your profile.  Set its Source to `${WORK_DIR_PATH}/${MAP_BASE_NAME}.bsp` and its Target to `${GAME_DIR_PATH}/${MODS[-1]}/maps`.  Now create an additional Copy Files task, but change ".bsp" to ".lit"; even if you're not generating .lit files, it won't hurt to have this extra task here, the task will just fail but your map will still be copied to the appropriate directory.

Now you have completed your first profile!  From here, if you'd like, you can try it out by clicking "Compile".

### Additional Profiles

You may at some point decide you want to modify your compile profile, but leave the original in tact.  Let's do that now by creating profiles for no lighting and release.

Right click on your Base profile in the pane on the left, and select "Duplicate".  You should now have two profiles with the same name.  Rename it to "No Light".  Select the light task in the Details pane, and click the "-" button to remove it.

Create a new task, set its Tool field to "rm"\*, and Parameters to `${GAME_DIR_PATH}/${MODS[-1]}/maps/${MAP_BASE_NAME}.lit`.  This will remove your map's .lit file, if it exists, since it will be incompatible with your compiled map.

> \*BIG TODO. This will only work on Linux and other \*nix systems/platforms.  The Windows command for deleting files is DEL, but like CD, this is built into cmd.exe.  There's probably a way around this, like running a batch file as a tool, or cmd.exe itself with some sort argument that tells it to run DEL.

Now let's create a release profile.  Duplicate your base profile again, and name the copy "Release".  Select your light task, and insert "-extra4" to the beginning of the Parameters, with a space separating it from the path argument.  Notice the leading hyphen; this indicates that it is a *switch* or *option* that modifies the behavior of the light executable.  In this case, light will generate more rays when lighting your map, and average the results, resulting in lightmaps that are effectively antialiased.  Please note that switches must come *before* the file path, otherwise light will fail.  The same applies to vis and qbsp.

### Launch Configuration

The same expression language that you used for your compile profiles can be used in your launch parameters.  Close the compile dialog and from your menu bar go to "Run > Launch Engine...".  Set your Parameters to `-game ${MODS[-1]} +map ${MAP_BASE_NAME}`.  If you've already compiled your map, you can hit "Launch" to launch the game (so long as your engine is configured).

### Additional Resources

For additional parameters for the compile tools, ericw's tools come provided with the tools; the documentation is also available online at <https://ericwa.github.io/ericw-tools/>
