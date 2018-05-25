# Pure Data: Unofficial File Format Spec

### General
* Statements are typically semicolon (;) delimited.
* Statements generally begin with a ```#``` followed by an uppercase character, either ```N``` (canvas, struct), ```X``` (object, other), or ```A``` (array data).
* Commas (,), tokens (e.g. $1), and non-delimiting semicolons (;) in text/message objects are typically escaped (e.g. ```\,```, ```\;```, or ```\$1```).
* N.B. this is a high-level overview, intended to be used for parsing and generating .pd files.

### Canvas

* Begins with ```#N canvas```
* Followed by, in order:
	* Origin X (single integer)
	* Origin Y (single integer)
	* Canvas Width (single integer)
	* Canvas Height (single integer)
	* Font size (single integer)
* For example:
	* ```#N canvas 100 100 400 700 12;```

##### Canvas Properties
* Hide scrollbars:
	* ```#X scroll 1;``` may be appended to the bottom of the file.
* Graph on parent (on):
	* Begins with ```#X coords```
	* Followed by, in order:
		* Data-scaling: left (single integer)
		* Data-scaling: top (single integer)
		* Data-scaling: right (single integer)
		* Data-scaling: bottom (single integer)
		* Width (single integer)
		* Height (single integer)
		* Hide name and arguments (single integer; 1: off, 2: on)
		* X Offset (single integer)
		* Y Offset (single integer)
		* Typically at end of file or before ```#X scroll 1;```
		* N.B. data-scaling values apply for garrays and data structures. They set vertical (top, bottom) and horizontal (left, right) ranges.
* Graph on parent (off):
	* Begins with ```#X coords```
	* Followed by, in order:
		* Data-scaling: X value 1 = ```0``` if ```x-scale > 0``` else ```abs(x-scale) * width```
		* Data-scaling: Y value 1 = ```0``` if ```y-scale < 0``` else ```y-scale * height```
		* Data-scaling: X value 2 = ```x-scale``` if ```x-scale > 0``` else ```abs(x-scale) * (width - 1)```
		* Graph width (single integer)
		* Graph height (single integer)
		* 0 (single integer, possibly graph on/off)

##### Subpatches
* Subpatches also begin with ```#N canvas```
* Followed by, in order:
	* Origin X (when open; single integer)
	* Origin Y (when open; single integer)
	* Width (when open; single integer)
	* Height (when open; single integer)
	* Subpatch name (or, if empty, ```(subpatch)```)
* This is followed by a list of objects and connections located within the subpatch. N.B. this includes properties like ```#X scroll 1;```.
* Finally, a closing tag appears. This is in the form ```#X restore```, followed by:
	* Subpatch Origin X (single integer)
	* Subpatch Origin Y (single integer)
	* Either ```pd``` (e.g. for a typical subpatch) or ```graph``` (e.g. for garrays)
	* If the subpatch has a name, the name (optional)

### Objects (General)
* Begin with ```#X obj```
* Followed by, in order:
	* Origin X (single integer)
	* Origin Y (single integer)
	* Object name
	* Any arguments to object
* For example:
	* ```#X obj 100 100 osc~ 440;```

### Connections
* Begin with ```#X connect```
* Followed by, in order:
	* From object (correlated to line number, 0-indexed after ```#N canvas```; single integer)
	* From object outlet index (single integer)
	* To object (correlated to line number, 0-indexed after ```#N canvas```; single integer)
	* To object inlet index (single integer)
* For example:
	* ```#X connect 0 0 1 0;```

### Colors
* Object colors are often expressed as ```-```-prefixed 18-bit 1-indexed decimal integer values (between ```-1``` and ```-262144```).
* In some cases (like garray colors), they may be expressed as ```#```-prefixed 6-digit hexadecimal number values (from ```#000000``` to ```#ffffff```).

### Specific Objects

##### Messages
* Begin with ```#X msg```
* Followed by, in order:
	* Origin X (single integer)
	* Origin Y (single integer)
	* Message text (if present)

##### Numbers
* Begin with ```#X floatatom```
* Followed by, in order:
	* Origin X (single integer)
	* Origin Y (single integer)
	* Width (single integer; unit: characters)
	* Minimum value (single number)
	* Maximum value (single number)
	* Label position (single integer; 0: left, 1: right, 2: top, 3: bottom)
	* Label text (string)
	* Receive symbol (string)
	* Send symbol (string, followed by unescaped comma character (,))
	* ```f``` (single character)
	* Width (single integer; unit: characters)
* N.B. the second instance of the width, preceded by the ```f``` character, appears to be able to override the first instance, but not vice-versa. I.e. if the patch is opened with two conflicting values, the latter will decide the displayed width (and overwrite the former, if the patch is saved).

##### Symbols
* Begin with ```#X symbolatom```
* Followed by, in order:
	* Origin X (single integer)
	* Origin Y (single integer)
	* Width (single integer; unit: characters)
	* Minimum value (single number; likely unused)
	* Maximum value (single number; likely unused)
	* Label position (single integer; 0: left, 1: right, 2: top, 3: bottom)
	* Label text (string)
	* Receive symbol (string)
	* Send symbol (string, followed by unescaped comma character (,))
	* ```f``` (single character)
	* Width (single integer; unit: characters)
* N.B. as noted with the Number object, the second instance of the width, preceded by the ```f``` character, appears to be able to override the first instance, but not vice-versa. I.e. if the patch is opened with two conflicting values, the latter will decide the displayed width (and overwrite the former, if the patch is saved).

##### Comments
* Begin with ```#X text```
* Followed by, in order:
	* Origin X (single integer)
	* Origin Y (single integer)
	* Comment text (if present)

##### Dropdowns
* Begin with ```#X dropdown```
* Followed by, in order:
	* Origin X (single integer)
	* Origin Y (single integer)
	* Width (single integer; unit: characters)
	* Output type (single integer; 0: index, 1: value)
	* ```0``` (single integer, possibly unused)
	* Label position (single integer; 0: left, 1: right, 2: top, 3: bottom)
	* Label text (string)
	* Receive symbol (string)
	* Send symbol (string, followed by unescaped comma character (,))
	* ```f``` (single character)
	* Width (single integer; unit: characters)
* N.B. as noted with the Number and Symbol objects, the second instance of the width, preceded by the ```f``` character, appears to be able to override the first instance, but not vice-versa. I.e. if the patch is opened with two conflicting values, the latter will decide the displayed width (and overwrite the former, if the patch is saved).

##### Bangs
* Begin with ```#X obj```
* Followed by, in order:
	* Origin X (single integer)
	* Origin Y (single integer)
	* ```bng``` (string)
	* Size (single integer)
	* Hold (single integer)
	* Interrupt (single integer)
	* Init (single integer; 0: off, 1: on)
	* Send symbol (string)
	* Receive symbol (string)
	* Label text (string)
	* Label X offset (single integer)
	* Label Y offset (single integer)
	* Font index (single integer; 0: DejaVu Sans Mono, 1: Helvetica, 2: Times)
	* Font size (single integer)
	* Background color (single integer; ```-```-prefixed)
	* Foreground color (single integer; ```-```-prefixed)
	* Label text color (single integer; ```-```-prefixed)

##### Toggles
* Begin with ```#X obj```
* Followed by, in order:
	* Origin X (single integer)
	* Origin Y (single integer)
	* ```tgl``` (string)
	* Size (single integer)
	* Init (single integer; 0: off, 1: on)
	* Send symbol (string)
	* Receive symbol (string)
	* Label text (string)
	* Label X offset (single integer)
	* Label Y offset (single integer)
	* Font index (single integer; 0: DejaVu Sans Mono, 1: Helvetica, 2: Times)
	* Font size (single integer)
	* Background color (single integer; ```-```-prefixed)
	* Foreground color (single integer; ```-```-prefixed)
	* Label text color (single integer; ```-```-prefixed)
	* Current value (single number; only recalled if init is enabled)
	* Nonzero value (single number)

##### Number2
* Begins with ```#X obj```
* Followed by, in order:
	* Origin X (single integer)
	* Origin Y (single integer)
	* ```nbx``` (string)
	* Width (single integer; unit: (apparently) characters)
	* Height (single integer)
	* Minimum value (single number)
	* Maximum value (single number)
	* Logarithmic scaling (single integer; 0: off, 1: on)
	* Init (single integer; 0: off, 1: on)
	* Send symbol (string)
	* Receive symbol (string)
	* Label text (string)
	* Label X offset (single integer)
	* Label Y offset (single integer)
	* Font index (single integer; 0: DejaVu Sans Mono, 1: Helvetica, 2: Times)
	* Font size (single integer)
	* Background color (single integer; ```-```-prefixed)
	* Foreground color (single integer; ```-```-prefixed)
	* Label text color (single integer; ```-```-prefixed)
	* Current value (single number; only recalled if init is enabled)
	* Log height (single integer)
	* ```0``` (single integer, possibly unused)

##### Sliders
* Begin with ```#X obj```
* Followed by, in order:
	* Origin X (single integer)
	* Origin Y (single integer)
	* ```vsl``` (vertical) or ```hsl``` (horizontal)
	* Width (single integer)
	* Height (single integer)
	* Minimum value (single number)
	* Maximum value (single number)
	* Logarithmic scaling (single integer; 0: off, 1: on)
	* Init (single integer; 0: off, 1: on)
	* Send symbol (string)
	* Receive symbol (string)
	* Label text (string)
	* Label X offset (single integer)
	* Label Y offset (single integer)
	* Font index (single integer; 0: DejaVu Sans Mono, 1: Helvetica, 2: Times)
	* Font size (single integer)
	* Background color (single integer; ```-```-prefixed)
	* Foreground color (single integer; ```-```-prefixed)
	* Label text color (single integer; ```-```-prefixed)
	* Current value/position (single integer; 0 to 12700)
	* Steady on click (single integer; 0: off, 1: on)

##### Radio Buttons
* Begin with ```#X obj```
* Followed by, in order:
	* Origin X (single integer)
	* Origin Y (single integer)
	* ```vradio``` (vertical) or ```hradio``` (horizontal)
	* Size (single integer)
	* ```0``` (single integer, possibly unused)
	* Init (single integer; 0: off, 1: on)
	* Number of cells (single integer)
	* Send symbol (string)
	* Receive symbol (string)
	* Label text (string)
	* Label X offset (single integer)
	* Label Y offset (single integer)
	* Font index (single integer; 0: DejaVu Sans Mono, 1: Helvetica, 2: Times)
	* Font size (single integer)
	* Background color (single integer; ```-```-prefixed)
	* Foreground color (single integer; ```-```-prefixed)
	* Label text color (single integer; ```-```-prefixed)
	* Current value (single integer)

##### VU
* Begins with ```#X obj```
* Followed by, in order:
	* Origin X (single integer)
	* Origin Y (single integer)
	* ```vu``` (string)
	* Width (single integer)
	* Height (single integer)
	* Receive symbol (string)
	* Label text (string)
	* Label X offset (single integer)
	* Label Y offset (single integer)
	* Font index (single integer; 0: DejaVu Sans Mono, 1: Helvetica, 2: Times)
	* Font size (single integer)
	* Background color (single integer; ```-```-prefixed)
	* Label text color (single integer; ```-```-prefixed)
	* Scale visibility (single integer; 0: off, 1: on)
	* ```0``` (single integer, possibly unused)

##### Canvas Rectangles
* Begin with ```#X obj```
* Followed by, in order:
	* Origin X (single integer)
	* Origin Y (single integer)
	* ```cnv``` (string)
	* Selectable size (single integer)
	* Width (single integer)
	* Height (single integer)
	* Send symbol (string)
	* Receive symbol (string)
	* Label text (string)
	* Label X offset (single integer)
	* Label Y offset (single integer)
	* Font index (single integer; 0: DejaVu Sans Mono, 1: Helvetica, 2: Times)
	* Font size (single integer)
	* Background color (single integer; ```-```-prefixed)
	* Label text color (single integer; ```-```-prefixed)
	* ```0``` (single integer, possibly unused)

##### Graphical Arrays
* Begin with ```#X array```
* Followed by, in order:
	* Array name (string)
	* Array size (single integer)
	* Array data type (string; defaults to ```float```)
	* The sum total (single integer) of: 
		* Graph draw mode (0: polygon, 2: points, 4: Bezier curve, 6: Bar graph)
		* Save contents (0: off, 1: on)
		* Jump on click (0: off, 16: on)
	* Fill color (six-digit hexadecimal; ```#```-prefixed; applies to bar graph)
	* Outline color (six-digit hexadecimal; ```#```-prefixed)
* If save contents is enabled, this is typically followed with the array data beginning with ```#A``` followed by:
	* A single integer N that appears to be able to zero the first N elements of the array
	* A sequence of numbers representing the elements of the array
* N.B. for GArrays, this information is typically wrapped in ```#N canvas``` and ```#X restore``` subpatch statements.
