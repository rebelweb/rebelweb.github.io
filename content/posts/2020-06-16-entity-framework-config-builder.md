---
title: Entity Framework Config Builder
date: 2020-06-16 11:12 -500
draft: false
---

I work with a legacy Microsoft SQL Server database with many
inconsistencies and large tables. When working on a .NET Core API with
Entity Framework Core, I had to map tables from the database, which 
became a long a tedious process. I know there had to be a better way,
than writing the configuration by hand. This where I worked to write a
small console app to create the config for me.

This app started as a simple ruby script at first. As our team grew I 
converted it over to a .NET Core console app, so It could be used across
our team. The app takes a CSV output of a sp_columns stored procedure 
from SQL Server database, reads it and outputs the config to the console. 

You can check it out here: https://github.com/rebelweb/ConfigBuilder

hen you face a tedious repetitive task, see if you can write a few lines
of code to save hours of time. 