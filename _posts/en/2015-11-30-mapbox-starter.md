---
layout: post
title: "Universities in United States"
categories: ['en']
tags: ['Visualization','Design']
published: True
---

Using Mapbox to visualize the distribution of universities in United States, the data is from Linkedin University Rankings and offical admission sites of universities.

Data format is `GeoJSON`, you can modify and add other universities on it if you want. Welcome to fork it on [Github](https://github.com/hectorguo/Universities-in-US/)!

<link href="https://api.tiles.mapbox.com/mapbox.js/v2.2.3/mapbox.css" rel="stylesheet">
<link rel="stylesheet" href="/Universities-in-US/style.min.css">
<div class="map-container">
    <div id="map"></div>
    <nav class="options">
        <input type="checkbox" id="ivy" checked />
        <label for="ivy">
            <span class="ivy" title="常春藤盟校">Ivy League</span>
        </label>
        <h4 title="Linkedin统计排行">Linkedin Rank</h4>
        <ul id="linkedin">
            <li>
                <input type="radio" name="lk" id="lk2" value="dev">
                <label for="lk2"><span title="软件开发相关TOP">Top for DEV</span></label>
            </li>
            <li>
                <input type="radio" name="lk" id="lk3" value="des">
                <label for="lk3"><span title="设计相关TOP">Top for DES</span></label>
            </li>
            <li>
                <input type="radio" name="lk" id="lk1" value="none" checked>
                <label for="lk1"><span>Hide</span></label>
            </li>
        </ul>
        <h4 title="托福最低要求">TOEFL</h4>
        <ul id="TOEFL">
            <li>
                <input type="checkbox" id="op1" value="100" checked />
                <label for="op1">
                    <span>100+</span>
                </label>
            </li>
            <li>
                <input type="checkbox" id="op2" value="90" checked />
                <label for="op2">
                    <span>90~100</span>
                </label>
            </li>
            <li>
                <input type="checkbox" id="op3" value="80" checked />
                <label for="op3">
                    <span>80~90</span>
                </label>
            </li>
            <li>
                <input type="checkbox" id="op4" value="1" checked />
                <label for="op4">
                    <span>0~80</span>
                </label>
            </li>
            <li>
                <input type="checkbox" id="op5" value="-1" checked />
                <label for="op5">
                    <span>No-Spec</span>
                </label>
            </li>
        </ul>
    </nav>
</div>
<script src="https://api.tiles.mapbox.com/mapbox.js/v2.2.3/mapbox.js"></script>
<script src="/Universities-in-US/app.min.js"></script>

## Data Categories

1. [Ivy League](https://en.wikipedia.org/wiki/Ivy_League)
2. [Best graduate universities for software developers](http://www.linkedin.com/edu/rankings/us/graduate-software-engineering?trk=edu-rankings-flt-ctg-dd)
3. [Best graduate universities for designers](http://www.linkedin.com/edu/rankings/us/graduate-design?trk=edu-rankings-flt-ctg-dd)
4. TOEFL minimum requirement