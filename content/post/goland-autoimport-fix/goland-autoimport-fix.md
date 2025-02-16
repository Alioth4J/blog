---
title: Fixing Import Block Vanishment While Coding in Goland
description: Goland configuration guide
date: 2025-02-16
slug: goland-autoimport-fix
image: 
categories:
    - Go
---

## Trouble
Pressing `Enter` after typing `import (` in Goland IDE will lead to an immediate vanishment of the import block.  

<video controls src="goland-import-trouble.webm"></video>

## Why Does the Import Block Vanish?
Goland's default settings trigger automated format rules.  

Specifically:  
After pressing `Enter`, the formatting function was triggered, and it removes the import block.  

## Fix
Settings -> Go -> Imports -> **Disable** "Optimize imports on the fly"  

## Happy Coding!

