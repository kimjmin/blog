#blog

> Kaijun/hexo-theme-huxblog 로 부터 fork.
> 정적 웹페이지 블로그 플랫폼인 hexo.io 기반.

###[블로그 바로가기](http://kimjmin.net)

nvm 사용 권장.
```
nvm use
npm install -g hexo
npm install
```

로컬 서버에서 실행
```
hexo serve
```

Static Web Page 빌드
```
hexo generate
```

`deploy` 이후에는 패키지들 사라짐 `npm install` 다시 해줘야 함. 왜그런지 모르겠다.


이하는 원 repository의 README.md 내용.


#Hexo-Theme-Huxblog

> Ported Theme of [Hux Blog](https://github.com/Huxpro/huxpro.github.io), Thank [Huxpro](https://github.com/Huxpro) for designing such a flawless theme.

###[Demo &rarr;](http://kaijun.rocks/hexo-theme-huxblog/)


![](http://huangxuan.me/img/blog-desktop.jpg)

## Usage

I didn't publish it as a single theme folder because a few of the pages are added and modified manually, so you should manually create some extra folders in `scaffolds` for the new pages and modify the `_config.yml` if you only have the single theme folder.

So i just pushed the whole hexo project for your convenience, all pre settings and boilerplates are included, have a look and go ahead customizing your own blog!

##### 1.Init

```
git clone https://github.com/Kaijun/hexo-theme-huxblog.git
cd hexo-theme-huxblog
npm install
```

##### 2.Modify
Modify `_config.yml` file with your own info.
Especially the section:

```
deploy:
  type: git
  repo: https://github.com/Kaijun/hexo-theme-huxblog
  branch: gh-pages
```
Replace with your own repo!

##### 3.Writting/Serve/Deploy

```
hexo new post IMAPOST
hexo serve // run hexo in local environment
hexo clean && hexo deploy // hexo will push the static files automatically into the specifig branch(gh-pages) of your repo!
```

##### 4.Enjoy! 
Please [**Star**](https://github.com/kaijun/hexo-theme-huxblog/stargazers) this Project if you like it! [**Following**](https://github.com/Kaijun) would also be appreciated!
