셰이더(Shaders)는 컴퓨터의 그래픽 카드에서 실행되는 프로그램으로, 일반적으로 시각적인 효과를 만들기 위해 사용됩니다.
셰이더는 화면에 그려지는 픽셀이나 3D 모델을 구성하는 정점(버텍스) 등을 조작할 수 있습니다.

우리는 셰이더 프로그램을 그래픽 카드에서 일반적인 계산을 수행하는 데도 사용할 수 있습니다.
이 기술은 머신 러닝이나 고성능 연산 작업 등에서도 활용됩니다.

게임에서는 셰이더를 사용하여 다양한 효과를 만들어냅니다. 예를 들어, 스프라이트 주위에 외곽선을 추가하거나, 사물을 빛나게 하거나,
물웅덩이에 반사 효과를 흉내 내는 등 다양한 시각적 효과를 표현할 수 있습니다.

![shader_example_outline.mp4](images/shader_example_outline.mp4)

이 예시는 _Learn 2D Gamedev From Zero with Godot 4_ 의 M6. Looting! 모듈에서 가져온 것입니다.
상자 주위에 셰이더로 외곽선을 그려주는 모습을 보여줍니다.

셰이더는 또한 현대 그래픽의 핵심이기도 합니다. 여러분이 Godot을 사용할 때, 셰이더를 직접 작성하지 않아도 셰이더는 항상 사용되고 있습니다.
엔진이 여러분이 만든 2D 및 3D 씬을 렌더링할 때 셰이더를 이용하기 때문입니다.

### Shaders use a specific programming language

우리는 셰이더를 C 프로그래밍 언어와 유사한 특정 프로그래밍 언어로 작성합니다.
셰이더 언어는 여러 가지가 있지만, 가장 널리 사용되는 것은 GLSL(OpenGL Shading Language)입니다. Godot의 셰이더는 GLSL 언어의 변형을 사용합니다.

다음은 나뭇잎이 바람에 흔들리는 효과를 만드는 셰이더 예시입니다.

코드는 다음과 같습니다. 이해하지 못해도 괜찮습니다. 셰이더가 어떻게 생겼는지 감을 잡기 위한 예시일 뿐입니다:

```GLSL
shader_type canvas_item;

uniform sampler2D wind_sampler : repeat_enable;
uniform vec2 offset = vec2(0.0);
uniform float sway_amount = 1.0;

// 2D 벡터를 주어진 각도만큼 회전시키는 함수
vec2 rotate_2d(vec2 position, float angle) {
    float cosa = cos(angle);
    float sina = sin(angle);
    return vec2(
        cosa * position.x - sina * position.y,
        cosa * position.y + sina * position.x
    );
}

void vertex() {
    float angle = 0.5 - texture(wind_sampler, vec2(TIME * 0.04691, TIME * 0.0264) + offset).x;
    VERTEX = rotate_2d(VERTEX, angle * sway_amount);
}
```

프로그래밍 경험이 있다면, 2D 공간에서 무언가를 회전시키기 위해 우리가 직접 함수를 작성해야 한다는 점에 주목해보세요.

이는 셰이더가 매우 저수준(low-level) 프로그램이기 때문입니다. 
성능과 정밀한 제어가 중요하므로, GLSL과 같은 언어에는 Godot처럼 많은 내장 함수가 없습니다. 직접 많은 코드를 작성해야 합니다.

위 코드는 스프라이트가 바람에 흔들리는 것처럼 보이도록 애니메이션을 줄 수 있습니다.

우리는 이 셰이더를 아래 영상에서 나뭇잎을 움직이는 데 사용했습니다:

![shader_example_leaves_wind.mp4](images/shader_example_leaves_wind.mp4)
 
셰이더는 수학에 크게 의존하며, 게임 개발에서 독립적인 하위 분야로 간주됩니다.

보통 큰 개발 팀에서는 기술 아티스트나 그래픽 프로그래머가 셰이더를 작성하고, 다른 개발자들은 그것을 사용해 게임에 시각 효과를 추가합니다.