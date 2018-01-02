---
title: 开发者须知-女性玩家和手机游戏
date: 2018-01-02 19:02:37
tags: [译文, 科普]
categories: 译文
---

{% asset_img title.jpg %}

> * 原文地址：[Women and mobile games: learnings for developers](https://medium.com/googleplaydev/women-and-mobile-games-learnings-for-developers-cc4ac63da3f2)
> * 原文作者：[Tobias Knoke](https://medium.com/@tobias.knoke?source=post_header_lockup)
> * 译文出自：[掘金翻译计划](https://github.com/xitu/gold-miner)
> * 本文永久链接：[https://github.com/xitu/gold-miner/blob/master/TODO/women-and-mobile-games-learnings-for-developers.md](https://github.com/xitu/gold-miner/blob/master/TODO/women-and-mobile-games-learnings-for-developers.md)
> * 译者：[corresponding](https://github.com/corresponding)
> * 校对者：[hanliuxin5](https://github.com/hanliuxin5)，[tanglie1993](https://github.com/tanglie1993)


# 开发者须知：女性玩家和手机游戏
Women and mobile games: learnings for developers

## 让手机游戏变得更加多元化，更具包容性，更具吸引力的市场机遇
The market opportunity in making mobile gaming more diverse, more inclusive, and more engaging


目前世界上有二十多亿部活跃的安卓设备，这意味着比起过去更多的人在玩手机游戏。手机游戏玩家的增长导致了这些游戏玩家的特点，需求和动机的多样性在不断扩大。我们之前文章 [谁在玩手机游戏](https://medium.com/googleplaydev/who-plays-mobile-games-8b33f76bb6d8) 讨论过，现在需要以满足在玩家玩游戏时的需求来对待玩家，而不是根据一些刻板印象和像人口统计那样的死板数据。与此同时，现在在游戏界有很多关于性别和包容的讨论，而关于**女性手机游戏玩家**的研究和讨论却很少。
There are now over two billion active Android devices which means more people are playing mobile games than ever before. The growth of the mobile games audience had led to an expansion of diversity in characteristics, needs, and motivations of those playing games. In our previous post Who plays mobile games we discussed the opportunity to think of players in terms of the needs met by playing games, rather than in terms of stereotypes and demographics. At the same time, there is a lot of conversation in the gaming community about gender and inclusivity, but relatively little research or discussion about the experiences of women who play mobile games.

我们想更多的了解这方面，所以我们和游戏情报商 NewZoo 合作，进行定量的研究，希望以此了解美国女性玩家的体验和看法。我们和数十位游戏作者，测评人员，玩家和学者一起合作，把我们的研究场景化。通过 [交互性的体验](https://play.google.com/about/changethegame) 和 [在总结中收获更多](http://services.google.com/fh/files/misc/changethegame_white_paper.pdf) ，我们深入研究。请继续读下去，了解开发者如何让游戏更具包容性，并吸引近在咫尺的玩家。
We wanted to understand more about this so we partnered with gaming intelligence provider Newzoo to produce a quantitative research study to understand the experiences and perceptions of women who play games in the United States. We worked with dozens of game makers, critics, players, and academics to contextualize our research. Dive deeper into the insights through this interactive experience or discover more learnings in a summary of our findings. Read on to find out how you as a developer can make your game more inclusive and appealing to all players out there.

{% asset_img 0.png %}

### **了解你的用户**
Know your audience

无论从人数上和偏好上来说，女性手机游戏玩家有巨大市场潜力。在我们的调查中，美国有大量女性手机游戏玩家（其中 65% 都处于 10 - 65 岁年龄段）。在过去的一年里，这比去电影院看过电影的比例（62%）或读过一本书的比例（44%）更加高。这里可以清晰的看出，比起其他娱乐活动，女性更愿意玩手机游戏。报告还表明，女性就和男性一样喜欢玩手机游戏 — **有一半的玩家是女性**！相比其他平台，女性不仅更喜欢在手机上游戏，而且**女性玩的频率比男性更高**。
There is significant market potential in women who play mobile games, both in terms of volume and preference for the platform. In our research we found a significant number of women in the US play mobile games today (65% of those aged between 10–65 years). This is a higher percentage than those who have watched a movie at a movie theater (62%), or read a book (44%), in the last year. It’s clear that women engage significantly more in mobile gaming than in other forms of entertainment, and that’s not it… Research has shown that they are just as likely to play mobile games as men — women represent half of all mobile game players! They are not only more likely to prefer mobile compared to other platforms, women also tend to play more frequently than men.

作为游戏开发者，你意识到女性游戏的机会了吗？这可能是个通过了解这些用户而进入这个未开发的市场的机会。第一步，你可能要衡量和评估女性玩家的占比。**在你的用户群体中，女性是否被很好的代表了？她们和男性是否有不同的游戏体验？**
As a games developer, do you consider the opportunity presented women for your games? There may be an opportunity for you to grow your business in untapped markets by better understanding your players. As a starting point you might want to measure and evaluate the percentage of players who are women. Are women well represented in your player base? Is there a difference in their user experience compared to your male players?

对开发新游戏时的建议：当你在设计游戏或者思考未来的发展方向时，多去想想那些目标用户，而不用去讨好那些你所认为的"典型"用户。不同用户的游戏体验可能会不同吗？通过彻底分析和研究用户，你可以从那些尚未被顾及的用户（比如那些女性游戏玩家）中寻找商机。
The same goes for new developments: when designing your game, or thinking of future development, it can be a useful exercise to think of the range of people who may play your game, rather than gravitating to the ‘typical’ player. Could there be a difference in the user experience of some players? By thoroughly analysing and researching user audiences you can potentially make a strong business case to capture an underserved audience — such as women who play games.

{% asset_img 1.png %}

### **制作更具包容性的游戏**
Build more inclusive games

如果你观察 [ Google 应用市场](https://play.google.com/store) 中受欢迎的游戏，部分图像和图标的性质暗示着女性玩家在游戏世界中是一个相对较小的群体。在Google应用市场收入前 100 的游戏中，以男性角色作为图标的游戏数量比以女性角色作为图标的游戏数量多 44%。所以在我们的调查中，尽管女性玩家玩的更多，但是她们还是认为自己不属于现在的游戏社区。在推广游戏时，更换更合适的图标和形象，会让你从竞争中脱颖而出，也会减少你错过潜在玩家的可能。请尝试以下几点：
If you look at popular games on Google Play, the nature of much of the imagery and icons would imply women are a relatively niche group in the world of gaming: among the top 100 revenue generating games on Google Play, characters who are men are featured in their app icons 44% more often than characters who are women. Consequently our research found women often do not feel like they belong to this community although they are playing games very actively. Using alternative, or less alienating, iconography, characters, and imagery when promoting your game could help differentiate it clearly from your competition, and ensure you don’t miss out on reaching potential players. Try the following quick tips:

* 在做 [商店列表](https://support.google.com/googleplay/android-developer/answer/6227309?hl=en-GB) 时，**测试更有包容性的图像**。
Test more inclusive imagery when running store listing experiments.
* **多关注应用的图标，截图和视频**，并考虑测试不同图像对转换率的影响。
Pay attention to your icon, screenshots, and videos, and consider testing the impact of different imagery on conversion rates.
* 考虑下**使用女性角色来体验游戏**，或者在运行**电话回访**时**尝试新的可能性**。
Think of launching with characters who are women, or testing new ones when running LiveOps.
* 追踪用户对游戏角色的共鸣，同时**倾听用户群体的反馈**也很重要。
Track how people are resonating with the characters, and, invaluably, listen to the feedback of your community.

### **发展一个多样化的开发团队**
Grow a diverse team

在一大群人中提取共同需求很难。我们都想开发**自己**想玩的游戏。为了减少偏见，**在游戏生命周期的几个阶段，都需要获取潜在用户的反馈**。
It is very difficult to have the empathy and perspective needed to build a product that meets the needs of a wide range of potential people. It’s human nature to build games you want to play. To reduce potential bias, request feedback from a broad representation of your potential players at several stages throughout your game’s lifecycle.

你的开发团队的形象也影响能否取到悦更多的用户。尽管有如此多的女性玩家，游戏行业仍然只关注男性用户: IDGA的调查发现，女性、跨性别者和其他只占 [全球 27.8% 的游戏产业](http://c.ymcdn.com/sites/www.igda.org/resource/resmgr/files__2016_dss/IGDA_DSS_2016_Summary_Report.pdf) 。这种不平衡的现象呼应我们的研究结果，只有 23％ 的女性和 40％ 的男性认为在游戏行业中人人享有平等的待遇和机会。
The profile of your development team also impacts your ability to build games that appeal to a wide spectrum of players. Despite the high proportion of women playing mobile games, men are overrepresented in the gaming industry: according to the IDGA, only 27.8% of the gaming industry globally are women, transgender, or other people. This imbalanced representation is felt by players who responded to our research with only 23% of women and 40% of men believing there is equal treatment and opportunity for all in the games industry.

来自团队成员多样化的观点将帮助您开发真正创新且有意思的游戏，并且吸引更多的潜在玩家。现在就看下自己的团队和游戏的用户构成的差别。你的团队成员能代表你的受众吗？您的团队是否已经整装待发，来帮助您最大化的捕获潜在受众，并且让您的游戏吸引所有人？
Diversity of perspectives from your team members will help you build truly innovative and exciting games that will appeal to a broad spectrum of potential players. Look at your team and compare it with the user composition of your game. Is your team truly representative of your audience? Is it well equipped to help you capture the maximum potential audience possible by making your game appealing to everyone?

{% asset_img 2.png %}

### **把握这个机遇**
Take advantage of this opportunity

尽管未来女性玩家数量巨大，但研究结果却让人惊讶，他们比男性更难以真正接受自己的游戏爱好。大部分女性玩家不属于游戏世界。一般女性玩家不太喜欢和朋友交谈游戏内容，为游戏付费，以及享受付费所带来的快乐。
While there is great potential in the sheer volume of women playing mobile games, it is striking from the research how less likely they are than men to truly embrace their play habits. With a few notable exceptions, there is a sense that women don’t belong in the world of gaming. They are less likely to talk about games with their friends, pay for content, or feel good when they do pay.

我们相信，这是一个游戏产业真正与女性玩家互动的绝佳机会。随着用户获取成本的上升，需要想想如何能开发出与所有玩家产生共鸣并且能病毒式传播的游戏。只有让女性玩家参与到游戏中，你才能解决这个问题。
We believe this represents a great opportunity for the industry to truly engage with women who play games. As user acquisition costs rise, think of the potential virality of building a game that resonates with and excites all players. This can only be achieved by recognizing and addressing the barriers for women to engage with your game.

### **着眼未来**
The road ahead

我们相信，游戏市场还有很大的空间，能让我们使手机游戏更加多元化，更具包容性，更吸引所有玩家。为了把握这个机会，您的第一步是：
We believe there is a great opportunity in market growth and making mobile gaming more diverse, more inclusive, and more engaging for all players. The first steps for you in order to take advantage of this opportunity are:

* 了解你的用户：当前用户和潜在用户
Know your audience: current and potential
* 研究你的游戏是怎样把一些潜在用户排除在外的
Consider how your games may exclude some potential players
* 评估你团队观点的多样性，这将如何影响你开发的游戏
Assess the range of perspectives in your team, and how this affects the games you build
* 头脑风暴，你可能会想出一款所有人都喜欢的游戏
Brainstorm how you may produce the next game that all players could embrace

手机游戏生为大众。为了褒奖和激励女性玩家和女性开发者，我们启动了 [改变游戏](http://g.co/changethegame) 的计划。这是 Google 应用商店的一项旨在促进游戏多样性的计划，同时这项计划也褒奖所有女性玩家，并通过正在进行的研究和合作为下一代游戏开发者提供支持。
Mobile games are for everyone. To celebrate and empower women who play games and creators, we’re launching CHANGE THE GAME; a new Google Play initiative to promote diversity in games, celebrate all women who play games, and empower the next generation of game-makers through ongoing research, development programs, and partnerships.

作为一个应用开发者，你能影响未来游戏的走向。我希望你能参与到我们活动中，和我们一起让游戏世界变得更加包容性。如果我们一起努力，手机游戏的世界会变得更加有趣。
As a mobile developer, you have a major influence on how future games will look like. We hope you can join our efforts of making the gaming world a more inclusive community; if we all contribute to this, mobile games will bring us even more joy than they do today.

* * *

### 你是怎么想的呢？
What do you think?

你有没有想过开发人员怎样去设计更有包容性的游戏呢？在文章下面留言或者 twitter 中添加**#AskPlayDev**标签后发言，我们会通过 [@GooglePlayDev](http://twitter.com/googleplaydev) （那里我们会展示在 Google 应用商店获得成功的窍门）回复。
Do you have thoughts on how developers can build more inclusive games? Join the discussion in the comments below or tweet using the hashtag #AskPlayDev and we’ll reply from @GooglePlayDev, where we regularly share news and tips on how to be successful on Google Play.

---