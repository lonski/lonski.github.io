---
layout: post
title: Word Tower
subtitle: A typing game
date: 2018-03-21
categories: [programming]
---
I had an idea for a mini-game for training keyboard typing. A tower with falling blocks. Each block has a word on it, when typed - dissappears. If any other blocks were laying on it - they all fall (a physics engine Box2d is used). The goal is to type all the words and get as hight score as possible. You score points by typing the words of course. I had few ideas for the score rules, for example: the more blocks were laying on the block that dissappeard, the highest score for making him dissappear. Eventually there could be also harder mode, in which game would over if any of the blocks would fall on the floor.

I wrote a very first version:
[https://github.com/lonski/word-tower](https://github.com/lonski/word-tower)

Some screenshots:

<div style="text-align:center">

<a href="https://raw.githubusercontent.com/lonski/word-tower/master/screenshots/Screenshot_20180320_234518.png">
<img src="https://raw.githubusercontent.com/lonski/word-tower/master/screenshots/Screenshot_20180320_234518.png" width="400"></a>
<a href="https://raw.githubusercontent.com/lonski/word-tower/master/screenshots/Screenshot_20180320_230703.png">
<img src="https://raw.githubusercontent.com/lonski/word-tower/master/screenshots/Screenshot_20180320_230703.png" width="400"></a>
<a href="https://raw.githubusercontent.com/lonski/word-tower/master/screenshots/Screenshot_20180320_230740.png">
<img src="https://raw.githubusercontent.com/lonski/word-tower/master/screenshots/Screenshot_20180320_230740.png" width="400"></a>

</div>

Word database source: [https://github.com/dwyl/english-words](https://github.com/dwyl/english-words), it contains few hundred of english alphanumeric words.

Levels are defined as plain text file: [https://github.com/lonski/word-tower/tree/master/assets/levels](https://github.com/lonski/word-tower/tree/master/assets/levels)
