<!-- source_url: https://github.com/lvgl/lvgl -->
GitHub - lvgl/lvgl: Embedded graphics library to create beautiful UIs for any MCU, MPU and display type. · GitHub
Skip to content
Search or jump to...
Search code, repositories, users, issues, pull requests...
Search
Clear
Search syntax tips
Provide feedback
We read every piece of feedback, and take your input very seriously.
Include my email address so I can be contacted
Cancel
Submit feedback
Saved searches
Use saved searches to filter your results more quickly
Name
Query
To see all available qualifiers, see our
documentation
.
Cancel
Create saved search
Sign in
Sign up
Appearance settings
Resetting focus
You signed in with another tab or window.
Reload
to refresh your session.
You signed out in another tab or window.
Reload
to refresh your session.
You switched accounts on another tab or window.
Reload
to refresh your session.
Dismiss alert
{{ message }}
lvgl
/
lvgl
Public
Uh oh!
There was an error while loading.
Please reload this page
.
Notifications
You must be signed in to change notification settings
Fork
4.1k
Star
23.3k
master
Branches
Tags
Go to file
Code
Open more actions menu
Folders and files
Name
Name
Last commit message
Last commit date
Latest commit
History
12,441 Commits
12,441 Commits
.devcontainer
.devcontainer
.github
.github
configs
configs
demos
demos
docs
docs
env_support
env_support
examples
examples
libs/
nema_gfx
libs/
nema_gfx
scripts
scripts
src
src
tests
tests
zephyr
zephyr
.gitignore
.gitignore
.pre-commit-config.yaml
.pre-commit-config.yaml
.typos.toml
.typos.toml
CMakeLists.txt
CMakeLists.txt
CMakePresets.json
CMakePresets.json
COPYRIGHTS.md
COPYRIGHTS.md
Kconfig
Kconfig
LICENCE.txt
LICENCE.txt
README.md
README.md
SConscript
SConscript
component.mk
component.mk
idf_component.yml
idf_component.yml
library.json
library.json
library.properties
library.properties
lv_conf_template.h
lv_conf_template.h
lv_version.h
lv_version.h
lv_version.h.in
lv_version.h.in
lvgl.h
lvgl.h
lvgl.mk
lvgl.mk
lvgl.pc.in
lvgl.pc.in
lvgl_private.h
lvgl_private.h
View all files
Repository files navigation
English
|
中文
|
Português do Brasil
|
日本語
|
עברית
Light and Versatile Graphics Library
Website
|
LVGL Pro Editor
|
Docs
|
Forum
|
Demos
|
Services
Table of Contents
Overview
Features
Platform Support
LVGL Pro Editor
Commercial Services
Integrating LVGL
Examples
Contributing
📒 Overview
LVGL
is a free and open-source UI library that enables you to create graphical user interfaces
for any MCUs and MPUs from any vendor on any platform.
Requirements
: LVGL has no external dependencies, which makes it easy to compile for any modern target,
from small MCUs to multi-core Linux-based MPUs with 3D support. For a simple UI, you need only ~100kB RAM,
~200–300kB flash, and a buffer size of 1/10 of the screen for rendering.
To get started
, pick a ready-to-use VSCode, Eclipse, or any other project and try out LVGL
on your PC. The LVGL UI code is fully platform-independent, so you can use the same UI code
on embedded targets too.
LVGL Pro
is a complete toolkit to help you build, test, share, and ship UIs faster.
It comes with an XML Editor where you can quickly create and test reusable components,
export C code, or load the XMLs at runtime. Learn more here.
💡 Features
Free and Portable
A fully portable C (C++ compatible) library with no external dependencies.
Can be compiled for any MCU or MPU, with any (RT)OS. Make, CMake, and simple globbing are all supported.
Supports monochrome, ePaper, OLED, or TFT displays, or even monitors.
Displays
Distributed under the MIT license, so you can easily use it in commercial projects too.
Needs only 32kB RAM and 128kB Flash, a frame buffer, and at least a 1/10 screen-sized buffer for rendering.
OS, external memory, and GPU are supported but not required.
Widgets, Styles, Layouts, and More
30+ built-in
Widgets
: Button, Label, Slider, Chart, Keyboard, Meter, Arc, Table, and many more.
Flexible
Style system
with ~100 style properties to customize any part of the widgets in any state.
Flexbox
and
Grid
-like layout engines to automatically size and position the widgets responsively.
Text is rendered with UTF-8 encoding, supporting CJK, Thai, Hindi, Arabic, and Persian writing systems.
Data bindings
to easily connect the UI with the application.
Rendering engine supports animations, anti-aliasing, opacity, smooth scrolling, shadows, image transformation, etc.
Powerful 3D rendering engine
to show
glTF models
with OpenGL.
Supports Mouse, Touchpad, Keypad, Keyboard, External buttons, Encoder
Input devices
.
Multiple display
support.
📦️ Platform Support
LVGL has no external dependencies, so it can be easily compiled for any devices and it's  also available in many package managers and RTOSes:
Arduino library
PlatformIO package
Zephyr library
ESP-IDF(ESP32) component
NXP MCUXpresso component
NuttX library
RT-Thread RTOS
CMSIS-Pack
RIOT OS package
🚀 LVGL Pro Editor
LVGL Pro is a complete toolkit to build, test, share, and ship embedded UIs efficiently.
It consists of four tightly related tools:
XML Editor
: The heart of LVGL Pro. A desktop app to build components and screens in XML, manage data bindings, translations, animations, tests, and more. Learn more about the
XML Format
and the
Editor
.
Online Viewer
: Run the Editor in your browser, open GitHub projects, and share easily without setting up a developer environment. Visit
https://viewer.lvgl.io
.
CLI Tool
: Generate C code and run tests in CI/CD. See the details
here
.
Figma Plugin
: Sync and extract styles directly from Figma. See how it works
here
.
Together, these tools let developers build UIs efficiently, test them reliably, and collaborate with team members and customers.
Learn more at
https://pro.lvgl.io
🤝 Commercial Services
LVGL LLC provides several types of commercial services to help you with UI development. With 15+ years of experience in the user interface and graphics industry, we can help bring your UI to the next level.
Graphics design
: Our in-house graphic designers are experts in creating beautiful modern designs that fit your product and the capabilities of your hardware.
UI implementation
: We can implement your UI based on the design you or we have created. You can be sure that we will make the most of your hardware and LVGL. If a feature or widget is missing from LVGL, don't worry, we will implement it for you.
Consulting and Support
: We also offer consulting to help you avoid costly and time-consuming mistakes during UI development.
Board certification
: For companies offering development boards or production-ready kits, we provide board certification to show how the board can run LVGL.
Check out our
Demos
as references. For more information, take a look at the
Services page
.
Contact us
and tell us how we can help.
🧑‍💻 Integrating LVGL
Integrating LVGL is very simple. Just drop it into any project and compile it as you would compile other files.
To configure LVGL, copy
lv_conf_template.h
as
lv_conf.h
, enable the first
#if 0
, and adjust the configs as needed.
(The default config is usually fine.) If available, LVGL can also be used with Kconfig.
Once in the project, you can initialize LVGL and create display and input devices as follows:
#include
\"lvgl/lvgl.h\"
/*Define LV_LVGL_H_INCLUDE_SIMPLE to include as \"lvgl.h\"*/
#define
TFT_HOR_RES
320
#define
TFT_VER_RES
240
static
uint32_t
my_tick_cb
(
void
)
{
return
my_get_millisec
();
}
static
void
my_flush_cb
(
lv_display_t
*
disp
,
const
lv_area_t
*
area
,
uint8_t
*
px_map
)
{
/*Write px_map to the area->x1, area->x2, area->y1, area->y2 area of the
*frame buffer or external display controller. */
/* signal LVGL that we're done */
lv_display_flush_ready
(
disp
);
}
static
void
my_touch_read_cb
(
lv_indev_t
*
indev
,
lv_indev_data_t
*
data
)
{
if
(
my_touch_is_pressed
()) {
data
->
point
.
x
=
touchpad_x
;
data
->
point
.
y
=
touchpad_y
;
data
->
state
=
LV_INDEV_STATE_PRESSED
;
   }
else
{
data
->
state
=
LV_INDEV_STATE_RELEASED
;
   }
}
void
main
(
void
)
{
my_hardware_init
();
/*Initialize LVGL*/
lv_init
();
/*Set millisecond-based tick source for LVGL so that it can track time.*/
lv_tick_set_cb
(
my_tick_cb
);
/*Create a display where screens and widgets can be added*/
lv_display_t
*
display
=
lv_display_create
(
TFT_HOR_RES
,
TFT_VER_RES
);
/*Add rendering buffers to the screen.
*Here adding a smaller partial buffer assuming 16-bit (RGB565 color format)*/
static
uint8_t
buf
[
TFT_HOR_RES
*
TFT_VER_RES
/
10
*
2
];
/* x2 because of 16-bit color depth */
lv_display_set_buffers
(
display
,
buf
,
NULL
,
sizeof
(
buf
),
LV_DISPLAY_RENDER_MODE_PARTIAL
);
/*Add a callback that can flush the content from `buf` when it has been rendered*/
lv_display_set_flush_cb
(
display
,
my_flush_cb
);
/*Create an input device for touch handling*/
lv_indev_t
*
indev
=
lv_indev_create
();
lv_indev_set_type
(
indev
,
LV_INDEV_TYPE_POINTER
);
lv_indev_set_read_cb
(
indev
,
my_touch_read_cb
);
/*The drivers are in place; now we can create the UI*/
lv_obj_t
*
label
=
lv_label_create
(
lv_screen_active
());
lv_label_set_text
(
label
,
\"Hello world\"
);
lv_obj_center
(
label
);
/*Execute the LVGL-related tasks in a loop*/
while
(
1
) {
lv_timer_handler
();
my_sleep_ms
(
5
);
/*Wait a little to let the system breathe*/
}
}
🤖 Examples
You can check out more than 100 examples at
https://docs.lvgl.io/master/examples.html
The Online Viewer also contains tutorials to easily learn XML:
https://viewer.lvgl.io/
Hello World Button with an Event
C code
static
void
button_clicked_cb
(
lv_event_t
*
e
)
{
printf
(
\"Clicked\\n\"
);
}
[...]
lv_obj_t
*
button
=
lv_button_create
(
lv_screen_active
());
lv_obj_center
(
button
);
lv_obj_add_event_cb
(
button
,
button_clicked_cb
,
LV_EVENT_CLICKED
,
NULL
);
lv_obj_t
*
label
=
lv_label_create
(
button
);
lv_label_set_text
(
label
,
\"Hello from LVGL!\"
);
In XML with LVGL Pro
<
screen
>
\t<
view
>
\t\t<
lv_button
align
=
\"
center
\"
>
\t\t\t<
event_cb
callback
=
\"
button_clicked_cb
\"
/>
\t\t\t<
lv_label
text
=
\"
Hello from LVGL!
\"
/>
\t\t</
lv_button
>
\t</
view
>
</
screen
>
Styled Slider with Data-binding
C code
static
void
my_observer_cb
(
lv_observer_t
*
observer
,
lv_subject_t
*
subject
)
{
printf
(
\"Slider value: %d\\n\"
,
lv_subject_get_int
(
subject
));
}
[...]
static
lv_subject_t
subject_value
;
lv_subject_init_int
(
&
subject_value
,
35
);
lv_subject_add_observer
(
&
subject_value
,
my_observer_cb
,
NULL
);
lv_style_t
style_base
;
lv_style_init
(
&
style_base
);
lv_style_set_bg_color
(
&
style_base
,
lv_color_hex
(
0xff8800
));
lv_style_set_bg_opa
(
&
style_base
,
255
);
lv_style_set_radius
(
&
style_base
,
4
);
lv_obj_t
*
slider
=
lv_slider_create
(
lv_screen_active
());
lv_obj_center
(
slider
);
lv_obj_set_size
(
slider
,
lv_pct
(
80
),
16
);
lv_obj_add_style
(
slider
,
&
style_base
,
LV_PART_INDICATOR
);
lv_obj_add_style
(
slider
,
&
style_base
,
LV_PART_KNOB
);
lv_obj_add_style
(
slider
,
&
style_base
,
0
);
lv_obj_set_style_bg_opa
(
slider
,
LV_OPA_50
,
0
);
lv_obj_set_style_border_width
(
slider
,
3
,
LV_PART_KNOB
);
lv_obj_set_style_border_color
(
slider
,
lv_color_hex3
(
0xfff
),
LV_PART_KNOB
);
lv_slider_bind_value
(
slider
,
&
subject_value
);
lv_obj_t
*
label
=
lv_label_create
(
lv_screen_active
());
lv_obj_align
(
label
,
LV_ALIGN_CENTER
,
0
,
-30
);
lv_label_bind_text
(
label
,
&
subject_value
,
\"Temperature: %d °C\"
);
In XML with LVGL Pro
<
screen
>
\t<
styles
>
\t\t<
style
name
=
\"
style_base
\"
bg_opa
=
\"
100%
\"
bg_color
=
\"
0xff8800
\"
radius
=
\"
4
\"
/>
\t\t<
style
name
=
\"
style_border
\"
border_color
=
\"
0xfff
\"
border_width
=
\"
3
\"
/>
\t</
styles
>
\n\t<
view
>
\t\t<
lv_label
bind_text
=
\"
value
\"
bind_text-fmt
=
\"
Temperature: %d °C
\"
align
=
\"
center
\"
y
=
\"
-30
\"
/>
\t\t<
lv_slider
align
=
\"
center
\"
bind_value
=
\"
value
\"
style_bg_opa
=
\"
30%
\"
>
\t\t\t<
style
name
=
\"
style_base
\"
/>
\t\t\t<
style
name
=
\"
style_base
\"
selector
=
\"
knob
\"
/>
\t\t\t<
style
name
=
\"
style_base
\"
selector
=
\"
indicator
\"
/>
\t\t\t<
style
name
=
\"
style_border
\"
selector
=
\"
knob
\"
/>
\t\t</
lv_slider
>
\t</
view
>
</
screen
>
Checkboxes in a Layout
C code
/*Create a new screen and load it*/
lv_obj_t
*
scr
=
lv_obj_create
(
NULL
);
lv_screen_load
(
scr
);
/*Set a column layout*/
lv_obj_set_flex_flow
(
scr
,
LV_FLEX_FLOW_COLUMN
);
lv_obj_set_flex_align
(
scr
,
LV_FLEX_ALIGN_SPACE_EVENLY
,
/*Vertical alignment*/
LV_FLEX_ALIGN_START
,
/*Horizontal alignment in the track*/
LV_FLEX_ALIGN_CENTER
);
/*Horizontal alignment of the track*/
/*Create 5 checkboxes*/
const
char
*
texts
[
5
]
=
{
\"Input 1\"
,
\"Input 2\"
,
\"Input 3\"
,
\"Output 1\"
,
\"Output 2\"
};
for
(
int
i
=
0
;
i
<
5
;
i
++
) {
lv_obj_t
*
cb
=
lv_checkbox_create
(
scr
);
lv_checkbox_set_text
(
cb
,
texts
[
i
]);
}
/*Change some states*/
lv_obj_add_state
(
lv_obj_get_child
(
scr
,
1
),
LV_STATE_CHECKED
);
lv_obj_add_state
(
lv_obj_get_child
(
scr
,
3
),
LV_STATE_DISABLED
);
In XML with LVGL Pro
<
screen
>
\t<
view
flex_flow=\
\"
column
\"
style_flex_main_place=\
\"
space_evenly
\"
style_flex_cross_place=\
\"
start
\"
style_flex_track_place=\
\"
center
\"
>
\t\t<
lv_checkbox
text
=
\"
Input 1
\"
/>
\t\t<
lv_checkbox
text
=
\"
Input 2
\"
/>
\t\t<
lv_checkbox
text
=
\"
Input 3
\"
checked
=
\"
true
\"
/>
\t\t<
lv_checkbox
text
=
\"
Output 1
\"
/>
\t\t<
lv_checkbox
text
=
\"
Output 2
\"
disabled
=
\"
true
\"
/>
   </
view
>
</
screen
>
🌟 Contributing
LVGL is an open project, and contributions are very welcome. There are many ways to contribute, from simply speaking about your project, writing examples, improving the documentation, fixing bugs, or even hosting your own project under the LVGL organization.
For a detailed description of contribution opportunities, visit the
Contributing
section of the documentation.
More than 600 people have already left their fingerprint on LVGL. Be one of them! See you here! 🙂
... and many more.
About
Embedded graphics library to create beautiful UIs for any MCU, MPU and display type.
lvgl.io
Topics
c
gui
microcontroller
embedded
graphics
tft
mcu
Resources
Readme
License
MIT license
Code of conduct
Code of conduct
Uh oh!
There was an error while loading.
Please reload this page
.
Activity
Custom properties
Stars
23.3k
stars
Watchers
315
watching
Forks
4.1k
forks
Report repository
Releases
70
Release v9.5.0
Latest
Feb 18, 2026
+ 69 releases
Sponsor this project
Sponsor
Uh oh!
There was an error while loading.
Please reload this page
.
Learn more about GitHub Sponsors
Uh oh!
There was an error while loading.
Please reload this page
.
Contributors
Uh oh!
There was an error while loading.
Please reload this page
.
Languages
C
89.2%
C++
8.1%
Python
1.8%
HTML
0.4%
CMake
0.2%
Ruby
0.1%
Other
0.2%
You can’t perform that action at this time.