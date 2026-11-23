# motion-capture

동작 영상에서 YOLO Pose로 뽑은 스켈레톤(Keypoint) CSV 데이터입니다.

## 파일 이름 규칙

`S01_A001_T01` = **S**ubject(사람) · **A**ction(동작) · **T**ake(촬영)

- `S01` : 촬영 대상자 번호
- `A001` ~ `A017` : 동작 번호 (아래 표)
- `T01` / `T02` / `T03` : 같은 동작을 **다른 방향**에서 찍은 영상 (`T03` = 뒷모습, 11/23 재촬영)

## 동작 목록

| 번호 | 동작 | action_label |
|---|---|---|
| A001 | 키오스크 두드리기 | kiosk_tap |
| A002 | 한손 물건 던지기 | throw_one_hand |
| A003 | 앞차기 | front_kick |
| A004 | 잡고 흔들기 | grab_shake |
| A005 | 밀치기 | push |
| A006 | 휘두르기 | swing |
| A007 | 바닥에 앉기 | sit_floor |
| A008 | 위에 앉기 | sit_on_object |
| A009 | 앞으로 눕기 | lie_forward |
| A010 | 뒤로 눕기 | lie_backward |
| A011 | 몸통 박치기 | body_slam |
| A012 | 쪼그려 물건 빼내기 | crouch_take_item |
| A013 | 샷건 | shotgun |
| A014 | 구토 | vomit |
| A015 | 옆으로 쓰러지기 | fall_sideways |
| A016 | 뒤로 넘어지기 | fall_backward |
| A017 | 앞으로 넘어지기 | fall_forward |

같은 내용이 [configs/motion_capture_action_classes.csv](configs/motion_capture_action_classes.csv)에 있습니다.

## S01 현황

- 영상 32개, 60fps (영상 파일은 저장소에 올리지 않음)
- A016, A017은 `T01`만 있음
- A009, A010, A012의 `T02`는 가로 영상, 나머지는 세로 영상
- 32개 모두 전체 프레임에서 사람 검출 성공

## S04 현황

- 영상 34개 (A001~A017 × `T01`·`T02` 모두), 60fps (영상 파일은 저장소에 올리지 않음)
- A009, A010, A012, A016, A017의 `T02`는 가로 영상, 나머지는 세로 영상
- 사람 검출: 31개는 전체 프레임 성공, A010_T02 99.6% · A016_T01 98.2% · A017_T01 97.0%
- `S04_A017_T02`는 뒤에 다른 사람이 함께 잡히는 프레임이 30%쯤 있음 (`single_person_ratio` 0.70) → 박스 신뢰도 1등만 기록되므로 학습 전에 확인 필요
- 같은 스크립트로 S01을 다시 뽑아 올라온 CSV와 비교했을 때 32개 모두 값이 같음 (같은 방법으로 만든 데이터)
- **11/23 뒷모습 `T03` 17개 추가** (A001~A017, 세로 영상) → 모두 51개
  - 사람 검출: 15개는 전체 프레임 성공, A010_T03 99.5% · A015_T03 98.7%
  - 뒤에서는 얼굴이 안 보여 평균 관절 신뢰도가 정면보다 낮음 (T03 0.72~0.80, T01 0.83~0.98)
  - 기존 34개는 다시 뽑아도 값이 같음 (최대 차이 0)

## 폴더 구성

```
configs/motion_capture_action_classes.csv   # 동작 번호 ↔ 이름
scripts/46_extract_motion_capture_keypoints.py
data/motion_capture_keypoints/
├── manifest_S01.csv       # 영상별 요약 (프레임 수, 해상도, 검출률, 신뢰도, 가로/세로)
├── manifest_S04.csv
├── S01/S01_A001_T01.csv   # 영상과 같은 이름, 32개
└── S04/S04_A001_T01.csv   # 51개 (T01 정면 · T02 측면 · T03 뒤)
```

## CSV 형식

한 행이 한 프레임이며 총 95컬럼입니다.

- 기본 정보 10개: `clip_id, frame_id, time_sec, subject, action_id, action_label, take, person_count, selected_box_conf, pose_detected`
- COCO 17관절 × 5개: `<관절>_x_px, <관절>_y_px, <관절>_x_norm, <관절>_y_norm, <관절>_conf`
  - `_px`: 픽셀 좌표, `_norm`: 영상 가로·세로로 나눈 좌표, `_conf`: 관절 신뢰도
  - 사람이 여러 명 잡힌 프레임은 박스 신뢰도가 가장 높은 1명만 기록

## 스켈레톤 다시 만들기

스크립트는 `subject10_pose_action` 프로젝트 안에서 실행합니다
(그 프로젝트의 `src/`, `scripts/26_extract_keypoint_dataset.py`, `yolo11n-pose.pt`를 사용).
영상은 `data/motion_capture/S01/`에 둡니다.

```bash
.venv/bin/python scripts/46_extract_motion_capture_keypoints.py --subject S01
```
