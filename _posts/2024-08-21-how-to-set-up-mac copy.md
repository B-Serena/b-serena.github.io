---
title: Project A: How to Set Up Virtual Environment on Mac M2
categories: [Dev]
comments: true
---


# How to Manage Virtual Environment on Mac M2


VScode 에서 가상환경을 관리하는 방법은 여러가지가 있습니다.
MacOS M2 에서 환경을 구축하는 방법을 정리해보았습니다.
데이터 사이언티스트인 저를 기준으로 작성되었습니다.
저도 웹 상에 공유된 환경 구축 세팅을 참고하며 더 쾌적하고 빠르게 좋은 환경을 만드는 방법을 고민해보고 있습니다.

대상:
- 새 컴퓨터에 새로운 환경을 구축하려는 분
- 코딩을 처음 시작하시려는 분

스펙:
MacBook Air M2, 2022
Chip: Apple M2
Memory: 16GB
macOS: Sonoma 14.6.1

기본적으로, 저는 가상 환경을 만들 때는 conda 를 사용하고 있습니다.
예전에는 conda 사용 시 환경이 꼬이는 문제가 종종 발생하여 venv를 한참 사용 했는데, 현재 conda 23.3.1 사용 중에는 문제가 없었습니다.
venv는 더 직관적이며, 가볍게 느껴져 회사 Windows 컴퓨터에서 잘 사용하고 있습니다.
추후에 Windows 환경 세팅도 작성하여 공유해보려 합니다.
먼저, MacOS 에서 conda 를 사용하여 가상환경 세팅을 하는 방법을 정리해보았습니다.

방법:
0. conda 를 macOS 용으로 설치해줍니다.
1. Finder 에서 내가 코딩할 때 전적으로 사용할 폴더를 하나 만들어줍니다. 예시: /Users/username/Documents/dev/
2. 그 안에 내가 특정 프로젝트에 사용할 가상환경을 만들 폴더를 하나 더 만들어줍니다. 예시: /Users/username/Documents/dev/myenv/
3. VScode 에서 macOS 용을 다운로드 받고 설치해줍니다.
4. VScode 를 실행하고 1번에서 만든 폴더를 열어줍니다.
5. VScode 의 상단 메뉴에서 터미널 탭을 클릭해줍니다.
6. 터미널 탭 안에서 새 터미널을 클릭해줍니다.
7. 터미널에서 다음 명령어를 입력하여 가상환경을 생성해줍니다.   

```bash
conda create -n myenv python=3.12
```

8. 가상환경을 활성화해줍니다.

```bash
conda activate myenv
```

9. 가상환경에 필요한 패키지를 설치해줍니다.

```bash
conda install pandas
```

10. shift+cmd+p 를 눌러 명령어 팔레트를 열어줍니다.
11. 명령어 팔레트에서 Python: Select Interpreter 를 선택해줍니다.
12. 가상 환경의 이름과 같은 interpreter 를 선택해줍니다.

13. 이제 가상환경에서 코딩을 시작할 수 있습니다.

주의할 점은 가상환경을 활성화할 때마다 터미널에서 명령어를 입력해줘야 한다는 점입니다.
또한 터미널을 여러개 띄워놓고 작업해야 할 때는 각각의 터미널마다 가상환경을 활성화해줘야 합니다.
interpreter 역시 프로젝트마다 다르게 설정해줄 수 있기 때문에 코드 실행 전에는 항상 확인해줘야 합니다.


그리고 바로 github repository 로 연결하여 사용합니다.
저는 개인적으로 fork 라는 프로그램을 사용하여 branch 관리하는 것이 편했습니다.
git을 접한지 얼마 안되었을 때 개념 잡기도 좋았습니다.

0. vscode 에서 github과 연결 시켜줄 프로젝트 폴더로 이동합니다.
1. fork 에서 new local repository 를 만들어줍니다.
2. 그 상태에서 github remote repo에 연결되어있는 폴더를 열어줍니다.
3. first commit을 하면서 branch를 설정해주고, remote repo를 연결해줍니다.