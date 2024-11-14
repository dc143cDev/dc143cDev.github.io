---
layout: post
title: "프롬프트 한 문장으로 프로젝트 시각화하기"
date: 2024-10-24
tags: [AI, Tool, IDE, Documentation]
read_time: 7
subtitle: "Cursor IDE의 코드베이스 기능과 머메이드 AI의 마크다운을 이용하여 프로젝트의 구조도를 그려봅시다"
---

### 개요

첫 직장. 첫 프로젝트를 완수한 후 얼마 지나지 않아 제게 새로운 미션이 생겼습니다.

그동안 개발했던 앱의 일부 핵심 기능을 뽑아 추후 신규 프로젝트에 활용할수 있도록 고도화 하라는 임무였죠.
작업에 앞서 기존 프로젝트의 구조와 주요 기능의 동작을 파악하는 과정이 필요했습니다.

하지만 작지 않은 규모의 이 기존 앱은 아직 신입의 실력인 제가 개발 영역을 거의 혼자 담당했기에, 각각의 모듈이나 주요 함수를 문서화하기는 커녕, 프로젝트의 전반적인 구조도조차 만들지 못했습니다.

새로운 미션은 감사하고 기쁜 일이지만, 프로젝트의 문서화를 게을리 해온 저에게는 막연한 일이었습니다.

그래서 지난날의 저의 나태함을 만회하기 위해 커서와 머메이드를 활용해보기로 했습니다.



### 머메이드(Mermaid)가 뭐에요?

우선, 프롬프트로 프로젝트를 시각화하기 위해 머메이드라는 도구에 대해 알아야 합니다.

**Mermaid**는 간단한 문법을 사용하여 다양한 다이어그램과 차트를 생성할 수 있는 오픈 소스 도구입니다.

마크다운(Markdown)과 함께 사용하여 코드 내에서 흐름도, 시퀀스 다이어그램, 클래스 다이어그램 등을 손쉽게 작성하고 시각화할 수 있습니다.

- **홈페이지**: [https://www.mermaidchart.com/mermaid-ai](https://www.mermaidchart.com/mermaid-ai)
- **GitHub 레포지토리**: [https://github.com/mermaid-js/mermaid](https://github.com/mermaid-js/mermaid)




<figure>
  <img src="/assets/images/post-2041029-012.png" alt="머메이드 구조도 스크린샷" class="screenshot">
  <figcaption>머메이드 AI 예시를 위한 플러터 게임 샘플 프로젝트 구조도</figcaption>
</figure>


```markdown
graph TD
    A[Main] --> B[EndlessRunner Game]
    B --> C[Game Features]
    B --> D[UI Components]
    B --> E[Audio System]

    C --> C1[Player]
    C --> C2[World]
    C --> C3[Collision]
    C --> C4[Background]

    D --> D1[Main Menu]
    D --> D2[Settings]
    D --> D3[Level Selection]
    D --> D4[Game Screen]

    E --> E1[Sound Effects]
    E --> E2[Background Music]
    E --> E3[Audio Controller]

```

예시로, 위 구조도를 구현한 마크다운입니다.

위의 마크다운 문법을 사용하면 계층형으로 표현된 구조도를 그릴수 있습니다.

<figure>
  <img src="/assets/images/post-241029-02.png" alt="머메이드 시퀀스 다이어그램 스크린샷" class="screenshot">
  <figcaption>머메이드 AI 예시를 위한 플러터 게임 샘플 시퀀스 다이어그램</figcaption>
</figure>

마크다운:
```markdown
graph TD
 sequenceDiagram
    actor Player
    participant MainMenu
    participant LevelSelection
    participant GameScreen
    participant EndlessWorld
    participant PlayerComponent
    participant ScoreSystem
    
    Player->>MainMenu: 게임 시작
    MainMenu->>LevelSelection: Play 버튼 클릭
    
    Player->>LevelSelection: 레벨 선택
    LevelSelection->>GameScreen: 레벨 로드
    
    GameScreen->>EndlessWorld: 월드 초기화
    EndlessWorld->>PlayerComponent: 플레이어 생성
    
    loop Game Loop
        Player->>PlayerComponent: 화면 탭하여 점프
        PlayerComponent->>EndlessWorld: 충돌 체크
        EndlessWorld->>ScoreSystem: 점수 업데이트
        
        alt 목표 점수 달성
            ScoreSystem->>GameScreen: 승리 다이얼로그 표시
            GameScreen->>LevelSelection: 다음 레벨 이동
        else 장애물과 충돌
            EndlessWorld->>PlayerComponent: 피해 효과 적용
            PlayerComponent->>ScoreSystem: 점수 리셋
        end
    end

```

이번엔 시퀀스 다이어그램의 사례입니다.

위와 같이 시퀀스 마크다운 문법을 사용하여 어떠한 모듈이나 기능을 시연하는 유저의 행동 흐름을 시각화할 수 있습니다.



### 그래서, 어떻게 만드나요?

위와 같은 프로젝트/기능 시각화는 커서 IDE의 코드베이스 기능을 활용합니다.

머메이드는 생소하실수 있지만, 커서 IDE는 이미 알고 계신 분들이 많을 겁니다.

기존에도 깃허브 코파일럿처럼 코드 작성을 도와주는 생성형 AI 도구는 존재했지만, 커서 IDE는 이런 코드 작성 기능을 더욱 발전시켜 많은 개발자들에게 인기를 얻고 있죠.

이러한 커서의 기능 중 핵심은 코드베이스 분석 기능입니다.

대화창에 코드베이스 기능(cmd + Enter)을 활성화하고 프롬프트를 요청하면, 커서가 현재 작업중인 프로젝트 디렉토리의 모든 파일을 분석하여 우리의 프롬프트를 더욱 거시적으로 이해합니다.


<figure>
  <img src="/assets/images/post-241029-03.png" alt="커서 코드베이스 기능을 통해 마크다운 생성" class="screenshot">
  <figcaption>커서 코드베이스 기능을 통해 마크다운 생성</figcaption>
</figure>

프롬프트:
> 이 플러터 게임 프로젝트의 작동 구조를 파악하고, 유저가 게임을 진행하는 과정을 시퀀스 다이어그램으로 나타내주세요. 머메이드 AI에 입력할수 있도록 마크다운을 생성해주세요.

우리의 목적인 프로젝트 시각화에도 이 기능을 활용합니다.

커서의 대화창에서 코드 베이스 모드를 활성화한 뒤, 프로젝트의 구조(혹은 특정 기능의 시퀀스)를 파악하고 머메이드 마크다운을 생성해달라는 프롬프트를 요청합니다.

<figure>
  <img src="/assets/images/post-241029-04.png" alt="머메이드 에디터에 완성된 마크다운 입력" class="screenshot">
  <figcaption>머메이드 에디터에 완성된 마크다운 입력</figcaption>
</figure>

그 후 머메이드 에디터에 완성된 마크다운을 입력하면 쉽게 시각화를 완성할 수 있습니다.



### 요약

1. 커서의 대화창에서 코드 베이스 모드를 활성화합니다. (cmd + Enter)

2. 프로젝트의 구조(혹은 특정 기능의 시퀀스)를 파악하고 머메이드 마크다운을 생성해달라는 프롬프트를 요청합니다.

   예시: 이 플러터 게임 프로젝트의 작동 구조를 파악하고, 유저가 게임을 진행하는 과정을 시퀀스 다이어그램으로 나타내주세요. 머메이드 AI에 입력할수 있도록 마크다운을 생성해주세요.

3. 머메이드 에디터에 완성된 마크다운을 입력하면 쉽게 시각화를 완성할 수 있습니다.

4. 머메이드 AI를 활용하여 마크다운의 스타일을 조정할 수 있습니다. (선택 사항)

### 마치며

저 역시도 오늘 소개한 프로젝트 시각화 방법을 유용하게 사용하고 있습니다.

상술한 개요의 예시처럼 이미 작업한 프로젝트의 문서화/시각화를 빠르게 마무라하는 것 뿐만 아니라, 처음 만나는 오픈소스 프로젝트의 코드를 일일히 확인하며 이해할 시간이 모자랄때 빠르게 훑어보기 위해 사용하기도 합니다.

커서의 코드베이스 기능은 단지 코드 생성을 더 정확하게 만들어주는 것 뿐만 아니라 이처럼 프로젝트를 시각화하는 등 다양한 영역에 활용될 가능성이 높아 보입니다.

여러분도 커서의 코드베이스 기능을 다양하게 활용하는 방법을 연구해 보셨으면 좋겠습니다.

읽어주셔서 감사합니다!
