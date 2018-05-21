---
layout: single
title: Webscraping a wiki with Python
categories:
  - programming
tags: 
  - python
---

Its been a little while since my last post. Its been pretty chaotic with my job search lately and what not. Been shot a couple times but I wont let that get me down! Anyways...

## Background

I started a new project called [Bladedrone](https://github.com/kmcgamer/bladedrone), which is an Angular 6 application for players of the free to play, first person shooter called ["Ironsight"](). My goal for this project is to allow users to view statistics on weapons, compare/sort them, as well as create & share custom builds. I am super excited about it and have been working on it almost every day!

## The Problem

In order to be able to sort and compare weapons, we kinda need the data, don't we? There are about 40-ish different weapons in the game with about 7-ish stats per gun. Let me tell ya, I was not about to record each individual stat into the database myself. Luckily, somebody had already done a lot of the work for me. I found a [wiki](http://ironsightgame.wikia.com/wiki/Ironsight_Wiki) for the game, and the author had written down the stats for each weapon! **Awesome!**

The wiki has a page for [all weapons](http://ironsightgame.wikia.com/wiki/Category:Weapon) with links to each individual one. The links then go to another page for each [specific weapon](http://ironsightgame.wikia.com/wiki/AK-12). On each weapon page is a short description of the weapon, a picture of it, its available attachments, and its statistics.

Problem was, there wasn't really a good way for me to retrieve this data. I mean sure, I could have copied and pasted the values into MongoDB manually with a bit of tweaking. And true, that method would have saved me a lot of time to begin with, but it still would have taken me a couple hours to complete. There had to be an even better way.

## The Solution

Let me introduce you to [Scrapy](https://scrapy.org/), a Python framework that allows you to extract data from websites! Scrapy is a web crawler where you can specify how it navigates, extracts, and parses the web and its data. It is super easy to learn, and a lot of fun to use!

With this framework, I was able to tell the script to start at a certain URL, navigate to several links on the page, and extract the data that was on it. Additionally, I was able parse the data as it was extracting into JSON which saved me even more time with importing the data into MongoDB!

## The Script

