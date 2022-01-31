---
title: "Github blog에 도메인 붙이기"
date: 2022-01-31T19:52:38+09:00
draft: false
---

도메인을 하나만 사 두면 서브도메인을 붙여가면서 활용할 수 있을 것 같아서 사 보았다.  
[GeekNews](https://news.hada.io/topic?id=1621)에서 발견한 도메인 [Porkbun](https://porkbun.com/)에서 구매하였다.  
**zamonia500.com**이라는 도메인을 구입한 뒤, **blog**서브도메인을 사용하여 내 기술블로그에 대한 도메인으로 삼기로 했다.

porkbun에서 도메인을 구입하는 과정에 대한 설명은 생략하고 구입 이후에 진행해야 하는 두 가지 과정에 대해서 설명하겠다.  
설명하는 매뉴얼 내용과 사진은 [docs.github.com](https://docs.github.com/en/pages/configuring-a-custom-domain-for-your-github-pages-site/managing-a-custom-domain-for-your-github-pages-site#configuring-a-subdomain)를 참고했다.

1. **site repository** -> **Settings** -> **Pages**로 이동
![site repository](/images/posts/site-repository.png)
![settings](/images/posts/settings.png)

2. 내 도메인 등록 & 저장
![settings](/images/posts/pages.png)

3. Porkbun에 blog.zamonia500.com CNAME 등록
![](/images/posts/porkbun1.png)
![](/images/posts/porkbun2.png)
![](/images/posts/porkbun3.png)
![](/images/posts/porkbun4.png)
![](/images/posts/porkbun5.png)

### 끝!

이제 [blog.zamonia500.com](https://blog.zamonia500.com)으로 접속하면 잘 접속된다.
기존처럼 zamonia500.github.io로 접속해도 blog.zamonia500.com으로 리다이렉트 되는 것을 확인할 수 있다.

### Refs

- https://docs.github.com/en/pages/configuring-a-custom-domain-for-your-github-pages-site/managing-a-custom-domain-for-your-github-pages-site#configuring-a-subdomain
