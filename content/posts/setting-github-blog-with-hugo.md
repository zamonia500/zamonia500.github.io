---
title: "Official docs 만 참고해서 hugo github blog 세팅하기"
date: 2022-01-13T21:03:58+09:00
draft: false
---

한글 자료를 보고 따라했더니 뻑나서 삽질하다가 작성하는 문서. 
hugo & github official docs 만 참고해서 github blog를 hugo를 사용하도록 세팅했다. 
수행해야 하는 작업의 목록은 꽤나 많아 보이지만 사실 별거 없다. 쭉쭉 따라하면 20분 컷 가능하다. 
실수를 적게 할 수 있는 방향으로 작업의 순서와 일부 명령어를 편집하였다. 

# Install Hugo
[Official doc : Quick Start](https://gohugo.io/getting-started/quick-start/)를 참고했다.

```bash
# hugo 설치
brew install hugo
```

```bash
# hugo 설치 확인
hugo version
```

{{< asciicast ItACREbFgvJ0HjnSNeTknxWy9 >}}

나는 `hugo v0.91.2+extended darwin/amd64 BuildDate=unknown` 버전이 설치되었다.

# Create a New Site

나는 일반적인 URL(https://\<USERNAME\>.github.io/)을 사용하는 블로그를 만들고 있으므로 site 이름을 `zamonia500.github.io`라고 명명하겠다.

```bash
hugo new site zamonia500.github.io
```

# Add a Theme

테마를 추가한다.
원하는 테마는 `https://themes.gohugo.io/` 에서 찾아보자 나는 PaperMod라는 테마로 정했다.

테마 설치 가이드는 [Official doc : hugo-PaperMod/wiki/Installation](https://github.com/adityatelange/hugo-PaperMod/wiki/Installation#method-1)을 참고했다.

```bash
cd zamonia500.github.io
git init --initial-branch main
git clone https://github.com/adityatelange/hugo-PaperMod themes/PaperMod --depth=1
echo theme = \"PaperMod\" >> config.toml
```

# .nojekyll 파일 생성

`<USERNAME>.github.io`라는 포멧을 가진 github repository는 자동으로 jekyll이 enable 된다. 
jekyll이 아닌, 다른 hugo를 사용할 것이므로 [Official doc : Static site generators](https://docs.github.com/en/pages/getting-started-with-github-pages/about-github-pages#static-site-generators)를 참고하여 jekyll을 disable 하였다. 
그냥 루트 디렉토리에 `.nojekyll` 파일만 생성해 주면 된다.

```bash
touch .nojekyll
```

# Create a blog repository

<USERNAME>.github.io repository를 생성하자.

![repo](/images/posts/create-github-blog-repo.png)

# Add remote

```bash
git remote add origin git@github.com:zamonia500/zamonia500.github.io.git
```

# Site build & deploy를 위한 github action 설정

그냥 아래 내용을 복붙만 해도 설정이 자동!

```bash
mkdir -p .github/workflows
cat <<EOF > ./github/workflows
name: github pages

on:
  push:
    branches:
      - main  # Set a branch to deploy
  pull_request:

jobs:
  deploy:
    runs-on: ubuntu-20.04
    steps:
      - uses: actions/checkout@v2
        with:
          submodules: true  # Fetch Hugo themes (true OR recursive)
          fetch-depth: 0    # Fetch all history for .GitInfo and .Lastmod

      - name: Setup Hugo
        uses: peaceiris/actions-hugo@v2
        with:
          hugo-version: 'latest'
          # extended: true

      - name: Build
        run: hugo --minify

      - name: Deploy
        uses: peaceiris/actions-gh-pages@v3
        if: github.ref == 'refs/heads/main'
        with:
          github_token: ${{ secrets.GITHUB_TOKEN }}
          publish_dir: ./public
EOF
```

# Push to the github

이제 반영된 내용들을 github에 push 한 뒤 레포의 pages 설정을 업데이트 해 준다.
`Settings -> Pages`에서 Source branch를 gh-pages로 변경해주면 된다.
