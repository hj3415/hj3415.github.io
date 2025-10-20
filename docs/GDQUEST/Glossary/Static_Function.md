정적 함수(static function)는 입력값에 대해 계산만 수행하고 그 결과값을 반환하는 함수입니다. 이 함수는 [Class](Class.md)자체에 속하며,
**그 클래스의 [Object](Object.md)\(인스턴스\)** 에 속하지 않습니다.

이는 곧, 정적 함수는 **클래스 인스턴스에만 존재하는
[Property, member variable](Property_member_variable.md)들에는 접근할 수 없다는 것**을 의미합니다.
처음엔 별로 유용하지 않게 들릴 수도 있겠죠?

하지만 좋은 점도 있습니다.
**정적 함수는 클래스를 인스턴스화하지 않고도 호출할 수 있습니다!**
즉, 해당 클래스 내부 정보 없이도 동작해야 하는 **유틸리티 함수**에 매우 적합합니다.

정적 함수는 다음과 같은 작업에 사용할 수 있습니다:

- 수학 계산 (기하, 물리 등)
    
- 절차적 레벨 데이터 생성
    
- 디렉터리 안의 모든 파일 나열
    
- 텍스트 처리 및 수정
    
- 그리고 그 외 다양한 기능들

GDScript에서 정적 함수를 정의하려면 함수 정의 앞에 static 키워드를 사용합니다.
다음은 특정 디렉토리에서 특정 파일 확장자(예: “.png”, “.mp3” 등)를 가진 모든 파일을 찾는 정적 함수의 예시입니다:

```gdscript
static func find_files_in_directory(directory_path: String, file_extension: String) -> Array[String]:
	var file_names := DirAccess.get_files_at(directory_path)
	var file_paths: Array[String] = []
	for file_name in file_names:
		if file_name.get_extension() != file_extension:
			continue
		file_paths.append(directory_path.path_join(file_name))
	return file_paths
```

이 함수는:

1. 주어진 디렉토리 경로에 있는 모든 파일 이름을 가져오고,
    
2. 지정된 확장자와 일치하는 파일만 필터링하여,
    
3. 전체 경로를 리스트에 추가한 후,
    
4. 그 리스트를 반환합니다.