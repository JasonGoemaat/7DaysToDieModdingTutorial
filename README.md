# 7 Days To Die Modding Tutorial

## Game Commands

Hit `F1` to open the console when playing the game.

### Developer Mode

Enter 'dm' in the console to enable developer mode.

* `Q` hotkey will toggle buffGod, which makes you invulnerable and gives you full water and food, as well as removing any effects currently on you (including positive buffs)
* `H` hotkey toggles fly mode
* `ESC` menu has a section on the right allowing you to change time of day, toggle noclip, and other things
* `SHIFT+Q` will teleport you to where your cursor is pointed.  Be aware that if you fall a great height from there without god mode enabled you could die
* `CTRL+Click` on the map to teleport to that position
* When you hit `ESC`, there will an `Open POI Teleporter` option.  You can filter the POIs and teleport to them, for instance type 'trader' to see the traders and teleport to them easily
* `F6` opens a menu letting you spawn entities
* `SHIFT+F6` opens a menu letting you do 'actions' like some buffs
* `F8` cycles displaying FPS and Chunk data including heatmap.  Heatmap turns green->yellow->red until a screamer spawns and it switches back to green

To leave your player in a position and fly around, make sure fly mode is *disabled* (toggle off
with `H` if it is on).  Then:

* Hit `F5` to enable third-person view
* Hit `P` to detach camera
* Hit `[` to switch to moving camera instead of player

To get back into your body, hit `P` again and `F5` to go back to first person view.  This set of
commands took me a bit to figure out because if you don't do them all the game reacts funny.
For example if you're in first person mode, you still see your axe and can move around, but
your actual character is it's original position and is invisible, so any actions you do like
firing a weapon occur at that position.  And if you use `P` but not `[`, then the camera is
actually static and you move your player around the game world.

## Modding Basics

In steam, right-click on the game and select 'Manage' -> 'Browse local files'
to open the folder where the game is installed.  Create a `Mods` directory if it
doesn't already exist, and create a `ModInfo.xml` file in that directory:

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<xml>
    <ModInfo>
        <Name value="My Mod" />
        <Description value="Sample tutorial mod" />
        <Author value="Jason Goemaat" />
        <Version value="1.0.0" />
    </ModInfo>
</xml>
```

Now if you're in the game, exit it completely and restart from Steam for it
to register your mod.  Now if you make any changes you only need to exit your
current game to the main menu and continue, not restart the game completely.

### Game Data

The game data is incorporated into XML files in the `Data` directory under the game
(where you created the `Mods` directory).  Open it with Visual Studio Code and you
can browse and easily search for things to copy and modify.

### Mod Structure

Modding is done by creating a file with the same name of the file you wish to modify.
For instance when want to change the frame shapes material in `Data/Configs/materials.xml`,
I create a file `Mods/MyMod/Configs/materials.xml`.  

The mod file does not follow the same structure as the game files, rather it
has a `<configs>` root element with commands underneath on how to modify
elements in the game file with the same path and name.  For instance, the game
file has this structure with the material used by frame shapes (`...` is used to 
omit all the other materials)

```xml
<?xml version="1.0" encoding="UTF-8"?>
<materials>
  ...
  <!-- A20 Shape Materials -->
  <material id="Mwood_weak_shapes">
    <property name="damage_category" value="wood"/>
    <property name="surface_category" value="wood"/>
    <property name="forge_category" value="wood"/>
    <property name="Hardness" type="float" value="1"/>
    <property name="stepsound" value="wood"/>
    <property name="stability_glue" value="40"/>
    <property name="Mass" type="int" value="5"/>
    <property name="MaxDamage" value="100"/>
    <property name="Experience" value="2"/>
  </material>
  ...
</materials>
```

Whereas the mod's `materials.xml` file looks like this:

```xml
<configs>
  <set xpath="/materials/material[@id='Mwood_weak_shapes']/property[@name='MaxDamage']/@value">10000</set>
  <set xpath="/materials/material[@id='Mwood_weak_shapes']/property[@name='stability_glue']/@value">300</set>
</configs>
```

The elements under `<configs>` in a mod are commands for modifying the original xml file:

* `set` - set a value
* `remove` - remove an element or attribute
* `append` - append element(s)

Each command has an `xpath` attribute specifying where the modification is to occur
in the game's file.  For example we have this:

```xml
  <set xpath="/materials/material[@id='Mwood_weak_shapes']/property[@name='MaxDamage']/@value">10000</set>
```

In short we are targeting the `value` attribute of the `property` element that has a `name`
attribute equal to `MaxDamage`, which exists under a `material` element with the id attribute
equal to `Mwood_weak_shapes`, which exists under the `materials` element.  The mod file
must have the same name and be in the same directory under our mod as the file we wish to modify
is under the `Data` directory in the game folder.

#### set

The `set` operation changes the value of an attribute (the normal case) or the value
inside an element.

```xml
  <set xpath="/materials/material[@id='Mwood_weak_shapes']/property[@name='MaxDamage']/@value">10000</set>
```

An xpath is a 'path' through the xml, using a slash `/` to define the structure, braces `[]` as
a type of filter, and the at symbol `@` to denote an attribute instead of an element.  Parsing
this xpath then we see that we start at the root `/`, with the `materials` element (the root
element in the game's `materials.xml`), and look for `material` elements filtering on ones
where the `id` attribute (using `@` to denote an attribute) has the value `Mwood_weak_shapes`,
using single quotes to denote that it is a string.  Under that we are looking for `property`
elements with a `name` attribute of `MaxDamage`.  For those property elements, we are changing
the `value` attribute (using `@` to denote it's an attribute).  The text inside the
`<set>` and `</set>` tags is then used to replace that value attribute in the game.

In short it is changing this:

```xml
<materials>
  <material id="Mwood_weak_shapes">
    <property name="MaxDamage" value="100"/>
```

to this:

```xml
<materials>
  <material id="Mwood_weak_shapes">
    <property name="MaxDamage" value="10000"/>
```

#### remove

The `remove` operation removes an attribute or an entire element from the game files.
In our example we are changing the frame shape blocks in `blocks.xml` by removing
the `property` element that gives it the 'wood' tag, so it isn't especially vulnerable
to shotgun fire.  We create a `Configs/blocks.xml` file under our mod directory to match
the position of the game file `Data/Configs/blocks.xml`.

As in any xml mod file, we create a root `configs` element that tells the game
we want to modify the xml configs and add our `remove` element.


```xml
<configs>
  <remove xpath="/blocks/block[@name='frameShapes']/property[@name='Tags']" />
</configs>
```

The xpath tells us to:

* start at the root `/`
* find the `blocks` element
* find a `block` element under that
  * with the attribute (`@`) `name` that equals `frameShapes` (by using `[]`, `@name` to specify the name attribute, `=`, and the value for that attribute in single quotes `'frmaeShapes'`)
* find the `property` element under that block element that has the `name` attribute equal to `Tags`

So no we've found the `property` element and we will remove it.

#### append

Append lets you add entire elements to the game file.  In our fist sample we
created a second recipe for repair kits that lets us create them with one wood.
If we look at the game file it has a root `<recipes>` element with individual
`<recipe>` elements under it.  Here we specify we are adding new elements
to that root `<recipes>` element and copy the recipe for the repair kit
but change the ingredients:

```xml
<configs>
  <append xpath="/recipes">
    <recipe name="resourceRepairKit" count="1">
      <ingredient name="resourceWood" count="1"/>
    </recipe>
</configs>
```
