---
layout: page
title: Setup Instructions
excerpt: Step-by-step instructions for creating modified update packages.
permalink: /setup-instructions/
hide: true
---

These instructions are general and apply to most patch from this website.
Games used for the presentation does not have any relevance for the steps.

* TOC
{:toc}

# Creating custom update package with modified files

## 1. Dump your game using app dumper

-  I recommend [Modded Warfare](https://youtu.be/fo_fnPvOz74) guide on this.

-  If you got NP-32046-5 error when attempting to launch your game. Delete trophy data in sce_sys/trophy/trophy00.trp and rebuild your game. (Be warned as this will erase trophy data from your game)

## 2. Copy the necessary files needed to build the package

-  sce_sys folder

-  eboot.bin file

-  (Any additional files that is mentioned in patch markdown file)

## 3. Create a new patch folder

-  CUSAxxxxx being your CUSA titleID from your newly dumped game. (i.e CUSA00001-patch)

## 4. Open file using a hex editor

-   Note: Some patch may use a different file.

<p align="center">
<img src="{% link assets/images/setup/hxd0.png %}">
</p>
<div align="center">
<em>Open Search.</em>
</div>

<p align="center">
<img src="{% link assets/images/setup/hxd1.png %}">
<em>Paste copied first line into search field and change search to Hex Value as well as change direction to all and click 'Search All'.</em>
</p>

<p align="center">
<img src="{% link assets/images/setup/hxd2.png %}">
<em>Copy second line from patch file and Paste Write.</em>
</p>

-  Save your changes.

## 5. Open param.sfo

<p align="center">
<img src="{% link assets/images/setup/hxd4.png %}">
<em>Change gd to gp and 01.00 to 01.01</em>
</p>

-  Save your changes.
 
## 6. Genarate a new GP4 for your patch folder

<p align="center">
<img src="{% link assets/images/setup/gp4-0.png %}">
</p>

## 7. Build your package file and install it on the console

-  If your base package filename does not match, Be sure to rename it to what is listed in GP4 file. (i.e `app_path="UP4511-CUSA01127_00-PPPPPPPPTTTTTTTT-A0100-V0100.pkg"`)

-  If you have any further issues building the update, refer to this section of [Modded Warfare Video](https://youtu.be/fo_fnPvOz74?t=822) on building updates.
