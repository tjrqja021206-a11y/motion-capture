# motion-capture

동작 영상에서 YOLO Pose로 뽑은 스켈레톤(Keypoint) CSV와 17개 동작 구분 테스트 결과입니다.

## 파일 이름 규칙

`S01_A001_T01` = **S**ubject(사람) · **A**ction(동작) · **T**ake(촬영)

- `S01`, `S02` : 촬영 대상자 번호
- `A001` ~ `A017` : 동작 번호 (아래 표)
- `T01` / `T02` : 같은 동작을 **다른 방향**에서 찍은 영상

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

## 데이터 현황

| Subject | 영상 수 | 비고 |
|---|---|---|
| S01 | 32 | A016, A017은 `T01`만 있음. A009·A010·A012의 `T02`는 가로 영상 |
| S02 | 34 | 17개 동작 모두 `T01`/`T02`. A009·A010·A012·A016·A017의 `T02`는 가로 영상 |

- 모두 60fps, 영상 파일은 저장소에 올리지 않음
- S02_A010_T01은 누워 있는 구간에서 사람 검출이 일부 빠짐(검출률 89%), 나머지는 98~100%

## 폴더 구성

```
configs/motion_capture_action_classes.csv   # 동작 번호 ↔ 이름
scripts/
├── 46_extract_motion_capture_keypoints.py  # 영상 → 스켈레톤 CSV
├── 47_render_motion_capture_skeleton.py    # 스켈레톤을 영상 위에 그려 확인
└── 48_test_motion_capture_actions.py       # 17개 동작 구분 테스트
data/motion_capture_keypoints/
├── manifest_S01.csv, manifest_S02.csv      # 영상별 요약 (프레임 수, 해상도, 검출률, 신뢰도, 가로/세로)
├── S01/S01_A001_T01.csv ...
└── S02/S02_A001_T01.csv ...
reports/motion_capture/                     # 48번 결과 (요약, 영상별 예측, 혼동행렬)
```

## CSV 형식

한 행이 한 프레임이며 총 95컬럼입니다.

- 기본 정보 10개: `clip_id, frame_id, time_sec, subject, action_id, action_label, take, person_count, selected_box_conf, pose_detected`
- COCO 17관절 × 5개: `<관절>_x_px, <관절>_y_px, <관절>_x_norm, <관절>_y_norm, <관절>_conf`
  - `_px`: 픽셀 좌표, `_norm`: 영상 가로·세로로 나눈 좌표, `_conf`: 관절 신뢰도
  - 사람이 여러 명 잡힌 프레임은 박스 신뢰도가 가장 높은 1명만 기록

## 동작 구분 테스트 결과 (S01 + S02)

2초 창을 0.25초 간격으로 잘라 Random Forest로 학습하고, 영상 단위로 다수결한 정확도입니다.
우연히 맞을 확률은 약 6%(17개 중 1개)입니다.

| 테스트 | seq (좌표+각도) | robust (각도·높이, 좌우 무관) |
|---|---|---|
| 방향 교차 T01 → T02 | 19% | 13% |
| 방향 교차 T02 → T01 | 9% | 38% |
| 사람 교차 S02 → S01 | 53% | 59% |
| 사람 교차 S01 → S02 | 53% | 71% |

- 한 방향만 학습하면 다른 방향은 거의 못 맞힘 → 학습에 양방향을 모두 넣어야 함
- 잘 맞힘: 잡고 흔들기, 휘두르기, 바닥에 앉기
- 자주 틀림: 쪼그려 물건 빼내기(→ 앞으로 눕기), 샷건(→ 휘두르기), 키오스크 두드리기
- 사람이 2명뿐이라 아직 불안정함. S03…이 추가되면 48번이 한 명씩 빼고 자동으로 테스트함

## 실행 방법

스크립트는 `subject10_pose_action` 프로젝트 안에서 실행합니다
(그 프로젝트의 `src/`, `scripts/26_extract_keypoint_dataset.py`, `yolo11n-pose.pt`를 사용).
영상은 `data/motion_capture/S01/`처럼 Subject별 폴더에 둡니다.

```bash
.venv/bin/python scripts/46_extract_motion_capture_keypoints.py --subject S02
.venv/bin/python scripts/47_render_motion_capture_skeleton.py --subject S02
.venv/bin/python scripts/48_test_motion_capture_actions.py
```
