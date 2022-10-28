---
title: "{{ replace .Name "-" " " | title }}"
date: {{ .Date }}
image: "images/post/{{ replace .Name ".md" ".jpg" }}"
description: "This is a meta description."
categories: []
draft: true
---

