<!-- source_url: https://docs.lvgl.io/master/index.html -->
LVGL 9.6 documentation
Contents
Menu
Expand
Light mode
Dark mode
System, in dark mode
System, in light mode
View documentation source on GitHub
Skip to content
Select LVGL version
Introduction
Requirements
License
FAQ
The LVGL Repository
Getting started
Learn the Basics
Basic Examples
What's Next?
Examples
Integration
Overview
Running on PC
Linux
Windows
macOS
Browser
SDL Driver
UEFI
Embedded Linux Support
OpenGL Overview
OpenGL ES Draw Unit
SDL Draw Unit
NanoVG Draw Unit
OS Support
Buildroot
Quick Setup
RPi4 custom image
LVGL application
Yocto
Yocto Project Core Components
LVGL in Yocto
Yocto Project Terms
Torizon OS
Drivers
Linux Framebuffer Driver
DRM
OpenGL
GLFW
EGL
Wayland Display/Inputs driver
X11 Display/Inputs driver
evdev
libinput
RTOS Support
FreeRTOS
MQX RTOS
NuttX RTOS
PX5 RTOS
QNX
RT-Thread RTOS
Zephyr
Framework Support
Arduino
PlatformIO
Tasmota and berry
Board Support
LVGL Supported
Partner Supported
Boards
ICOP
Toradex
Riverdi
Viewe
Chip Vendor Support
Alif
Overview
Dave2D GPU
Arm
Overview
Arm-2D GPU
Espressif
Overview
Add LVGL to an ESP32 IDF project
2D Direct Memory Access (DMA2D) Support
PPA (Pixel Processing Accelerator) Support
Tips and tricks
NXP
Overview
NXP eLCDIF
NXP PXP GPU
VG-Lite General GPU
NXP G2D GPU
Renesas
Built-in Drivers
RA Family
RX Family
RZ/G Family
RZ/A Family
Supported Boards
Renesas GLCDC
Dave2D GPU
STM32
Overview
Use LVGL in an STM32 HAL Project
STM32 LTDC Display Driver
NeoChrom
STM32 DMA2D
SPI Display Driver Creation for STM32
Display Controller Support
EVE
EVE (FT81x)
EVE External GPU Renderer
Generic MIPI DCS-Compatible LCD Controller Driver
ILI9341 LCD Controller Driver
ST7735 LCD Controller Driver
ST7789 LCD Controller Driver
ST7796 LCD Controller Driver
NV3007 LCD Controller Driver
Build System Support
make
CMake
Bindings
Output API as JSON Data
Cpp
JavaScript
MicroPython
PikaScript
Main Modules
Display (lv_display)
Overview
Setting Up Your Display(s)
Screen Layers
Color Format
Refreshing
Display Events
Changing Resolution
Inactivity Measurement
Rotation
Constraints on Redrawn Area
Tiled Rendering
Extending/Combining Displays
Input devices (lv_indev)
Input devices
Touchpad and Mouse
Keypad and Keyboard
Encoder
Hardware Button
Groups
Gestures
Grid navigation
Fonts (lv_font)
Overview
Built-In Fonts
BinFont Loader
Tiny TTF Font Engine
FreeType Font Engine
Image font
BDF Font
Bidirectional Support
Adding a New Font Engine
Font Manager
Images
Overview
Image Sources
Color Formats
Adding Images to Your Project
Using Images
Image Decoders
Image Caching
Color (lv_color)
Timer (lv_timer)
Animation (lv_anim)
File System (lv_fs_drv)
Observer
How to Use
Examples
Drawing
Draw Pipeline
Draw API
Draw Layers
Draw Descriptors
Draw Units
Snapshot
Translation
Common Widget Features
Overview
API Conventions
Widget Tree
Screens
Position and Size
Parts and States
Layers
Styles
Styles Overview
Style Sheets
Local Styles
Transitions
Themes
Style Properties
Events
Flags
Layouts
Overview
Flex
Grid
Scrolling
Widget Properties
All Widgets
Base Widget (lv_obj)
3D Texture (lv_3dtexture)
Animation Image (lv_animimg)
Arc (lv_arc)
Arc Label (lv_arclabel)
Bar (lv_bar)
Button (lv_button)
Button Matrix (lv_buttonmatrix)
Calendar (lv_calendar)
Canvas (lv_canvas)
Chart (lv_chart)
Checkbox (lv_checkbox)
Drop-Down List (lv_dropdown)
GIF (lv_gif)
Image (lv_image)
Image Button (lv_imagebutton)
Pinyin IME
Keyboard (lv_keyboard)
Label (lv_label)
LED (lv_led)
Line (lv_line)
List (lv_list)
Lottie (lv_lottie)
Menu (lv_menu)
Message Box (lv_msgbox)
Roller (lv_roller)
Scale (lv_scale)
Slider (lv_slider)
Spangroup (lv_spangroup)
Spinbox (lv_spinbox)
Spinner (lv_spinner)
Switch (lv_switch)
Table (lv_table)
Tab View (lv_tabview)
Text Area (lv_textarea)
Tile View (lv_tileview)
Window (lv_win)
New Widget
LVGL Pro and XML
Introduction
Learn by Examples
Editor
Overview
Installation
User Interface
Hotkeys
License
XML Overview
Overview
Syntax
XML License
Integration
Using the Exported C Code
Loading XML Files at Runtime
Renesas
Arduino IDE
Zephyr RTOS
UI Elements
Components
Widgets
Screens
Animations
API
Constants
Events in XML
Preview
Styles
View
Assets
Images
Fonts
Features
Data binding (Subjects)
UI Testing
Language Translations
Tools
CLI
Online Share
Figma integration
LVGL Pro Best Practices
Auxiliary Modules
File Explorer
Fragment
3rd-Party Libraries
Font Support
FreeType Font Engine
Tiny TTF Font Engine
File System Support
File System Interfaces
Arduino ESP littlefs
Arduino SD
FrogFS
littlefs
Image Support
BMP Decoder
GIF (lv_gif)
libjpeg-turbo Decoder
libpng Decoder
LodePNG Decoder
WebP Decoder
LZ4 Decompression
RLE Decompression
Rlottie Player
SVG Decoder
Tiny JPEG Decoder
Video Support
FFmpeg Support
GStreamer
Barcode
glTF
QR Code
Debugging
GDB Plug-In
Logging
Monkey
Widget ID
Profiler
System Monitor (sysmon)
UI Testing
VG-Lite GPU Simulator
Guides
How-To Articles
Internal Subsystems
Contributing
Introduction
Ways to Contribute
Pull Requests
Developer Certification of Origin (DCO)
Coding Style
Assertions and Argument Checking
Change Log
API
lv_api_map_v8.h
lv_api_map_v9_0.h
lv_api_map_v9_1.h
lv_api_map_v9_2.h
lv_api_map_v9_3.h
lv_api_map_v9_4.h
lv_api_map_v9_5.h
lv_conf.h
lv_conf_kconfig.h
lv_init.h
lvgl.h
lvgl_private.h
core
lv_global.h
lv_group.h
lv_group_private.h
lv_obj.h
lv_obj_class.h
lv_obj_class_private.h
lv_obj_draw.h
lv_obj_draw_private.h
lv_obj_event.h
lv_obj_event_private.h
lv_obj_pos.h
lv_obj_private.h
lv_obj_scroll.h
lv_obj_scroll_private.h
lv_obj_style.h
lv_obj_style_gen.h
lv_obj_style_private.h
lv_obj_tree.h
lv_observer.h
lv_observer_private.h
lv_refr.h
lv_refr_private.h
debugging
monkey
lv_monkey.h
lv_monkey_private.h
sysmon
lv_sysmon.h
lv_sysmon_private.h
test
lv_test.h
lv_test_display.h
lv_test_fs.h
lv_test_helpers.h
lv_test_indev.h
lv_test_indev_gesture.h
lv_test_private.h
lv_test_screenshot_compare.h
display
lv_display.h
lv_display_private.h
draw
lv_draw.h
lv_draw_3d.h
lv_draw_arc.h
lv_draw_blur.h
lv_draw_buf.h
lv_draw_buf_private.h
lv_draw_image.h
lv_draw_image_private.h
lv_draw_label.h
lv_draw_label_private.h
lv_draw_line.h
lv_draw_mask.h
lv_draw_private.h
lv_draw_rect.h
lv_draw_rect_private.h
lv_draw_triangle.h
lv_draw_triangle_private.h
lv_draw_vector.h
lv_draw_vector_private.h
lv_image_decoder.h
lv_image_decoder_private.h
lv_image_dsc.h
convert
lv_draw_buf_convert.h
helium
lv_draw_buf_convert_helium.h
neon
lv_draw_buf_convert_neon.h
dma2d
lv_draw_dma2d.h
lv_draw_dma2d_private.h
espressif
ppa
lv_draw_ppa.h
lv_draw_ppa_private.h
eve
lv_draw_eve.h
lv_draw_eve_private.h
lv_draw_eve_ram_g.h
lv_draw_eve_target.h
lv_eve.h
nanovg
lv_draw_nanovg.h
lv_draw_nanovg_private.h
lv_nanovg_fbo_cache.h
lv_nanovg_image_cache.h
lv_nanovg_math.h
lv_nanovg_utils.h
nema_gfx
lv_draw_nema_gfx.h
lv_draw_nema_gfx_utils.h
lv_nema_gfx_path.h
nxp
g2d
lv_draw_g2d.h
lv_g2d_buf_map.h
lv_g2d_utils.h
pxp
lv_draw_pxp.h
lv_pxp_cfg.h
lv_pxp_osa.h
lv_pxp_utils.h
opengles
lv_draw_opengles.h
renesas
dave2d
lv_draw_dave2d.h
lv_draw_dave2d_utils.h
sdl
lv_draw_sdl.h
snapshot
lv_snapshot.h
sw
lv_draw_sw.h
lv_draw_sw_grad.h
lv_draw_sw_mask.h
lv_draw_sw_mask_private.h
lv_draw_sw_private.h
lv_draw_sw_utils.h
arm2d
lv_draw_sw_arm2d.h
lv_draw_sw_helium.h
blend
lv_draw_sw_blend.h
lv_draw_sw_blend_private.h
lv_draw_sw_blend_to_a8.h
lv_draw_sw_blend_to_a88.h
lv_draw_sw_blend_to_argb8888.h
lv_draw_sw_blend_to_argb8888_premultiplied.h
lv_draw_sw_blend_to_i1.h
lv_draw_sw_blend_to_l8.h
lv_draw_sw_blend_to_rgb565.h
lv_draw_sw_blend_to_rgb565_swapped.h
lv_draw_sw_blend_to_rgb888.h
arm2d
lv_blend_arm2d.h
helium
lv_blend_helium.h
neon
lv_blend_neon.h
lv_draw_sw_blend_neon_to_rgb565.h
lv_draw_sw_blend_neon_to_rgb888.h
riscv_v
lv_blend_riscv_v.h
lv_blend_riscv_v_private.h
lv_blend_riscv_vector_emulation.h
lv_draw_sw_blend_riscv_v_to_rgb888.h
vg_lite
lv_draw_vg_lite.h
lv_draw_vg_lite_type.h
lv_vg_lite_bitmap_font_cache.h
lv_vg_lite_decoder.h
lv_vg_lite_grad.h
lv_vg_lite_math.h
lv_vg_lite_path.h
lv_vg_lite_pending.h
lv_vg_lite_stroke.h
lv_vg_lite_utils.h
drivers
lv_drivers.h
display
drm
lv_linux_drm.h
lv_linux_drm_egl_private.h
fb
lv_linux_fbdev.h
ft81x
lv_ft81x.h
lv_ft81x_defines.h
ili9341
lv_ili9341.h
lcd
lv_lcd_generic_mipi.h
lovyan_gfx
lv_lgfx_user.hpp
lv_lovyan_gfx.h
nv3007
lv_nv3007.h
nxp_elcdif
lv_nxp_elcdif.h
renesas_glcdc
lv_renesas_glcdc.h
st7735
lv_st7735.h
st7789
lv_st7789.h
st7796
lv_st7796.h
st_ltdc
lv_st_ltdc.h
tft_espi
lv_tft_espi.h
draw
eve
lv_draw_eve_display.h
lv_draw_eve_display_defines.h
evdev
lv_evdev.h
lv_evdev_private.h
libinput
lv_libinput.h
lv_libinput_private.h
lv_xkb.h
lv_xkb_private.h
nuttx
lv_nuttx_cache.h
lv_nuttx_entry.h
lv_nuttx_fbdev.h
lv_nuttx_image_cache.h
lv_nuttx_lcd.h
lv_nuttx_libuv.h
lv_nuttx_mouse.h
lv_nuttx_profiler.h
lv_nuttx_touchscreen.h
opengles
lv_opengles_debug.h
lv_opengles_driver.h
lv_opengles_egl.h
lv_opengles_egl_private.h
lv_opengles_glfw.h
lv_opengles_private.h
lv_opengles_texture.h
lv_opengles_texture_private.h
lv_opengles_window.h
assets
lv_opengles_shader.h
opengl_shader
lv_opengl_shader_internal.h
qnx
lv_qnx.h
sdl
lv_sdl_keyboard.h
lv_sdl_mouse.h
lv_sdl_mousewheel.h
lv_sdl_private.h
lv_sdl_window.h
uefi
lv_uefi.h
lv_uefi_context.h
lv_uefi_display.h
lv_uefi_edk2.h
lv_uefi_gnu_efi.h
lv_uefi_indev.h
lv_uefi_private.h
lv_uefi_std_wrapper.h
wayland
lv_wayland.h
lv_wayland_backend_private.h
lv_wayland_keyboard.h
lv_wayland_pointer.h
lv_wayland_pointer_axis.h
lv_wayland_private.h
lv_wayland_touch.h
lv_wayland_window.h
windows
lv_windows_context.h
lv_windows_display.h
lv_windows_input.h
lv_windows_input_private.h
x11
lv_x11.h
font
lv_font.h
lv_font_private.h
lv_symbol_def.h
binfont_loader
lv_binfont_loader.h
fmt_txt
lv_font_fmt_txt.h
lv_font_fmt_txt_private.h
font_manager
lv_font_manager.h
lv_font_manager_recycle.h
imgfont
lv_imgfont.h
indev
lv_gridnav.h
lv_indev.h
lv_indev_gesture.h
lv_indev_gesture_private.h
lv_indev_private.h
lv_indev_scroll.h
layouts
lv_layout.h
lv_layout_private.h
flex
lv_flex.h
grid
lv_grid.h
libs
barcode
lv_barcode.h
lv_barcode_private.h
bin_decoder
lv_bin_decoder.h
bmp
lv_bmp.h
ffmpeg
lv_ffmpeg.h
lv_ffmpeg_private.h
freetype
lv_freetype.h
lv_freetype_private.h
fsdrv
lv_fsdrv.h
gltf
gltf_data
lv_gltf_data_internal.h
lv_gltf_data_internal.hpp
lv_gltf_model.h
lv_gltf_model_loader.h
lv_gltf_model_node.h
gltf_environment
lv_gltf_environment.h
lv_gltf_environment_private.h
gltf_view
lv_gltf.h
lv_gltf_view_internal.h
assets
lv_gltf_view_shader.h
math
lv_3dmath.h
lv_gltf_math.hpp
gstreamer
lv_gstreamer.h
lv_gstreamer_internal.h
libjpeg_turbo
lv_libjpeg_turbo.h
libpng
lv_libpng.h
libwebp
lv_libwebp.h
lodepng
lv_lodepng.h
qrcode
lv_qrcode.h
lv_qrcode_private.h
rle
lv_rle.h
rlottie
lv_rlottie.h
lv_rlottie_private.h
svg
lv_svg.h
lv_svg_decoder.h
lv_svg_parser.h
lv_svg_render.h
lv_svg_token.h
tiny_ttf
lv_tiny_ttf.h
tjpgd
lv_tjpgd.h
misc
lv_anim.h
lv_anim_private.h
lv_anim_timeline.h
lv_anim_timeline_private.h
lv_area.h
lv_area_private.h
lv_array.h
lv_assert.h
lv_async.h
lv_bidi.h
lv_bidi_private.h
lv_check_arg.h
lv_circle_buf.h
lv_color.h
lv_color_op.h
lv_color_op_private.h
lv_event.h
lv_event_private.h
lv_ext_data.h
lv_fs.h
lv_fs_private.h
lv_grad.h
lv_iter.h
lv_ll.h
lv_log.h
lv_lru.h
lv_math.h
lv_matrix.h
lv_palette.h
lv_pending.h
lv_profiler.h
lv_profiler_builtin.h
lv_profiler_builtin_private.h
lv_rb.h
lv_rb_private.h
lv_style.h
lv_style_gen.h
lv_style_private.h
lv_templ.h
lv_text.h
lv_text_ap.h
lv_text_private.h
lv_timer.h
lv_timer_private.h
lv_tree.h
lv_utils.h
cache
lv_cache.h
lv_cache_entry.h
lv_cache_entry_private.h
lv_cache_private.h
class
lv_cache_class.h
lv_cache_lru_ll.h
lv_cache_lru_rb.h
lv_cache_sc_da.h
instance
lv_cache_instance.h
lv_image_cache.h
lv_image_header_cache.h
osal
lv_linux.h
lv_os.h
lv_os_none.h
lv_os_private.h
others
file_explorer
lv_file_explorer.h
lv_file_explorer_private.h
fragment
lv_fragment.h
lv_fragment_private.h
translation
lv_translation.h
lv_translation_private.h
stdlib
lv_mem.h
lv_mem_private.h
lv_sprintf.h
lv_string.h
builtin
lv_tlsf.h
lv_tlsf_private.h
themes
lv_theme.h
lv_theme_private.h
default
lv_theme_default.h
mono
lv_theme_mono.h
simple
lv_theme_simple.h
tick
lv_tick.h
lv_tick_private.h
widgets
3dtexture
lv_3dtexture.h
lv_3dtexture_private.h
animimage
lv_animimage.h
lv_animimage_private.h
arc
lv_arc.h
lv_arc_private.h
arclabel
lv_arclabel.h
lv_arclabel_private.h
bar
lv_bar.h
lv_bar_private.h
button
lv_button.h
lv_button_private.h
buttonmatrix
lv_buttonmatrix.h
lv_buttonmatrix_private.h
calendar
lv_calendar.h
lv_calendar_chinese.h
lv_calendar_header_arrow.h
lv_calendar_header_dropdown.h
lv_calendar_private.h
canvas
lv_canvas.h
lv_canvas_private.h
chart
lv_chart.h
lv_chart_private.h
checkbox
lv_checkbox.h
lv_checkbox_private.h
dropdown
lv_dropdown.h
lv_dropdown_private.h
gif
lv_gif.h
image
lv_image.h
lv_image_private.h
imagebutton
lv_imagebutton.h
lv_imagebutton_private.h
ime
lv_ime_pinyin.h
lv_ime_pinyin_private.h
keyboard
lv_keyboard.h
lv_keyboard_private.h
label
lv_label.h
lv_label_private.h
led
lv_led.h
lv_led_private.h
line
lv_line.h
lv_line_private.h
list
lv_list.h
lottie
lv_lottie.h
lv_lottie_private.h
menu
lv_menu.h
lv_menu_private.h
msgbox
lv_msgbox.h
lv_msgbox_private.h
objx_templ
lv_objx_templ.h
property
lv_obj_property_names.h
lv_style_properties.h
roller
lv_roller.h
lv_roller_private.h
scale
lv_scale.h
lv_scale_private.h
slider
lv_slider.h
lv_slider_private.h
span
lv_span.h
lv_span_private.h
spinbox
lv_spinbox.h
lv_spinbox_private.h
spinner
lv_spinner.h
lv_spinner_private.h
switch
lv_switch.h
lv_switch_private.h
table
lv_table.h
lv_table_private.h
tabview
lv_tabview.h
lv_tabview_private.h
textarea
lv_textarea.h
lv_textarea_private.h
tileview
lv_tileview.h
lv_tileview_private.h
win
lv_win.h
lv_win_private.h
Back to top
View this page
中⽂
Chinese Translation
Welcome to LVGL Docs
¶
Create beautiful UIs for any MCU, MPU and display type.
Introduction
Get familiar with LVGL.
Getting Started
Learn how LVGL works.
Going deeper
Get your feet wet with LVGL.
Integrating LVGL into Your Project
Learn how to add LVGL to your project for any platform, framework and display type.
Widgets
Learn to use LVGL Widgets with examples.
Contributing
Be part of LVGL's development.
Introduction
¶
Introduction
Getting started
Details
¶
Examples
Integration
Main Modules
Common Widget Features
All Widgets
LVGL Pro and XML
Auxiliary Modules
3rd-Party Libraries
Debugging
Guides
Appendix
¶
Contributing
Change Log
API