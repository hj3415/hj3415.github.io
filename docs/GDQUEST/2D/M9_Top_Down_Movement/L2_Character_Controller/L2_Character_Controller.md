# L2 Character Controller

M4 강의인 _To Space and Beyond_에서는 8방향으로 움직일 수 있고 영역을 감지할 수 있는 우주선을 만들었습니다.
하지만 이 우주선은 어떤 것에도 부딪힐 필요가 없었지요. 이렇게 작동하는 게임은 거의 없습니다!

<video controls width="800" preload="metadata" playsinline>
  <source src="/GDQUEST/2D/M9_Top_Down_Movement/L2_Character_Controller/videos/050_ship_with_steering 3.mp4" type="video/mp4">
  동영상을 보려면 브라우저가 video 태그를 지원해야 합니다.
</video>

게임에는 장애물, 부딪힐 수 있는 요소들, 그리고 움직임을 제한할 벽이 필요합니다. 그렇지 않으면 레벨 디자인 자체가 불가능해집니다!

이번 강의에서는 CharacterBody2D 노드를 사용하여 캐릭터를 위한 탑다운 방식의 컨트롤러를 직접 코딩해보시게 됩니다. 
우리 캐릭터는 M4에서 만든 우주선처럼 움직이지만, 몇 강의 뒤에 벽과 장애물을 추가하게 되면 그 차이가 분명히 드러날 것입니다.

<video controls width="800" preload="metadata" playsinline>
  <source src="/GDQUEST/2D/M9_Top_Down_Movement/L2_Character_Controller/videos/020_character_010_runner.mp4" type="video/mp4">
  동영상을 보려면 브라우저가 video 태그를 지원해야 합니다.
</video>

이번 강의에서는 다음과 같은 내용을 배우시게 됩니다:

- 탑다운 방식의 캐릭터 컨트롤러 만들기
    
- 캐릭터를 여러 방향으로 움직이기
    
- CharacterBody2D 노드가 무엇이며, 이를 어떻게 사용하는지 학습하기
    
우선, **충돌(collision)** 에 대해 이야기해보겠습니다.

