<!-- source_url: https://www.elecrow.com/wiki/Tutorials.html -->
LVGL ESP32 Display Tutorial-A Step-by-Step Guide to LVGL GUI Development - Elecrow Wiki
Skip to content
Initializing search
Shield
Crowtail
Crowbits
Sensors
Wireless
Display
HMI Display
CrowVision
CrowView
Mechanical Keyboard
Elecrow Services
Modules for Prototyping
Accessories
Application Kits
Elecrow Cooperator/Partner
Raspberry Pi
Arduino Kit
Lora
STEAM Education
Tutorials
Tutorials
Back to Community
Tutorials
LVGL, or Light and Versatile Graphics Library, is a powerful open-source graphics library that can be used to develop advanced graphical user interfaces (GUIs) for a wide range of embedded systems. ESP32 is a powerful microcontroller widely used in IoT and embedded systems projects. By combining LVGL with ESP32, you can develop sophisticated and responsive GUI designs that enhance the user experience and add value to your projects, such as support for animation, themes, fonts, and touch input.
In this 16-lesson, step-by-step tutorial, we will guide you through the process of setting up LVGL on your ESP32 development board, creating a basic GUI application, and adding advanced features such as animation, themes, and touch input. We will also cover best practices for optimizing your LVGL GUIs for performance and memory usage.
Whether you are an experienced embedded systems developer or just getting started with LVGL and ESP32, this tutorial will give you the knowledge and skills you need to create stunning GUI features for your ESP32 projects. So let's get started!
Lesson01 Introducing the ESP32 Display series and environment configuration
¶
In the first lesson of this LVGL ESP32 tutorial, we explored the basic concepts and applications of Human Machine Interface (HMI) and introduced the related ESP32 display family. We also can learn how to configure the Arduino environment, a crucial step in developing electronics projects. With these displays and related software environments, we can easily develop various graphical and user interfaces for ESP32 display projects.
Lesson02 Start the ESP32 DISPLAY GUI drawing via LovyanGFX Graphics Library
¶
Starting to draw your first user interface on an ESP32 display is a big step in GUI design. In lesson 2 of this ESP32 GUI tutorial, we'll cover how to use the LovyanGFX library and related functions to draw graphics on a screen, including creating sprites, drawing rectangles, lines, circles, and text, and drawing the sprite to the screen. This lesson also includes sample programs that demonstrate these functions, and you will learn how to extend these functions to draw more complex graphics, such as a small house.
Lesson03 How to Display Pictures on ESP32 Boards
¶
Lesson 3 of the LVGL Tutorial for ESP32 Boards shows how to display images on an RGB screen using an ESP32 development board and an SD card. The key function for displaying bitmap images is the print_img() function. The program initializes the SPI bus, the LCD display, and the SD card, and then loads and displays images from the SD card. The program can be extended to switch between multiple images.
Lesson04 LVGL Basics: How to install LVGL for ESP32 Displays
¶
Lesson 4 of the LVGL ESP32 Tutorial introduces all the basics for beginners to know about LVGL, it provides step-by-step instructions on how to download, install and use SquareLine Studio to create a new project, design the user interface, run and test the application, edit the code and generate code. The LVGL graphics library supports various types of input devices and provides a set of GUI components and graphical effects, and has the advantages of a small memory footprint, easy portability, rich functionality, and a visual GUI design tool that allows rapid creation and design of user interfaces. Primarily used for GUI development in embedded systems, the LVGL graphics library can be used for smart home control panels, IoT device control interfaces, vending machines, electronic scales, and other embedded devices. SquareLine Studio, an IDE based on the LVGL framework, provides a drag-and-drop interface for GUI design and development.
Lesson05 Introduction to the 5 categories of LVGL GUI library Widgets
¶
Lesson 5 discusses the cross-platform GUI library LVGL can support multiple operating systems and platforms, including embedded systems (such as Arduino and Raspberry Pi) and desktop systems (such as Windows and Linux). LVGL's API is simple and easy to use, providing many customizable widgets and features to quickly create beautiful and responsive user interfaces. The graphics library offers advanced features such as animation, easing, gradient, and graphic drawing. In this lesson, we will list five categories of widgets you can use on the open source graphics library LVGL: screen widgets, basic widgets, controller widgets, visualizer widgets, and styles, and provides a detailed introduction to each widget. Developers can refer to this information to configure the widgets when using LVGL.
Lesson06 Use Squareline Studio to start your 1\nst\nhuman machine interface project
¶
To know how to adequately master LVGL GUI design, the first step is to know how to use Squareline Studio. In Lesson 6, we'll discuss a project using Squareline Studio software to create a simple human-machine interface by simply dragging and setting properties in the Squareline Studio platform such as buttons, labels, and panels to teach the beginner the basic use of these widgets. Lesson 6 provides a step-by-step guide to creating the human machine interface and running it on an Arduino. In practice, this LVGL Arduino Human Machine Interface project can be modified and extended as needed, for example by adding background or wallpaper functions.
Lesson07 How to implement text information input with Squareline Studio
¶
Lesson 7 presents a project that illustrates the importance of implementing textual information input in GUI design. The project demonstrates how to use a keyboard widget to allow users to enter text and see the entered content in a text information widget. In addition, the project introduces the image widget of the open source graphics library LVGL. The main goal of this project is to provide a basic understanding of the keyboard, text information, and image widgets in Squareline Studio. Although the project is simple, it can be helpful for beginners to gain practical experience with these widgets, which can be modified and extended as needed, for example by adding a clear button or formatted text functionality.
Lesson08 How to make the menu and a progress bar with Squareline Studio
¶
Lesson 08 presents a project that demonstrates how to create a drop-down menu and a progress bar using Squareline Studio software. The drop-down menu widget allows users to select appropriate options to determine the content, while the slider widget can display appropriate progress. This lesson aims to provide step-by-step instructions on the basic use of these widgets and how they can be modified and extended as needed for LVGL Arduino GUI design.
Lesson09 How to make an analysis report on ESP32 Display
¶
The Rotation Analysis Report project uses Squareline Studio software to create a simple analysis report that uses charts to display different types of data. In addition, this type of project can be further developed to demonstrate data analysis, reports, and trends, providing valuable data support for decision-makers. The Open Source Graphics Library project demonstrates the basic use of chart widgets and how they can be customized for different applications. Lesson 9 provides detailed LVGL GUI design instructions for creating and implementing page jumping functionality for viewing multiple charts.
Lesson10 Create a 3D Printer UI Project on ESP32 Display
¶
The 3D Printer Interface project uses Squareline Studio software to create a user interface that allows users to easily set printer parameters using components such as rollers, switches, text boxes, and buttons. The project demonstrates the basic use of roller and switch components and how they can be customized for different applications. Lesson 10 provides a detailed guide to creating the interface, adding buttons, sliders, and panels, and implementing page jump to move between different screens on the ESP32 display. The project also includes adding temperature and material selection interfaces and a fan switch. Of course, this LVGL ESP32 project can be modified and extended for other purposes.
Lesson11 How to Make a Mixer Interface on ESP32 Display
¶
This lesson shows how to create a mixer interface using Squareline Studio's layout tool and Spinner widgets. It introduces the ARC widget and animation as ways to create dynamic elements and increase visual appeal. The steps shown in the simple project taught in this lesson include adding components, adjusting positions, setting attributes, and adding event settings to achieve the desired effect. Learning this part of the graphics library and practicing it on the ESP32 display helps beginners understand the basic use of the ARC component and animation, and provides a practical application for modifying and extending the Mixer interface.
Lesson12 Make Your ESP32 Display the Lantern Control terminal
¶
Lesson 12 teaches beginners how to use Squareline Studio to create an RGB light control terminal based on an ESP32 display. The project introduces the image widget and its various uses, such as displaying images, and icons, and visualizing data. As the simple project in the lesson suggests, these steps include adding a panel, labels, and image widgets, setting their positions, adding event functions, and controlling the light through the main program. Learning this lesson will help beginners understand the combination of a GUI interface and external sensors, and how to modify generated functions to achieve sensor control through the interface to further utilize the many features of the LVGL open source graphics library.
Lesson13 DIY Electronic Control Terminal on ESP32 Display with Squareline Studio
¶
It's important to build an electronic control panel to measure and monitor the parameters of a whole electronic project, Lesson 13 teaches beginners to resort to Squareline Studio to DIY an ESP32-based electrical control cabinet terminal. The user can control an LED light on/off by clicking the switch button on the ESP32 display project, which uses various functions from the LVGL open source graphics library such as lv_slider_set_value() and lv_img_set_src() and requires a linear potentiometer and a collision switch. The project shown in this lesson includes steps for creating panels, labels, sliders, and image components, adding event functions, and controlling the program from the main program. In addition, the project can be modified and extended to suit individual needs. Overall, this project is useful for beginners to understand the GUI design of external sensor control.
Lesson14 Create Car Control Screen on ESP32 Display: A Step-by-Step Guide
¶
Lesson 14 provides a detailed beginner's guide to designing and implementing a user interface (UI) for the central control screen of a car to display speed, RPM, and other information using a counting sensor and various hardware modules such as buttons and servo modules. The GUI design must take into account user usability and readability. By using the Squareline Studio, the project steps include adding and arranging panels, labels, sliders, and image components, setting their positions, adding event functions, and controlling the car's functions from the main program. The project can be modified and extended according to individual needs.
Lesson15 Smart Agriculture Monitoring: IoT-Based Real-Time ESP32 Display Project
¶
The Smart Agriculture Monitoring Station is an IoT-based agricultural monitoring system that combines the functions of the ESP32 display and the open source LVGL graphics library to monitor and record real-time environmental information such as temperature, humidity, and light intensity of crops. The user interface developed by Squareline Studio can visually display this information in the form of charts and can set value ranges to trigger an alarm when exceeded. The GUI design you can create in this lesson is simple, elegant, and easy to use, and the hardware modules used in the project include digital light sensors, soil moisture sensors, and DHT11 temperature sensors. The Smart Agriculture Monitoring Station project includes steps such as creating panels and labels and adding event functions to control the monitoring program that can help agricultural producers improve productivity and quality.
Lesson16 ESP32 Display for Smart Home Central Control: A Home Automation Solution
¶
The LVGL Smart Home GUI design developed by Squareline Studio in Lesson 16 is a smart home control panel based on a 3.5\" SPI ESP32 display. The panel integrates flame sensors, air quality sensors, temperature and humidity sensors, UV sensors, gesture sensors, and WiFi modules to monitor and control smart home devices. The ESP32 display project is connected to the Internet via WiFi, and users can control smart home devices and view indoor environmental data and device status using smartphones or tablets. Lesson 16 provides a step-by-step guide for beginners to use the open source graphics library LVGL to create panels and labels and add event capabilities to control the monitoring program, which can improve users' quality of life by providing real-time monitoring and control of smart home devices.
Resources
¶
Lesson_code.zip
Made with
Material for MkDocs