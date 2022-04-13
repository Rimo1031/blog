---
title: "Markdown 문법 정리"
excerpt: "Markdown의 기초적인 사용법을 정리한다."
categories: 
  - Programming Language
tags:
  - Markdown
toc: true
toc_sticky: true
# last_modified_at:
---

Markdown은 [경량 마크업 언어](https://ko.wikipedia.org/wiki/가벼운_마크업_언어)의 일종으로, 일반 마크업 언어보다 문법이 쉬우면서 HTML, rtf 등 서식으로의 변환이 간단해 널리 쓰이고 있다. 당장 이 블로그의 포스팅도 전부 Markdown으로 작성되고 있지만, 아마 Markdown을 가장 많이 볼 수 있는 곳은 소프트웨어의 README 파일일 것이다. Markdown을 잘 활용하면 직관적인 문법으로 가독성 높은 문서를 만들 수 있다. 

## 사용법

### 일반 텍스트
```
일반적인 텍스트는 그대로 적으면 된다.
```
일반적인 텍스트는 그대로 적으면 된다.

---

### 헤더(제목)
제목을 만들 때는 제목으로 만들 텍스트 앞에 #을 붙이면 된다. #의 개수에 따라 HTML 태그의 `<h1>` ~ `<h6>`에 대응한다.

```
# This is h1
## This is h2
### This is h3
#### This is h4
##### This is h5
###### This is h6
####### This is h7 //h7부터는 존재하지 않는다.
```

![]({{site.url}}{{site.baseurl}}/assets/img/markdown-header-img.png)

---

### 줄바꿈

Markdown은 줄바꿈 문자 2개 이상부터 줄바꿈으로 인식한다.

```
Change Line
Change Line

Change Line
```
Change Line
Change Line

Change Line

---

Markdown은 문서 내부에 HTML 태그를 포함해도 정상적으로 인식한다(inline HTML). 따라서 줄바꿈을 의미하는 HTML 태그 `<br>`을 삽입해도 줄바꿈이 가능하다.

```
줄바꿈<br>줄바꿈
```

줄바꿈<br>줄바꿈

---

### 인용

인용문으로 설정하려는 텍스트 앞에 `>` 기호를 붙인다. 여러 줄에 걸쳐 연속적으로 붙어있으면 하나의 인용문으로 인식한다.

```
> This is a blockquote with two paragraphs. Lorem ipsum dolor sit amet,
> consectetuer adipiscing elit. Aliquam hendrerit mi posuere lectus.
> Vestibulum enim wisi, viverra nec, fringilla in, laoreet vitae, risus.
> 
> Donec sit amet nisl. Aliquam semper ipsum sit amet velit. Suspendisse
> id sem consectetuer libero luctus adipiscing.
```

> This is a blockquote with two paragraphs. Lorem ipsum dolor sit amet,
> consectetuer adipiscing elit. Aliquam hendrerit mi posuere lectus.
> Vestibulum enim wisi, viverra nec, fringilla in, laoreet vitae, risus.
> 
> Donec sit amet nisl. Aliquam semper ipsum sit amet velit. Suspendisse
> id sem consectetuer libero luctus adipiscing.

---

`>`의 개수로 인용문의 단계를 설정할 수 있다.

```
> This is the first level of quoting.
>
> > This is nested blockquote.
>
> Back to the first level.
```
> This is the first level of quoting.
> 
> > This is nested blockquote.
> >
> > > This is double-nested blockquote.
>
> Back to the first level.

인용문 안에 다른 Markdown 요소가 포함될 수도 있다.

```
> # This is h1
> ## This is h2
```

![]({{site.url}}{{site.baseurl}}/assets/img/markdown-quote-header.png)

### 목록

순서가 있는 목록은 1. 2. 와 같은 숫자로, 순서가 없는 목록은 *, +, - 등의 기호로 표현한다. 들여쓰기하면 하위 목록으로 인식한다.

```
1. Ordered list 1
2. Ordered list 2
3. Ordered list 3
    1. Ordered list 3-1


* Unordered list 1
+ Unordered list 2
- Unordered list 3
    * Unordered list 3-1
    + Unordered list 3-2
        - Unordered list 3-2-1
```

1. Ordered list 1
2. Ordered list 2
3. Ordered list 3
    1. Ordered list 3-1

* Unordered list 1
+ Unordered list 2
- Unordered list 3
    * Unordered list 3-1
    + Unordered list 3-2
        - Unordered list 3-2-1

### 코드

일반적인 텍스트에서 4칸 들여쓰기하거나, 코드 블록을 백틱(`) 3개로 감싸는 방식으로 코드를 정의한다. 백틱 1개로 블록을 감싸면 문장 내의 코드 블록이 된다.

    Normal paragraph

        Code Block

    Normal paragraph

    ```
    Code Block
    ```

    This is `inline code block`.

Normal paragraph

    Code Block

Normal paragraph
```
Code Block
```

This is `inline code block`.

### 수평선
3개 이상의 asterisk(*), 하이픈(-), 언더바(_)를 나열해서 만들 수 있다.

```
***
---
___
****
```

***
---
___
****

### 링크
`[링크 텍스트](링크할 주소)`와 같은 형식으로 만들 수 있다. 링크할 주소에는 같은 사이트 내의 상대 주소나, 현재 문서의 다른 헤더로 이동하는 것도 가능하다. 다른 헤더로 이동할 때는 헤더 제목을 적절하게 변환한 다음 링크해야 한다.

#### 헤더 제목 변환
1. 헤더 제목에 대문자가 포함되어 있으면 모두 소문자로 바꾼다.
2. 헤더 제목의 공백을 모두 하이픈으로 바꾼다.
3. 헤더 제목의 특수문자를 모두 제거한다.
4. 헤더 제목 앞에 #을 붙인다.

예를 들어 헤더 이름이 `Another (Not really exists)`라면 `(#another-not-really-exists)`로 변환해 링크한다.

```
[Google](https://www.google.com)

[Blog Main](/blog/main)

[Move to header 코드](#코드)
```

[Google](https://www.google.com)

[Blog Main](/blog/main)

[Move to header 코드](#코드)


### 강조

asterisk(*)나 언더바(_) 1개로 감싸면 이탤릭체로, 2개로 감싸면 볼드체로 표시한다.
물결표(~) 2개로 감싸면 취소선을 표시한다.

```
*Italic* _Italic_

**Bold** __Bold__

~~CancelLine~~
```

*Italic* _Italic_

**Bold** __Bold__

~~CancelLine~~


### 이미지

링크를 걸 때와 동일하되, 맨 앞에 !를 붙여주고 [] 안에는 대체 텍스트를 입력한다. (비워도 된다.)

```
![](/assets/img/profile-niko.JPG)

![Alt Text](/assets/img/teaser.jpeg)
```

![]({{site.url}}{{site.baseurl}}/assets/img/profile-niko.JPG)

![Alt Text]({{site.url}}{{site.baseurl}}/assets/img/teaser.jpeg)


## Reference

- [Markdown Documentation](https://daringfireball.net/projects/markdown/)