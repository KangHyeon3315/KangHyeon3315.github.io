---
title:  "Git Commit 메시지 규칙"
excerpt: ""

categories:
- Git

tags:
- []

toc: true
toc_sticky: true

date: 2022-10-27
last_modified_at: 2022-10-27
---

개발을 하다 보면 Git을 이용하면서 수많은 Commit을 하게 됩니다. 그리고 항상 Commit Message를 작성할 때마다 어떻게 Message를 작성해야 할지 고민입니다.


그러다 이번에 우아한 테크 코스를 진행하면서 Commit Message를 작성하는 규칙을 공유 받아 이번 기회에 정리해 보고자 합니다.


# Commit Message 구조

기본적으로는 아래 구조와 같이 이루어지고, 공백으로 이루어지는 행으로 구분합니다.

그리고 body와 footer는 선택사항으로 작성하지 않아도 됩니다.

```text
<type>: <subject>

<body>

<footer>
```

# Commit Type

|    type    | 설명                            |
|:----------:|-------------------------------|
|    feat    | 새로운 기능 추가                     |
|    fix     | 버그 수정                         |
|    docs    | 문서 수정                         |
|   style    | 코드 포맷팅, 세미콜론 누락, 코드 변경이 없는 경우 |
|  refactor  | 코드 리팩토링                       |
|    test    | 테스트 코드, 리팩토링 테스트 코드 추가        |
|   chore    | 빌드 업무 수정, 패키지 매니저 수정          |

# Commit Subject

제목은 50자 이내로 작성해야 합니다. 그리고 첫 글자를 대문자로 작성하고 마침표를 붙이지 않습니다.

그리고 과거시제보다 현재 시제를 사용해 명령어를 작성합니다.

# Commit Body

커밋에 약간의 설명과 컨텍스트가 필요한 경우에만 사용하면 됩니다. (선택사항)

본문을 작성할 때는 어떻게에 대한 정보보다는 "무엇을" 그리고 "왜"에 대한 내용을 설명합니다.

각 라인은 72자를 넘기지 않게 작성합니다.

# Commit Footer

Footer는 Issue tracker ID를 작성할 때 사용되고 선택사항입니다.

# Example

```text
feat: Summarize changes in around 50 characters or less

More detailed explanatory text, if necessary. Wrap it to about 72
characters or so. In some contexts, the first line is treated as the
subject of the commit and the rest of the text as the body. The
blank line separating the summary from the body is critical (unless
you omit the body entirely); various tools like `log`, `shortlog`
and `rebase` can get confused if you run the two together.

Explain the problem that this commit is solving. Focus on why you
are making this change as opposed to how (the code explains that).
Are there side effects or other unintuitive consequences of this
change? Here's the place to explain them.

Further paragraphs come after blank lines.

- Bullet points are okay, too

- Typically a hyphen or asterisk is used for the bullet, preceded
  by a single space, with blank lines in between, but conventions
  vary here

If you use an issue tracker, put references to them at the bottom,
like this:

Resolves: #123
See also: #456, #789

```

# 출처
[http://udacity.github.io/git-styleguide/](http://udacity.github.io/git-styleguide/)
