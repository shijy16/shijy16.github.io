---
permalink: /
title: "About Me"
author_profile: true
redirect_from:
  - /about/
  - /about.html
---

<style>
  .page__content h2 { font-size: 1.25em; margin-top: 1.5em; }
  .page__content p, .page__content li { font-size: 0.95em; line-height: 1.5; }
  .page__content .pub-item { margin-bottom: 0.8em; }
  .page__content .pub-title { font-weight: 600; }
  .page__content .pub-venue { font-style: italic; color: #555; }
  .page__content .pub-authors { color: #555; font-size: 0.92em; }
</style>

<div markdown="0" style="text-align: center; margin-bottom: 1em;">
  <em>Ph.D. Candidate</em><br/>
  Institute of Information Engineering, Chinese Academy of Sciences
</div>

<div markdown="0" style="text-align: center; margin-bottom: 2em;">
  <a href="mailto:shijingyi16@gmail.com">Email</a> &middot;
  <a href="https://scholar.google.com/citations?user=r_SGfmMAAAAJ">Google Scholar</a> &middot;
  <a href="https://github.com/shijy16">GitHub</a>
</div>

I am a Ph.D. candidate at the Institute of Information Engineering, Chinese Academy of Sciences, advised by Prof. Wei Huo and Prof. Yang Xiao.

I received my B.Eng. degree from the Department of Computer Science and Technology at Tsinghua University in 2020. In 2025, I was a visiting Ph.D. student at Nanyang Technological University, advised by Prof. Yang Liu.

My research focuses on the security of modern software and AI ecosystems, with particular interests in software supply chain security, AI agents for security, and AI infrastructure security.

## Research Interests

* **Software Supply Chain Security**
* **AI Agents for Security**
* **AI Infrastructure Security**

## Publications

{% for post in site.publications reversed %}
<div class="pub-item">
  <div class="pub-title">{{ post.title }}. <span class="pub-venue">{{ post.venue }}.</span></div>
  <div class="pub-authors">{{ post.authors | replace: "Jingyi Shi", "<strong>Jingyi Shi</strong>" }}</div>
</div>
{% endfor %}

## Honors and Recognition

* Listed in the **Google Bug Hunters Hall of Fame**.
* Listed in the **Intel Security Researcher Hall of Fame**.
* Invited to participate in **Google BugSWAT 2023** in Tokyo, Japan.

