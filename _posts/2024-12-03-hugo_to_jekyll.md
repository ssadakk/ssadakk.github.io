---
layout: post
title: "Hugo에서 Jekyll로 이전하기"
date: 2024-12-03 12:00:00 +0900
categories: [Blogging, Migration]
tags: [jekyll, hugo, migration]
excerpt: "Hugo에서 Jekyll로 이전"
author: "ssadakk"
---
### Hugo에서 Jekyll로 이전하기
최근 블로그를 [Hugo](https://gohugo.io/)에서 [Jekyll](https://jekyllrb.com/)로 옮김
오랜만에 들어와보니 Hugo는 자유도에 비해 해야하는것들이 좀 많게 느껴졌고, 글 쓰는 정도만 하는지라 Jekyll이 공식 지원도 되고 나쁘지 않다고 판단.

---

### **왜 Jekyll로 바꿨을까?**

1. **GitHub Pages랑 찰떡궁합**  
   Jekyll은 GitHub Pages에서 기본으로 지원되니까 배포가 간단.
   
2. **심플한 설정**  
   설정 파일도 직관적이라 커스터마이징하기 훨씬 편리.

3. **풍부한 테마와 플러그인**  
   다양한 테마랑 플러그인이 있어서 필요하면 바로 활용 가능.

---

### **Hugo 콘텐츠를 Jekyll로 변환하기**
Hugo는 content/ 디렉토리에 Markdown 파일을 저장.
근데 Jekyll은 _posts/ 디렉토리에 YYYY-MM-DD-title.md 형식을 요구.
포맷이 비슷한듯 하지만 파일 이름을 형식에 맞게 바꿔야 하고 시작 부분도 다른 부분이 있어서, 파이썬 스크립트로 바꿔봤다.

```python
import os
import re
import yaml
from datetime import datetime

def convert_front_matter(content):
    """
    Converts Hugo front matter (TOML/YAML) to Jekyll-compatible YAML front matter.
    """
    if content.startswith("---"):
        # Already YAML, split by '---'
        front_matter, body = content.split("---", 2)[1:]
    else:
        raise ValueError("Unknown front matter format")

    # Clean up and format YAML sequences
    front_matter = re.sub(r":\s*\[\s*(.*?)\s*\]", lambda m: f":\n  - " + "\n  - ".join(m.group(1).split(",")), front_matter)
    front_matter = re.sub(r"^\s*-\s*", r"  - ", front_matter, flags=re.MULTILINE)

    # Convert into a Python dictionary to validate the YAML
    try:
        yaml_data = yaml.safe_load(front_matter)
    except yaml.YAMLError as e:
        raise ValueError(f"Error parsing YAML: {e}")

    # Reconvert YAML back to a string to ensure consistency
    yaml_front_matter = f"---\n{yaml.dump(yaml_data, sort_keys=False)}---\n"
    return yaml_front_matter + body

def convert_file(hugo_file, output_dir):
    """
    Converts a single Hugo Markdown file to Jekyll format.
    """
    # Read the Hugo file
    with open(hugo_file, "r", encoding="utf-8") as file:
        content = file.read()

    # Convert the front matter
    converted_content = convert_front_matter(content)

    # Generate the output filename in Jekyll's `_posts` structure
    with open(hugo_file, "r", encoding="utf-8") as file:
        date_match = re.search(r"date:\s*([\d-]+)", file.read())
        post_date = date_match.group(1) if date_match else datetime.now().strftime("%Y-%m-%d")
        file_title = os.path.basename(hugo_file).replace(" ", "-").lower()
        file_title = re.sub(r"[^\w-]", "", file_title).replace(".md", "")
        jekyll_filename = f"{post_date}-{file_title}.md"

    # Write to the Jekyll `_posts` directory
    os.makedirs(output_dir, exist_ok=True)
    output_file = os.path.join(output_dir, jekyll_filename)
    with open(output_file, "w", encoding="utf-8") as file:
        file.write(converted_content)

    print(f"Converted: {hugo_file} -> {output_file}")

def process_directory(hugo_dir, jekyll_posts_dir):
    """
    Recursively processes all Markdown files in the Hugo directory.
    """
    for root, _, files in os.walk(hugo_dir):
        for file in files:
            if file.endswith(".md"):
                hugo_file = os.path.join(root, file)
                convert_file(hugo_file, jekyll_posts_dir)

# Paths
hugo_content_dir = "/Users/hmjoo/src/tmp/myblog/content/post"  # Replace with your Hugo content directory
jekyll_posts_dir = "/Users/hmjoo/src/tmp/myblog/content/output"  # Replace with your Jekyll `_posts` directory

# Process the directory
process_directory(hugo_content_dir, jekyll_poㅎts_dir)

```
문제없이 동작 했고 파일들을 뽑아낼 수 있었다.

그리고 Jekyll 로 변경하니, 확실히 편해지긴 한듯 하다.
커밋 하고 github action 을 통해 빌드하게 되니까 알아서 빌드하고 배포까지 한방에 끝 ㅎㅎㅎ