---
layout: post
title: 리눅스 방화벽 포트 관련 영구 설정
tags: [Linux, Port]
comments: true
mathjax: true
author: 이희두
---

# 리눅스 방화벽 포트 관련 영구 설정

# **작성 사유**

기존에는 리눅스 환경 내에서 sudo iptables -I INPUT 6 -m state --state NEW -p tcp --dport ${포트번호} -j ACCEPT 로 방화벽에 포트 적용하는 명령어로 설정했었음.

리부팅 후, 해당 설정들이 다 날라가는 이슈가 생김.

따라서 영구 설정이 필요함

# **netfilter-persistent**

1. iptables-persistent, netfilter-persistent 패키지 설치
    1. 해당 설정 파일 위치는 /etc/iptables에 rules.v4, rules.v6에서 확인 가능
2. sudo iptables -I INPUT 6 -m state --state NEW -p tcp --dport ${포트번호} -j ACCEPT 먼저 포트 적용
3. sudo iptables -L --line-numbers 추가 규칙 확인
4. sudo netfilter-persistent save && sudo netfilter-persistent reload 영구 저장 및 reload
5. /etc/iptables에 rules.v4, rules.v6에서 확인 가능

### **차단 방법**

위 방법에서 2번을 sudo iptables -I INPUT -p tcp --dport 8080 -j REJECT이나 DROP으로 진행하면 된다.

# **출처**

[https://darrengwon.tistory.com/699](https://darrengwon.tistory.com/699)
