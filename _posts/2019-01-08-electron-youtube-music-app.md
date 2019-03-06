---
title: "[Electron] Youtube Music desktop application 빌드 경험기"
date: 2019-01-08 12:44 -0400
categories: javascript
tags: javascript electron youtube
---

근 1년 전부터 멜론 정기결제를 중단하고 Youtube Premium을 이용하고 있다. Youtube Premium 회원은 Youtube Music이라는 별도 앱을 통해 모바일 기기에서 음원 스트리밍 서비스를 제공받을 수 있다. 

그러나, 데스크탑 환경에서는 앱이 존재하지 않아서 인터넷 브라우저에서 웹사이트를 통해 서비스를 제공 받아야 했다. 때문에 항상 여러개의 탭을 열어놓고 웹서핑을 하는 나는 이따금씩 실수로 탭을 닫아 음악이 중간에 끊기는 일이 발생했다. 이러한 불편을 없애기 위해 Electron을 이용해 간단한 데스크탑 앱을 만들기로 했다. 

## Electron이란?

> Build cross platform desktop apps with JavaScript, HTML, and CSS

위의 설명에서 볼 수 있듯이 Electron은 Javascript, HTML, CSS를 활용해 만든 웹사이트를 데스크탑 앱으로 빌드해주는 라이브러리이다. 나는 회사에서 해당 라이브러리를 활용해 제품을 생산하고 있다 :-)

## Youtube Music desktop application

Youtube Music의 경우 별도의 API가 제공되지 않고(Youtube API는 제공되고 있다), 이미 웹사이트가 잘 만들어져 있어서 별도의 개발 없이 기존 웹사이트를 감싸기만 해서 데스크탑 앱을 만들기로 했다.

다음 Github 주소에 접속하면 간단한 빌드 방법을 확인 할 수 있다.
https://github.com/armadillo-dev/youtube-music-app

![Youtube Music Desktop Application Home](/asserts/images/youtube-music-desktop-app-main.png)
*Youtube Music Desktop Application Home*

![Youtube Music Desktop Application Playlist](/asserts/images/youtube-music-desktop-app-playlist.png)
*Youtube Music Desktop Application Playlist*

개발자가 좋은 이유 중 하나는 필요한 것을 직접 만들어 쓸 수 있다는 점이다 :-)
앞으로도 필요한 것들은 직접 만들어서 공유하려고 한다.