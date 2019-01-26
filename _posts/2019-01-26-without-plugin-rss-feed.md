---
layout: posts
title:  "지킬 블로그 RSS Feed 설정하기"
date:   2019-01-26 18:45:21 +0900
comments: true
categories: blog
tags:
  - jekyll
---

저는 지킬 블로그를 개설하면서 구글 웹마스터와, 네이버 웹마스터 두곳에 현 블로그를 등록하여 검색엔진에 노출되도록 설정했었습니다.

구글 웹마스터 도구는 무리없이 사이트 등록이 깔끔하게 완료되었지만, 이상하게 NAVER 웹마스터에서만 RSS를 제출하려고 하면 서비스 거부를 당해서 방치;

지킬 블로그에서 RSS 피드를 생성하는 방법은 두 가지가 있습니다.

- **플러그인 사용**

- **RSS 피드 생성문 직접작성**

저는 편하게 하고싶어서 **jekyll-feed** 라는 플러그인을 사용하여 피드를 생성하고 있었는데, 이상하게 네이버 웹마스터에서 해당 플러그인으로 생성된 피드에 대하여 서비스를 거부하는 현상이 있어서, 플러그인을 포기하기로 결정했습니다. (*개인적인 생각으로는 RSS 버전이슈 같습니다. 피드문서 구조가 달랐거든요*)


현재상태
![RSS 제출 확인](https://user-images.githubusercontent.com/25237661/51785372-3cc90d00-219a-11e9-8afa-c70cbb441b76.jpg)


찾아보니 [지킬 코드샘플 사이트에서](https://jekyllcodex.org/without-plugin/rss-feed/) 기본 솔루션을 제공해 주고 있었습니다.

우선 아래처럼 RSS 피드를 생성하는 xml을 작성하고 원하는 이름으로 지킬 프로젝트 루트경로에 위치시킵니다. (기본 feed.xml)
```xml
---
layout: null
---
<?xml version="1.0" encoding="UTF-8"?>
<rss version="2.0" xmlns:atom="http://www.w3.org/2005/Atom">
  {% raw %}{% assign feed_path = "/feed.xml" %}{% endraw %} <!-- 피드경로 명시 -->
  <channel>
    <title>{% raw %} {{ site.title | xml_escape }} {% endraw %} </title>
    <description>{% raw %}{{ site.description | xml_escape }}{% endraw %}</description>
    <link>{% raw %}{{ site.url }}{{ site.baseurl }}{% endraw %}/</link>
    <atom:link href="{% raw %}{{ feed_path | prepend: site.baseurl | prepend: site.url }}{% endraw %}" rel="self" type="application/rss+xml"/>
    <pubDate>{% raw %}{{ site.time | date_to_rfc822 }}{% endraw %}</pubDate>
    <lastBuildDate>{% raw %}{{ site.time | date_to_rfc822 }}{% endraw %}</lastBuildDate>
    <generator>Jekyll v{% raw %}{{ jekyll.version }}{% endraw %}</generator>
    {% raw %}{% for post in site.posts limit:10 %}{% endraw %}
      <item>
        <title>{% raw %}{{ post.title | xml_escape }}{% endraw %}</title>
        <description>{% raw %}{{ post.content | xml_escape }}{% endraw %}</description>
        <pubDate>{% raw %}{{ post.date | date_to_rfc822 }}{% endraw %}</pubDate>
        <link>{% raw %}{{ post.url | prepend: site.baseurl | prepend: site.url }}{% endraw %}</link>
        <guid isPermaLink="true">{% raw %}{{ post.url | prepend: site.baseurl | prepend: site.url }}{% endraw %}</guid>
        {% raw %}{% for tag in post.tags %}{% endraw %}
        <category>{% raw %}{{ tag | xml_escape }}{% endraw %}</category>
        {% raw %}{% endfor %}{% endraw %}
        {% raw %}{% for cat in post.categories %}{% endraw %}
        <category>{% raw %}{{ cat | xml_escape }}{% endraw %}</category>
        {% raw %}{% endfor %}{% endraw %}
      </item>
    {% raw %}{% endfor %}{% endraw %}
  </channel>
</rss>
```

그후 head에 아래와 같은 태그를 추가시키면 됩니다.
```html
<link rel="alternate" type="application/rss+xml" href="{% raw %}{{ site.url }}{% endraw %}/feed.xml" title="{% raw %}{{ site.title }}{% endraw %} Feed">
```

의도한대로 동작하고 있는지 개발자도구(F12)를 실행시켜 확인해보고, feed URL을 체크해봅니다
![dev_tool_check](https://user-images.githubusercontent.com/25237661/51785473-95e57080-219b-11e9-9e01-8435e8952128.jpg)


정상적으로 동작하고 있는걸 확인했으니 네이버 웹마스터에 RSS 제출을 하겠습니다.
![RSS 제출](https://user-images.githubusercontent.com/25237661/51785371-3c307680-219a-11e9-99bb-fdfc60634a2b.jpg)

![RSS 제출 확인](https://user-images.githubusercontent.com/25237661/51785529-93374b00-219c-11e9-8fa6-6a1f7dcfa826.jpg)
