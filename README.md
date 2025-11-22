# DOP-Project
학과발표회 동아리 프로젝트

## 팀원
- 최윤호
- 노이수
- 김석희

## 0단계: 사전 준비 (Day 1 전반)

프로젝트 인프라를 구축하는 초기 단계. 

- **GitHub 저장소 생성**: main 브랜치와 각 팀별 브랜치(preprocessing, blob-analysis, candidate-generation) 생성
- **디렉토리 구조 설정**: `src/`, `data/raw/`, `data/processed/`, `tests/`, `notebooks/` 폴더 생성
- **Python 환경 세팅**: `requirements.txt` 작성 (opencv-python, numpy, matplotlib, scikit-learn)
- **공유 문서 작성**: 화면 키패드 좌표 매핑 (1~9, 0의 x,y 좌표), PIN 길이 규칙, 테스트 이미지 촬영 가이드라인 정의

## 1단계: 모듈별 설계 (Day 1 후반)

각 팀이 담당 모듈의 구조를 설계.

**A팀 (전처리 담당)**

- 반사 제거 알고리즘 조사 (`cv2.inpaint()` 사용법)
- 대비 증강 방법 실험 (CLAHE, high-pass filter)
- 입력: 원본 이미지 경로, 출력: 전처리된 이미지 파일 정의

**B팀 (얼룩 감지 담당)**

- Blob 감지 알고리즘 조사 (`cv2.SimpleBlobDetector`, `cv2.findContours()`, `cv2.connectedComponents()`)[opencv](https://opencv.org/blog/blob-detection-using-opencv/)
- Blob 특징 추출 계획 (중심 좌표, 면적, 강도)
- 입력: mask 이미지, 출력: 키별 점수 딕셔너리 포맷 설계

**C팀 (후보 생성 담당)**

- DFS/BFS 기반 PIN 조합 생성 알고리즘 설계
- 후보 점수 계산식 설계 (얼룩 강도 + 거리 가중치 + 방향성)
- 입력: 키별 점수 딕셔너리, 출력: Top-N PIN 후보 리스트 정의

## 2단계: 전처리 파이프라인 구현 (Day 2)

원본 이미지에서 스머지 마스크를 생성.

**A팀 주도, B/C팀 인터페이스 확인**

- **반사 제거**: `cv2.inpaint()` 사용하여 과도한 하이라이트 영역 제거
- **대비 강화**: CLAHE (Contrast Limited Adaptive Histogram Equalization) 적용
- **노이즈 필터링**: Gaussian blur + high-pass filter로 미세 얼룩 강조
- **마스크 생성**: Adaptive threshold + morphological operations (erosion, dilation)으로 이진 마스크 생성
- **저장**: `data/processed/` 폴더에 `<이미지명>_mask.png` 형태로 저장
- 함수 시그니처 예시: `preprocess_image(input_path, output_dir) -> mask_path`

**B팀/C팀**: 생성된 마스크 파일 포맷 확인, 샘플 데이터로 입력 테스트

## 3단계: 얼룩 분석 및 키 매핑 (Day 3)

마스크에서 얼룩을 감지하고 키패드에 매핑.

**B팀 주도, A팀 마스크 품질 검증**

- **Blob 감지**: `cv2.findContours()` 또는 `cv2.connectedComponentsWithStats()` 사용
- **특징 추출**: 각 blob마다 다음 계산
    - 중심 좌표 (centroid): `cv2.moments()` 사용
    - 면적 (area): contour 면적
    - 평균 강도 (intensity): 원본 이미지의 해당 영역 평균값
    - 방향성 (elongation): PCA 또는 회전 사각형 분석
- **키패드 매핑**: 각 blob 중심과 키패드 좌표(0~9) 간 최단 거리 계산, 각 키에 해당하는 blob들의 강도 합산
- **정규화**: 키별 점수를 0~1 범위로 정규화
- **출력 포맷**: JSON 또는 딕셔너리 형태 저장

`json{
  "1": 0.82,
  "2": 0.12,
  "3": 0.03,
  ...
}`

- 함수 시그니처 예시: `extract_blob_scores(mask_path, original_image_path, keypad_coords) -> dict`

**A팀**: 마스크와 원본 이미지 오버레이 시각화로 품질 확인

**C팀**: 후보 생성 알고리즘 골격 코드 작성

## 4단계: 후보 PIN 생성 (Day 4)

키별 점수를 기반으로 가능한 PIN 후보를 생성.

**C팀 주도, B팀 매핑 결과 검증**

- **키 필터링**: 점수 기준 상위 7~8개 키만 선택 (임계값 설정)
- **조합 생성**: DFS/BFS로 PIN 길이(예: 4자리 또는 6자리)에 맞는 모든 조합 생성
- **점수 계산**: 각 후보 PIN에 대해 다음 계산
    - 기본 점수: 각 키 점수의 합
    - 거리 가중치: 연속된 키 간 물리적 거리 (가까울수록 높은 점수)
    - 방향성 패널티: 급격한 방향 전환에 패널티 부여
- **정렬 및 출력**: 점수 기준 상위 N개 (예: Top-10 또는 Top-20) 후보 반환
- 함수 시그니처 예시: `generate_pin_candidates(key_scores, pin_length=4, top_n=10) -> list`

**B팀**: Blob 중심 좌표와 키 매핑 결과 시각화 (디버깅용)

**A팀**: 전처리 파이프라인 최종 튜닝 및 엣지 케이스 테스트

## 5단계: 통합 및 시각화 (Day 4 후반 ~ Day 5)

모든 모듈을 통합하고 시연용 자료를 준비.

**전체 팀 협업**

- **파이프라인 통합**: `main.py` 또는 `pipeline.py` 작성
    - 입력: 원본 이미지 경로
    - 출력: Top-N PIN 후보 + 시각화 이미지
- **시각화 구현**:
    - 원본 이미지 위에 감지된 blob 중심 표시 (원형 마커)
    - 각 키에 점수 텍스트 오버레이
    - Top-N 후보 PIN을 이미지 하단에 텍스트로 표시
- **Notebook 작성** (`notebooks/demo.ipynb`):
    1. 이미지 로드 및 전처리 결과 표시
    2. 마스크 생성 결과 비교 (전/후)
    3. Blob 감지 및 키 매핑 시각화
    4. 후보 PIN Top-N 리스트 출력
    5. 각 단계별 중간 결과 이미지 표시
- **테스트**: 5~10장의 다양한 조명/각도 이미지로 전체 파이프라인 검증

## 6단계: 문서화 및 발표 준비 (Day 5)

프로젝트 결과를 정리.

**전체 팀**

- **README.md 작성**: 프로젝트 개요, 설치 방법, 사용법, 각 모듈 설명
- **코드 정리**: Docstring 추가, 변수명 통일, 주석 정리
- **최종 발표 자료**: demo.ipynb 기반 시연, 성공/실패 케이스 분석, 정확도 평가 결과 포함
- **GitHub 정리**: 각 브랜치를 main에 병합, 최종 버전 태그 생성

**팀별 역할 요약**

- A팀: 전처리 코드 문서화, 파라미터 튜닝 가이드 작성
- B팀: Blob 분석 알고리즘 성능 분석 (검출률, 오검출률)
- C팟: 후보 생성 알고리즘 효율성 분석 (조합 수, 실행 시간)


위 계획은 추후 수정.
