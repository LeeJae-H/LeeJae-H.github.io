---
title: "stdio, stdout, stderr"
tags:
    - term
date: "2024-12-08"
thumbnail: "/assets/img/thumbnail/sample.png"
---

foreground - terminal i/o 잡고 있어서, 인터럽트(Ctrl+C)에 영향을 받음. 터미널이 죽으면 프로그램도 따라 죽는다.

background - 서버 프로그램(데몬)

0 - stdin
1 - stdout
2 - stderr

1. nohub java -jar 0.0.1-SNAPSHOT.jar > app.log 2>&1 & 
	- 2>&1 는 stderr를 stdout에 포함해서 기록해라. 그리고 & 는 백그라운드로 실행해라.

2. ps -aef | grep java