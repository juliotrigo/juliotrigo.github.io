---
layout: page
title: About
last_modified_at: 2020-05-08 18:00:00 +0000
permalink: /about/
redirect_from: /p/about-this-blog.html
---

```json
{
    "name": "{{ site.author | normalize_whitespace | escape }}",
    "alumnus": {
        "university": "University of Alicante",
        "degree": "Computer Science Engineering"
    },
    "occupation": [
        "Software engineer", "Scrum master (CSM)", "Python developer", "Team lead", "Dev lead"
    ],
    "address": {
        "city": "A̶l̶i̶c̶a̶n̶t̶e̶ L̶o̶n̶d̶o̶n̶ Valencia",
        "country": "E̶S̶ G̶B̶ ES"
    },
    "company": "C̶e̶s̶s̶e̶r̶ W̶e̶b̶f̶u̶s̶i̶o̶n̶ R̶a̶z̶o̶r̶ ̶O̶c̶c̶a̶m̶ S̶t̶u̶d̶e̶n̶t̶.̶c̶o̶m̶ Sohonet",
    "website": {
        "url": "{{ site.url | normalize_whitespace | escape }}"
    },
    "twitter": {
        "url": "https://twitter.com/{{ site.twitter_username | normalize_whitespace | escape }}"
    },
    "github": {
        "url": "https://github.com/{{ site.github_username | normalize_whitespace | escape }}"
    },
    "pypi": {
        "url": "https://pypi.org/user/{{ site.pypi_username | normalize_whitespace | escape }}"
    },
    "slides.com": {
        "url": "https://slides.com/{{ site.slides_username | normalize_whitespace | escape }}"
    }
}
```
