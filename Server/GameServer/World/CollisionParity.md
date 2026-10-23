# 월드 충돌 경계와 비교 계획

# ワールド衝突の境界と比較計画

> 공개 문서 범위 · 2026-10-23: 이번 게시 대상은 프로젝트 문서입니다. 아래 명령과 소스 경로는 개발 작업본 기준이며, 문서 게시가 제품 소스·실행 파일의 배포 또는 새 실행 검증을 뜻하지 않습니다. `Verification/`, `Saved/`, `.codex-workspace/`는 공개 저장소에 포함하지 않는 로컬 기록입니다. 과거 작업·승인·검증 상태는 당시 범위로 읽습니다.

> 公開文書の範囲 · 2026-10-23：今回の掲載対象はプロジェクト文書です。以下のコマンドとソースパスは開発作業コピーを基準とし、文書の掲載は製品ソース・実行ファイルの配布や新しい実行検証を意味しません。`Verification/`、`Saved/`、`.codex-workspace/` は公開リポジトリに含めないローカル記録です。過去の作業・承認・検証状態は当時の範囲として読みます。

이 선택적 서버 motor는 새 구현이다. 합성 기하 검사는 PhysX CharacterController 동등성, 전체 퀘스트, 영속 저장이나 세 encounter 시뮬레이션 전체를 인증하지 않는다. 기존 Unity 권위 서버를 전체 게임의 대체 경로와 비교 기준으로 유지한다.

この任意使用の server motor は新実装です。合成幾何検査は PhysX CharacterController との同等性、全クエスト、永続保存、3 encounter 全体の simulation を証明しません。既存 Unity 権威サーバーをゲーム全体の fallback と比較基準として維持します。

## 데이터 생성

## データ生成

격리된 Unity batch Editor에서 `--mode server`로만 실행한다.

隔離された Unity batch Editor で `--mode server` を指定した場合だけ実行します。

```
-executeMethod Mosslight.Editor.ServerWorldExport.Run
--server-world-export <unused-world-collision.json>
--server-world-export-report <unused-export-receipt.json>
```

exporter는 실제 게임의 세 지역 root를 만든다. 각 지역에서 enabled·active인 BoxCollider bounds를 캡처하고 닫힌 등불 봉인 하나를 `ruins.lantern-seal`로 내보내며, terrain/climb 정의와 controller profile을 캡처한 뒤 임시 객체를 제거한다. 회전/비box/비유한 collider와 trigger는 모양을 몰래 넓히지 않고 실패 처리한다. 소스·영수증 파일은 교체하지 않는다. Box ID는 지역/계층/이름별 출현 순서 문자열이며 Unity instance ID를 쓰지 않는다. Headless 모드로 renderer 전용 자식 객체에 영향을 받지 않는 식별자를 만든다. `sourceRevision`은 관련 소스 파일/설정을 해시하고 영수증은 정확한 JSON 바이트를 별도로 해시한다. 같은 우선순위의 지역은 첫 항목을 사용하므로 terrain 순서를 유지한다.

exporter は実ゲームの 3 region root を作ります。各 region の enabled・active な BoxCollider bounds を取得し、閉じた lantern seal 1 個を `ruins.lantern-seal` として出力し、terrain/climb 定義と controller profile を取得してから一時オブジェクトを破棄します。回転/非 box/非有限 collider と trigger は形状を黙って広げず、失敗させます。source と記録ファイルは置き換えません。Box ID は region/hierarchy/名前の出現順による文字列で、Unity instance ID は使いません。Headless mode により renderer 専用 child に左右されない識別子になります。`sourceRevision` は関連 source file/setting を hash し、記録は正確な JSON bytes を別途 hash します。同 priority の region は最初の entry を使うため、terrain 順序を維持します。

좌표는 기존 월드 공간의 미터 단위이며 X는 동쪽, Y는 위, Z는 북쪽이다. actor 위치는 transform 원점/발이며 capsule 중심은 별도의 profile offset이다. 월드 물리는 8/9 외 layer를 쓰고 trigger를 무시한다. gate bit는 그 충돌이 활성/닫힘이라는 뜻이다. CollisionWorld 구성 후 source DTO를 바꿔도 실행 중인 box는 움직이지 않는다. gate 상태는 클라이언트가 아닌 simulation tick이 소유한다.

座標は既存 world-space のメートルで、X は東、Y は上、Z は北です。actor position は transform origin/足元、capsule center は別の profile offset です。world physics は 8/9 以外の layer を使い、trigger を無視します。gate bit は collision が enabled/closed であることを示します。CollisionWorld 構築後の source DTO 変更では live box は動きません。gate state は client ではなく simulation tick が所有します。

## 이식 가능한 query와 motor

## 移植可能な query と motor

sphere-vs-box cast는 ray의 feature 구간에서 실제 모서리가 둥근 box까지의 제곱 거리를 푼다. AABB 확장은 broadphase일 뿐 모서리에서 충돌 판정의 답이 아니다. 수직 capsule은 capsule 중앙의 수직 선분만큼 확장한 box에 대한 sphere로 정확히 환원된다. Raycast는 원점을 포함하는 collider를 건너뛰며 기본 sphere/capsule sweep은 초기 overlap을 무시하므로 projectile 호출자는 명시적인 overlap query를 유지해야 한다. 진단용으로 `includeInitialOverlap: true`를 제공한다. overlap 검사는 엄격한 관통을 사용해 단순 접촉과 구분한다. 기본 query는 지역 내부로 제한하고 가장 가까운 hit를 선택하며, 같은 거리 hit는 export 순서를 유지한다.

sphere-vs-box cast は ray の feature 区間で、実際の丸い角を持つ box までの二乗距離を解きます。AABB expansion は broadphase だけで、角における衝突判定の答えではありません。垂直 capsule は、その中央の垂直線分だけ延長した box に対する sphere へ厳密に還元できます。Raycast は origin を含む collider を飛ばし、既定 sphere/capsule sweep は初期 overlap を無視するため、projectile 呼び出し側は明示的な overlap query を保持する必要があります。診断用には `includeInitialOverlap: true` が使えます。overlap は厳密な貫通で検査し、単なる接触と区別します。既定 query は region 内に限定し、最近接 hit を選び、同距離 hit は export 順序を保ちます。

motor는 연속적인 직립 capsule cast, 제한된 관통 복구(6), 제한된 slide 반복(6), 천장 검사를 포함한 step 시도와 실제 지지면 probe를 쓴다. Skin은 명시적인 반경/높이 여유다. 가파른 접촉 normal로 위쪽 변위를 만들어 step 제한을 피할 수 없다. 중력·점프·넉백·dash 부분 tick·stamina·수영 부력·등반 대상·HP·aggro·보상은 motor 밖의 actor 규칙이다. Teleport는 신뢰한 spawn/restore에만 쓰고 접지 상태를 초기화한다. boss의 기존 제한된 MoveTowards 돌진과 해석적 swept damage는 분리해서 유지해야 한다. boss는 원래 CharacterController를 쓰지 않았다.

motor は連続した直立 capsule cast、回数制限付き貫通回復（6）、回数制限付き slide（6）、天井検査付き step 試行、物理的な支持面 probe を使います。Skin は明示的な radius/height margin です。急な接触 normal による上方変位で step 制限を回避することはできません。重力・jump・knockback・dash 部分 tick・stamina・水泳浮力・climb target・health・aggro・reward は motor 外の actor rule です。Teleport は信頼する spawn/restore 専用で grounded を解除します。boss の既存 clamped MoveTowards charge と解析的 swept damage は分離して維持します。boss は元々 CharacterController を使っていません。

## 첫 비교 fixture

## 最初の比較 fixture

이동 동등성을 주장하기 전에 Editor fixture에서 기존 실제 RpgActor를 비어 있는 terrain/climb 목록과 변경하지 않은 30 Hz Tick 입력으로 만들어야 한다. 동일한 물리 box를 CollisionWorld로 내보내고 순서가 같은 delta/profile parameter를 재생한다. 권장하는 독립 비교 항목은 다음과 같다.

移動の同等性を主張する前に、Editor fixture は既存の実 RpgActor を空の terrain/climb list と不変の 30 Hz Tick 入力で作る必要があります。同じ物理 box を CollisionWorld に export し、同じ順序の delta/profile parameter を再生します。推奨する独立比較項目は以下です。

- 평평한 바닥: 안정화·idle·직선/대각선 걷기·정지·점프/낙하.
- 유한한 벽: 대각선 slide와 긴 dash. 한 tick의 이동이 벽을 통과하면 안 된다.
- 0.225m·0.30m step과 0.7m 벽을 양방향으로 비교.
- step 위 낮은 천장: 천장을 통과하지 않아야 한다.
- 실제 meadow의 0.005m 이음새와 bridge의 0.04m 이음새.
- 확장 AABB에만 걸리는 거짓 hit를 포함해 box 가장자리/모서리 옆의 sphere/capsule 궤적.
- 초기 overlap과 신뢰한 restore, 닫힘/열림/다시 닫힘 상태의 등불 봉인.
- 기존 ruins의 3.60m 상단 표면과 가로막히지 않은 지상 아래 통로.
- 후속 전체 맵 비교에서 lake shelf 진입/이탈·lava bridge의 높이 분리·양방향 climb 착지.

- 平らな床：安定化、idle、直線/斜め歩行、停止、jump/fall。
- 有限の壁：斜め slide と長い dash。1 tick の移動で通り抜けないこと。
- 0.225m・0.30m step と 0.7m 壁を両方向で比較。
- step 上の低い天井：天井を貫通しないこと。
- 実 meadow の 0.005m seam と bridge の 0.04m seam。
- 拡張 AABB だけに当たる偽 hit を含む、box の辺/角の横の sphere/capsule 軌道。
- 初期 overlap と信頼した restore、閉/開/再閉状態の lantern seal。
- 元の ruins の 3.60m 上面と、妨げられない地上の下側通路。
- 後の全 map 比較では lake shelf の入出、lava bridge の高さ分離、双方向 climb 着地。

각 입력 tick, actor 원점 위치, grounded flag, 적용 delta, profile/data SHA, 충돌 flag와 첫 접촉 거리/normal을 기록한다. 끝점만이 아니라 매 tick을 비교한다. 수치/지면 접촉 허용 오차는 후보 실행 전에 별도로 보존한 Unity 기준 관찰로 고정하고 실패 뒤 넓히지 않는다. 최초 작은 필드 runtime은 이 후속 동등성 행렬보다 의도적으로 좁은 범위다.

すべての input tick、actor-origin position、grounded flag、適用 delta、profile/data SHA、collision flag、初回接触 distance/normal を記録します。終点だけでなく各 tick を比較します。数値/地面接触の許容差は候補実行前に別途保存した Unity 基準観測から固定し、失敗後に広げません。初期の小規模 field runtime は、この将来の parity matrix より意図的に狭い範囲です。

`CollisionContractChecks.Run()`은 개별 Name/Passed/Detail 행을 반환하는 명시적인 BCL 전용 진입점이다. 서버를 구성할 때 자동 실행되지 않으며 socket·process·account·DB를 시작하지 않는다. 해석적 TOI 기대값과 topology 사례는 실제 Unity 비교를 보완할 뿐 대체하지 않는다.

`CollisionContractChecks.Run()` は個別 Name/Passed/Detail 行を返す、明示的で BCL のみを使う entry point です。server 構築時に自動実行されず、socket、process、account、database を開始しません。解析的 TOI 期待値と topology 事例は実 Unity 比較を補完するもので、代替ではありません。
