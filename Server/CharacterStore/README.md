# Mosslight 캐릭터 저장소

# Mosslight キャラクター保存サービス

> 공개 문서 범위 · 2026-10-23: 이번 게시 대상은 프로젝트 문서입니다. 아래 명령과 소스 경로는 개발 작업본 기준이며, 문서 게시가 제품 소스·실행 파일의 배포 또는 새 실행 검증을 뜻하지 않습니다. `Verification/`, `Saved/`, `.codex-workspace/`는 공개 저장소에 포함하지 않는 로컬 기록입니다. 과거 작업·승인·검증 상태는 당시 범위로 읽습니다.

> 公開文書の範囲 · 2026-10-23：今回の掲載対象はプロジェクト文書です。以下のコマンドとソースパスは開発作業コピーを基準とし、文書の掲載は製品ソース・実行ファイルの配布や新しい実行検証を意味しません。`Verification/`、`Saved/`、`.codex-workspace/` は公開リポジトリに含めないローカル記録です。過去の作業・承認・検証状態は当時の範囲として読みます。

Unity 권위 서버가 loopback 전용 .NET 10 서비스를 호출한다. 실제 캐릭터 상태는 MySqlConnector 2.6.2로 로컬 MySQL 8.4에 transaction으로 저장한다. 게임 클라이언트에는 service key나 DB 암호를 전달하지 않는다.

Unity 権威サーバーが loopback 専用の .NET 10 サービスを呼び出します。実キャラクター状態は MySqlConnector 2.6.2 を使い、ローカル MySQL 8.4 に transaction で保存します。ゲーム client に service key や DB パスワードは渡しません。

## 실행

## 実行

Docker Desktop을 설치·시작하고 .NET 10 SDK를 설치한다. 기존 publish 바이너리를 쓰면 그에 맞는 ASP.NET Core runtime을 준비한다. Windows PowerShell에서 다음을 실행한다.

Docker Desktop をインストール・起動し、.NET 10 SDK をインストールします。既存 publish binary を使う場合は対応する ASP.NET Core runtime を準備します。Windows PowerShell で次を実行します。

```powershell
powershell -NoProfile -File Tools/Start-CharacterStore.ps1 -Background
```

전용 label이 있는 `mosslight-mysql` 컨테이너와 `mosslight-mysql-data` volume만 생성·시작한다. MySQL은 `127.0.0.1:13316`, HTTP 서비스는 `127.0.0.1:17871`에만 bind한다. 기존 컨테이너·DB·사용자를 재사용하거나 변경하지 않는다. 소유하지 않은 같은 이름의 컨테이너/volume과 충돌하면 준비를 거부한다.

専用 label のある `mosslight-mysql` container と `mosslight-mysql-data` volume だけを作成・起動します。MySQL は `127.0.0.1:13316`、HTTP service は `127.0.0.1:17871` だけに bind します。既存 container・database・user は再利用・変更しません。所有しない同名 container/volume と競合すれば準備を拒否します。

`Server/CharacterStore/.local/config.json`에는 `serviceUrl`, `apiKey`, `connectionString`, `testConnectionString`이 들어간다. 이 파일과 나머지 로컬 비밀 파일은 Git에서 제외한다. 이 서버 디렉터리를 클라이언트 배포에 넣지 않는다. Unity 서버는 실행 시 이 설정을 읽는다.

`Server/CharacterStore/.local/config.json` は `serviceUrl`、`apiKey`、`connectionString`、`testConnectionString` を含みます。このファイルと他のローカル秘密ファイルは Git から除外します。この server directory を client 配布に含めません。Unity server は runtime にこの設定を読みます。

격리된 통합 테스트는 다음과 같이 실행한다.

隔離した統合テストは次のように実行します。

```powershell
powershell -NoProfile -File Tools/Start-CharacterStore.ps1 -Background -TestDatabase -Port 17873
```

테스트 서비스는 별도의 `mosslight_test` DB를 선택한다. `-NoDatabaseStart`는 HTTP만 시작하고 Docker를 그대로 둔다. ready 파일과 로그는 `.local` 아래에 쓴다. 실행기는 같은 key와 DB 모드의 정상 서비스를 재사용한다.

test service は別の `mosslight_test` database を選びます。`-NoDatabaseStart` は HTTP だけを起動し、Docker は変えません。ready file と log は `.local` 以下に書きます。launcher は同じ key・database mode の正常 service を再利用します。

## API

## API

아래의 공개 계정 경로 다섯 개는 `X-Mosslight-Key` 없이 계정 자격/session token을 받는다. `/auth/resolve`를 포함한 다른 모든 경로는 `X-Mosslight-Key`가 필요하다. 모든 캐릭터 요청에는 `playerId`(정규 GUID)와 `resumeToken`(32바이트 난수를 나타내는 64자리 16진수)이 들어간다. token은 MySQL에 SHA256 해시로 저장하며 URL query parameter로 받지 않는다.

以下の公開 account route 5 個は、`X-Mosslight-Key` なしで account credential/session token を受け取ります。`/auth/resolve` を含む他の全 route には `X-Mosslight-Key` が必要です。各 character request は `playerId`（正規 GUID）と `resumeToken`（32 byte の乱数を表す 64 桁 hexadecimal）を含みます。token は MySQL に SHA256 hash で保存し、URL query parameter では受け付けません。

| 경로 | 추가 요청 필드 | 결과 |
| --- | --- | --- |
| POST /session | 없음 | 새 신원 생성 또는 기존 신원 인증 |
| POST /character | 없음 | 인증한 캐릭터 조회 |
| POST /reward | eventId, enemyId, experience=10 | 슬라임당 물약 1개, 최초 검 1회, 10 XP·5 골드 |
| POST /boss-reward | eventId, enemyId=slime-king, experience=60 | 물약 2개, 60 XP·25 골드, 캐릭터당 Crown Charm 1개 |
| POST /item | requestId, action | usePotion / equipSword / unequipSword / equipCharm / unequipCharm |
| POST /quest | requestId, action | 아래에서 정한 첫 퀘스트와 고정 후속 action |
| POST /shop | requestId, action, expectedUpgrade | buyPotion / upgradeSword, 고정 서버 가격 |
| POST /save-vitals | requestId, hp, x, y, z | 제한 범위의 생명 상태/위치 저장 |
| GET /health | 없음 | MySQL 사용 가능 여부와 testDatabase flag |
| POST /shutdown | 없음 | HTTP 서비스만 정상 종료 |

| route | 追加 request field | 結果 |
| --- | --- | --- |
| POST /session | なし | 新 identity の作成、または既存 identity の認証 |
| POST /character | なし | 認証済み character の読込 |
| POST /reward | eventId, enemyId, experience=10 | slime ごとに potion 1 個、初回 sword 1 回、10 XP・5 gold |
| POST /boss-reward | eventId, enemyId=slime-king, experience=60 | potion 2 個、60 XP・25 gold、character ごとに Crown Charm 1 個 |
| POST /item | requestId, action | usePotion / equipSword / unequipSword / equipCharm / unequipCharm |
| POST /quest | requestId, action | 後述する最初の quest と固定後続 action |
| POST /shop | requestId, action, expectedUpgrade | buyPotion / upgradeSword、固定 server 価格 |
| POST /save-vitals | requestId, hp, x, y, z | 範囲を制限した vitals/position 保存 |
| GET /health | なし | MySQL availability と testDatabase flag |
| POST /shutdown | なし | HTTP service だけを正常停止 |

캐릭터 필드는 `characterId, level, experience, nextLevelExperience, maxHp, attackBonus, hp, potionCount, swordCount, swordEquipped, charmCount, charmEquipped, questStage, questSlimeKills, questChainStage, questBeaconMask, gold, swordUpgrade, revision, x, y, z`다. 성공 응답에는 `ok:true`와 `character`가 있으며 변경 응답에는 `duplicated`도 있다. 중복 요청은 현재 캐릭터를 반환하고 변경을 다시 실행하지 않는다. 같은 key를 다른 payload로 재사용하면 HTTP 409다. 잘못된 신원은 400/401, DB 사용 불가는 503이며 가짜 저장 성공을 반환하지 않는다.

character field は `characterId, level, experience, nextLevelExperience, maxHp, attackBonus, hp, potionCount, swordCount, swordEquipped, charmCount, charmEquipped, questStage, questSlimeKills, questChainStage, questBeaconMask, gold, swordUpgrade, revision, x, y, z` です。成功 response は `ok:true` と `character`、mutation response はさらに `duplicated` を含みます。重複 request は現在 character を返し、mutation を再実行しません。同じ key を異なる payload で再利用すると HTTP 409。不正 identity は 400/401、database 不可用は 503 で、架空の保存成功は返しません。

보상은 (playerId, kind, eventId)마다 유일하며 일반 보상은 kind `reward`, boss 보상은 `boss_reward`다. 권위 게임 서버는 완료한 boss 시도마다 유일한 event ID를 제공한다. 재시도로 XP·물약·charm을 더 받지 않는다. 모든 item/vitals 변경은 (playerId, kind, requestId)마다 유일하다. InnoDB 캐릭터 row lock이 서로 다른 동시 변경을 직렬화하며 캐릭터 갱신과 ledger 삽입은 같은 transaction에서 실행한다. Unity 서버는 플레이어별 작업 순서를 유지하고 단조 증가하는 revision을 적용해야 한다.

reward は (playerId, kind, eventId) ごとに一意で、通常 reward は kind `reward`、boss reward は `boss_reward` です。権威 game server が完了した boss attempt ごとに一意な event ID を渡します。retry で XP・potion・charm を追加取得しません。各 item/vitals mutation は (playerId, kind, requestId) ごとに一意です。InnoDB character row lock で異なる同時 mutation を直列化し、character 更新と ledger 挿入を 1 transaction にまとめます。Unity server は player ごとの操作順序を維持し、単調増加する revision を適用します。

level은 1에서 시작해 20까지다. 다음 level에는 currentLevel*20 XP가 필요하다. level-up은 HP를 전부 회복하며 max HP는 100+(level-1)*10, Crown Charm 장착 시 20을 더한다. 공격 보너스는 (level-1)*2에 검 장착 시 5+2*swordUpgrade, charm 장착 시 3을 더한다. charm 장착은 회복하지 않고 해제 시 줄어든 최대 HP로 현재 HP를 제한한다. 물약은 35를 회복하며 최대 HP 미만일 때만 소모한다. 소지 상한은 물약 9999개·검 1개·charm 1개다. 정당한 boss 재클리어는 60 XP와 물약 2개를 계속 주지만 charm은 1개로 유지한다. 새 캐릭터의 시작 위치는 (-8, 0.22, -3)이다.

level は 1 から始まり上限 20。level-up には currentLevel*20 XP が必要です。level-up で全回復し、max HP は 100+(level-1)*10、Crown Charm 装備中はさらに 20。attack bonus は (level-1)*2 に sword 装備時の 5+2*swordUpgrade、charm 装備時の 3 を加えます。charm 装備では回復せず、解除時は下がった最大値へ HP を制限します。potion は 35 回復し、満 HP 未満のときだけ消費します。inventory 上限は potion 9999 個、sword 1 個、charm 1 個。正当な boss 再攻略でも 60 XP と potion 2 個を支給し、charm は 1 個を維持します。新 character の spawn は (-8, 0.22, -3) です。

기존 로컬 bearer 신원은 변경 없이 계속 지원한다. 프로필 파일을 보존하면 같은 캐릭터로 직접 이어 하거나 명시적으로 계정에 연결할 수 있다. 캐릭터 소지품과 능력치는 서버 권위이며 클라이언트가 이 HTTP 서비스로 아이템을 지급할 수 없다.

既存ローカル bearer identity は不変のまま引き続き対応します。profile file を保存すれば、同じ character を直接再開するか明示的に account へ link できます。character inventory と stats はサーバー権威であり、client はこの HTTP service から item を支給できません。

## 백업과 다른 PC

## バックアップと別 PC

실제 MySQL 파일은 이 PC의 Docker named volume에 있다. 소스/config와 SQL 백업은 외장 프로젝트 드라이브에 있다.

MySQL の live file はこの PC の Docker named volume にあります。source/config と SQL backup は外付け project drive にあります。

```powershell
powershell -NoProfile -File Tools/Backup-LocalMySql.ps1
```

일관된 single-transaction SQL 백업과 SHA256 메타데이터를 `.local/backup`에 쓰며 기존 백업을 덮어쓰지 않는다. 새 PC에서는 Docker Desktop을 시작하고 `Setup-LocalMySql.ps1`를 실행해 새 빈 DB에 복원한 뒤 캐릭터 서비스를 시작한다.

整合した single-transaction SQL backup と SHA256 metadata を `.local/backup` に書き、古い backup は上書きしません。新 PC で Docker Desktop を起動し、`Setup-LocalMySql.ps1` を実行して新しい空 database へ restore 後、character service を起動します。

```powershell
powershell -NoProfile -File Tools/Setup-LocalMySql.ps1
powershell -NoProfile -File Tools/Restore-LocalMySql.ps1 -BackupPath "D:\path\mosslight-TIMESTAMP.sql"
powershell -NoProfile -File Tools/Start-CharacterStore.ps1 -Background
```

복원은 백업 메타데이터/해시를 검증하며 테이블이 하나라도 있는 대상에 덮어쓰기를 거부한다. 기존 캐릭터 DB를 가져오려면 복원 전에 새 서비스를 시작하지 않는다. `.local/config.json`과 클라이언트 신원 파일을 SQL 백업과 함께 보존한다. 이 도구는 DB 파일을 삭제하지 않는다.

restore は backup metadata/hash を検証し、既存 table が 1 個でもある対象への上書きを拒否します。既存 character database を import する場合は、restore 前に新 service を起動しません。`.local/config.json` と client identity file を SQL backup と一緒に保存します。これらのツールは database file を削除しません。

## 검증

## 検証

`TestCharacterStore.py`는 `mosslight_test`의 새 GUID 신원만 사용한다. 중복·서로 다른 보상의 동시 요청, 소지품 action, 인증 실패, request key 충돌, level 상한, 서비스 재시작, 실제 MySQL 중지/시작, DB 불가 실패, SQL 백업/복원 동등성과 덮어쓰기 거부를 검사한다. label이 있는 게임 소유 MySQL 컨테이너만 의도적으로 재시작하므로 실제 플레이 전에 실행한다. 게임·테스트 기록은 그대로 남긴다.

`TestCharacterStore.py` は `mosslight_test` の新 GUID identity だけを使います。重複/別 reward の同時実行、inventory action、認証失敗、request-key 競合、level cap、service 再起動、実 MySQL 停止/起動、database 不可用失敗、SQL backup/restore 一致、上書き拒否を検査します。label 付き game 所有 MySQL container だけを意図的に再起動するので、live play 前に実行します。game と test の記録はそのまま残します。

```powershell
python Server/CharacterStore/TestCharacterStore.py
```

직접 DB 테스트는 HTTP 포트 17872를 쓰고 결과를 `Verification/character-store-test-*`에 쓴다. Unity 두 클라이언트 테스트는 별도 HTTP 포트 17873을 쓴다.

直接 database test は HTTP port 17872 を使い、結果を `Verification/character-store-test-*` に書きます。Unity 2 client test は別 HTTP port 17873 を使います。

## Boss schema migration과 검증

## Boss schema migration と検証

시작할 때 schema별 MySQL advisory lock을 잡고 기본값 (0, false)의 charm 컬럼과 소유권 제약을 추가하며 HP 검사를 310까지 확장한다. 기존 캐릭터·소지품·기본키·mutation ledger는 그대로 둔다. 반복 시작으로 upgrade를 다시 적용하거나 초기화하지 않는다. upgrade 서비스의 첫 시작 전에 논리 백업을 만든다.

起動時に schema ごとの MySQL advisory lock を取り、既定値 (0, false) の charm column と所有制約を追加し、HP check を 310 まで拡張します。既存 character・inventory・primary key・mutation ledger は維持します。再起動しても upgrade の再適用・reset はしません。最初の upgrade service 起動前に logical backup を作ります。

`Tools/Test-BossStore.ps1`는 `mosslight_test`의 새 신원과 새 `mosslight_restore_*` schema를 쓴다. 이전 schema migration(또는 제공한 `-PreMigrationBackup`), 두 플레이어의 동시 중복 보상, 반복 boss 시도, charm 능력치와 장착 시 미회복, item 멱등성, level 상한 HP 310, 서비스/실제 MySQL 재시작과 SQL 백업/복원 동등성을 확인한다. 게임 소유 label 컨테이너만 재시작하며 이 프로젝트 게임 프로세스가 켜져 있으면 시작을 거부하므로 실제 플레이 전에 실행한다. 테스트 기록과 restore schema는 확인용으로 남긴다.

`Tools/Test-BossStore.ps1` は `mosslight_test` の新 identity と新 `mosslight_restore_*` schema を使います。旧 schema migration（または指定した `-PreMigrationBackup`）、2 player の同時重複 reward、反復 boss attempt、charm stats と装備時に回復しないこと、item 冪等性、level cap HP 310、service/実 MySQL 再起動、SQL backup/restore 一致を検査します。label 付き game 所有 MySQL container だけを再起動し、この project の game process が動作中なら起動を拒否するので、live play 前に実行します。test record と restore schema は点検用に残します。

## 첫 퀘스트: crown-court

## 最初の quest：crown-court

서버 소유 퀘스트 ID는 `crown-court`로 고정한다. 내부 `POST /quest` 요청에는 `playerId, resumeToken, requestId, action`이 들어가며 다른 경로와 같은 service key·신원 인증을 사용한다. 게임 서버가 NPC 상호작용과 거리를 확인한 뒤 호출한다. loopback 저장소는 퀘스트 진행과 변경 멱등성을 확인하지만 클라이언트가 주장하는 NPC 위치를 받거나 검증하지 않는다.

server 所有 quest の固定 ID は `crown-court`。内部 `POST /quest` request は `playerId, resumeToken, requestId, action` を含み、他 route と同じ service key・identity 認証を使います。gameplay server は NPC interaction と距離を検証後に呼びます。loopback store は quest progression と mutation 冪等性を検証し、client が申告する NPC position は受け取りも検証もしません。

| questStage | 의미 | questSlimeKills |
| --- | --- | --- |
| 0 | 미수락 | 0 |
| 1 | meadow 슬라임 사냥 | 0~2 |
| 2 | Slime King 처치 | 3 |
| 3 | NPC에게 보고 | 3 |
| 4 | 영구 완료 | 3 |

| questStage | 意味 | questSlimeKills |
| --- | --- | --- |
| 0 | 未受注 | 0 |
| 1 | meadow slime を狩る | 0～2 |
| 2 | Slime King を倒す | 3 |
| 3 | NPC に報告 | 3 |
| 4 | 永久完了 | 3 |

`acceptQuest`는 보상 없이 stage 0을 1로 바꾼다. 일반 `/reward` commit은 stage 1에서만 처치 수를 올리고 서로 다른 세 번째 처치가 stage 2로 전환한다. commit된 `/boss-reward`는 stage 2를 3으로 바꾼다. 이미 commit된 보상 재생은 퀘스트를 진행하지 않는다. 수락 전 처치/boss는 소급 인정하지 않는다. 일반·boss 보상의 기존 XP/item 규칙은 그 외에는 유지한다.

`acceptQuest` は reward なしで stage 0 から 1 へ変更します。通常 `/reward` commit は stage 1 だけで kill 数を増やし、異なる 3 体目の kill で stage 2 へ進みます。commit 済み `/boss-reward` が stage 2 から 3 へ進めます。既存 reward replay で quest は進みません。受注前の kill/boss を遡及計上しません。それ以外の通常/boss reward の XP/item rule は不変です。

`completeQuest`는 stage 3을 요구하며 한 transaction에서 stage 4 전환, 40 XP·20 골드·물약 2개 지급과 ledger 1행 삽입을 실행한다. ledger kind `quest`는 고정 퀘스트 ID와 action을 해시한다. 같은 request ID의 동시 재시도는 한 번 commit하고 현재 캐릭터를 반환한다. 같은 ID를 다른 action으로 쓰면 409다. 새 ID로 이미 수락한 퀘스트를 수락하거나 완료한 퀘스트를 다시 완료할 수 없다. 실패한 시도는 ledger 행과 보상을 추가하지 않는다.

`completeQuest` は stage 3 を要求し、stage 4 への変更、40 XP・20 gold・potion 2 個の付与、ledger 1 行挿入を原子的に実行します。ledger kind `quest` は固定 quest ID と action を hash します。同一 request ID の同時 retry は 1 回 commit して現在 character を返し、別 action への ID 再利用は 409。同じ quest の再受注/完了済み quest の再完了は新 ID でもできません。失敗試行で ledger 行や reward は追加しません。

시작할 때 기존 schema별 migration lock 아래 `quest_stage`, `quest_slime_kills`를 기본값 0으로 추가한다. 범위·조합 검사로 위 표를 강제한다. 기존 level·아이템·charm 장착·캐릭터 신원과 모든 mutation ledger는 보존한다.

起動時に既存 schema 別 migration lock の下で `quest_stage` と `quest_slime_kills` を既定 0 で追加します。範囲・組合せ check は上表を強制します。既存 level・item・charm 装備・character identity・全 mutation ledger はそのままです。

`Tools/Test-QuestStore.ps1`는 `mosslight_test`의 새 신원과 새 restore schema만 사용해 실제 MySQL에서 이 동작을 검사한다. 이전 schema migration, 수락/미수락 파티원, 부분 reload, 목표 전환, 동시 수락/완료 재시도 16건, 거부·재생, 서비스/실제 MySQL 재시작, SQL 백업/복원 동등성을 다룬다. label이 있는 게임 소유 MySQL 컨테이너만 재시작하므로 실제 플레이 전에 실행한다. 선택적 `-PreMigrationBackup`은 migration 검증용 이전 SQL 백업을 지정한다. 기존 DB·캐릭터는 삭제하지 않는다.

`Tools/Test-QuestStore.ps1` は `mosslight_test` の新 identity と新 restore schema だけを使い、実 MySQL でこの動作を検査します。旧 schema migration、受注/未受注 party member、partial reload、objective 遷移、16 件の同時受注/完了 retry、拒否/replay、service/実 MySQL 再起動、SQL backup/restore 一致を確認します。label 付き game 所有 MySQL container だけを再起動するので、live gameplay 前に実行します。任意 `-PreMigrationBackup` は migration 検証用の既存 SQL backup を選びます。既存 database と character は削除しません。

## 개인 후속 퀘스트 연쇄

## 個人の後続 quest chain

첫 퀘스트 필드는 그대로 둔다. 서버가 `questChainStage`, `questBeaconMask`를 추가하며 SQL 컬럼은 `quest_chain_stage`, `quest_beacon_mask`다. chain stage가 0보다 크려면 원래 `questStage=4`여야 한다. 게임 서버는 NPC/beacon action 전 현재 scene·거리·상호작용 가능 여부를 확인한다. loopback 저장소는 진행·인증·원자적 멱등성을 강제하며 클라이언트 제공 scene 좌표를 신뢰하지 않는다.

最初の quest field は不変です。server は `questChainStage` と `questBeaconMask` を追加し、SQL column は `quest_chain_stage` と `quest_beacon_mask` です。chain stage が 0 超なら元の `questStage=4` が必要です。gameplay server は NPC/beacon action 送信前に現在 scene・距離・interaction 可否を検証します。loopback store は progression・認証・原子的冪等性を強制し、client 提供 scene 座標は信頼しません。

| questChainStage | 의미 | questBeaconMask |
| --- | --- | --- |
| 0 | Rekindle 미수락, 첫 퀘스트 후 가능 | 0 |
| 1 | Rekindle: 서로 다른 meadow beacon 3개 점등 | 0~6 |
| 2 | Rekindle 보고 가능 | 7 |
| 3 | Rekindle 완료, Echo 가능 | 7 |
| 4 | Echo 수락: 새로운 Slime King 처치 | 7 |
| 5 | Echo 보고 가능 | 7 |
| 6 | Echo 영구 완료 | 7 |

| questChainStage | 意味 | questBeaconMask |
| --- | --- | --- |
| 0 | Rekindle 未受注、最初の quest 後に可能 | 0 |
| 1 | Rekindle：異なる meadow beacon 3 個を点灯 | 0～6 |
| 2 | Rekindle 報告可能 | 7 |
| 3 | Rekindle 完了、Echo が可能 | 7 |
| 4 | Echo 受注：新しい Slime King を倒す | 7 |
| 5 | Echo 報告可能 | 7 |
| 6 | Echo 永久完了 | 7 |

`acceptRekindle`은 stage 0을 1로 바꾼다. `lightBeacon0`, `lightBeacon1`, `lightBeacon2`는 각각 해당 bit를 설정한다. action에 beacon index가 있으므로 추가 HTTP 요청 필드는 받지 않는다. 점등 순서는 자유다. 이미 켠 beacon의 새 ID 요청은 거부하며 같은 commit ID 재시도는 추가 변경 없이 현재 상태를 반환한다. 서로 다른 세 번째 beacon이 stage 1을 2로 바꾼다. `completeRekindle`은 stage 2를 3으로 바꾸고 정확히 30 XP·15 골드·물약 2개를 한 transaction에서 지급한다.

`acceptRekindle` は stage 0 から 1 へ変更します。`lightBeacon0`、`lightBeacon1`、`lightBeacon2` は各 bit を設定します。action に beacon index を含むため追加 HTTP request field は受け付けません。点灯順序は自由です。点灯済み beacon への新 ID は拒否し、同じ commit 済み ID の retry は再変更せず現在 state を返します。異なる 3 個目の beacon が stage 1 から 2 へ進めます。`completeRekindle` は stage 2 から 3 へ変更し、正確に 30 XP・15 gold・potion 2 個を 1 transaction で付与します。

`acceptEcho`는 stage 3을 4로 바꾼다. 새로 commit한 `/boss-reward`만 stage 4를 5로 바꾸며 기존 60 XP·물약 2개의 boss 보상은 그대로다. Echo 수락 전 처치한 boss는 수락 후 재생을 포함해 소급 인정하지 않는다. `completeEcho`는 stage 5를 6으로 바꾸고 정확히 60 XP·30 골드·물약 3개를 준다. 미수락 파티원은 일반 boss 보상만 받고 chain은 진행하지 않는다.

`acceptEcho` は stage 3 から 4 へ変更します。新たに commit した `/boss-reward` だけが stage 4 から 5 へ進め、通常の 60 XP・potion 2 個の boss reward は不変です。Echo 受注前に倒した boss は、受注後の replay を含め遡及計上しません。`completeEcho` は stage 5 から 6 へ変更し、正確に 60 XP・30 gold・potion 3 個を付与します。未受注 party member は通常 boss reward だけを受け、chain は進みません。

모든 action은 ledger kind `quest`를 공유한다. 기존 `acceptQuest`/`completeQuest`는 원래 `crown-court|action` 해시를 유지해 schema upgrade 전 재시도를 보존한다. Rekindle은 `rekindle-lights|action`, Echo는 `crown-echo|action`을 해시한다. row lock이 서로 다른 action을 직렬화해 경합 중 별도 beacon bit가 유실되지 않게 한다. 진행·XP·아이템·mutation ledger 갱신은 원자적이다. 성공 payload는 한 번만 commit하고, 같은 ID의 다른 payload 또는 완료한 action의 새 ID는 HTTP 409다.

全 action は ledger kind `quest` を共有します。既存 `acceptQuest`/`completeQuest` は元の `crown-court|action` hash を維持し、schema upgrade 前の retry を保ちます。Rekindle は `rekindle-lights|action`、Echo は `crown-echo|action` を hash します。row lock で異なる action を直列化し、競合時の異なる beacon bit の喪失を防ぎます。progression・XP・item・mutation ledger 更新は原子的です。成功 payload は 1 回 commit し、同じ ID の異なる payload または完了 action への新 ID は HTTP 409 です。

시작 시 기존 schema advisory lock 아래 기본값 0의 두 컬럼과 SQL check 4개만 추가한다. check는 위 표와 첫 퀘스트 완료 선행 조건을 강제한다. 기존 캐릭터 신원·생명 상태·소지품·첫 퀘스트·revision·mutation ledger는 보존한다. upgrade 서비스 첫 시작 전에 백업하고, 실행기가 기존 정상 프로세스를 재사용하지 않도록 재시작 전에 옛 정상 서비스를 종료한다.

起動時に既存 schema advisory lock の下で、既定 0 の 2 column と SQL check 4 個だけを追加します。check は上表と最初の quest 完了前提を強制します。既存 character identity・vitals・inventory・元 quest・revision・mutation ledger は維持します。upgrade service の初回起動前に backup し、launcher が旧正常 process を再利用しないよう再起動前に旧 service を停止します。

`Tools/Test-QuestChainStore.ps1`는 새 테스트 신원과 새 restore schema로 실제 MySQL chain 테스트를 실행한다. additive migration·기존 행 보존, 선행 조건, beacon 순서·부분 reload, 동일 ID/서로 다른 bit 동시 변경, 정확한 완료 보상, 늦은 파티 수락, 옛 boss 재생, stage 순서, 서비스/MySQL 재시작, DB 불가 실패, SQL 제약과 백업/복원 동등성을 검증한다. label이 있는 게임 소유 MySQL 컨테이너만 재시작하고 이 프로젝트의 게임 프로세스가 열려 있으면 거부한다. 기존 행 보존 검사에는 데이터가 있는 upgrade 전 테스트 백업을 `-PreMigrationBackup`으로 제공한다. 테스트 기록·restore schema는 확인용으로 남긴다.

`Tools/Test-QuestChainStore.ps1` は新 test identity と新 restore schema で実 MySQL chain test を実行します。additive migration・旧行保存、前提、beacon 順序/partial reload、同 ID/異なる bit の同時 mutation、正確な完了 reward、遅い party 受注、旧 boss replay、stage 順序、service/MySQL 再起動、database 不可用失敗、SQL 制約、backup/restore 一致を検証します。label 付き game 所有 MySQL container だけを再起動し、この project の game process が開いていれば拒否します。旧行保存の検査にはデータ入り pre-upgrade test backup を `-PreMigrationBackup` で渡します。test record と restore schema は点検用に残します。

2026-10-20 실제 MySQL 검증에서 후속 chain **264/264**가 통과했다(`Verification/quest-chain-store-test-20261020-110255/overall.json`). 기존 테스트 캐릭터 48행과 ledger 3,282행 보존을 포함한다. 기존 첫 퀘스트 회귀 **121/121**도 통과했다(`Verification/quest-store-test-20261020-110437/overall.json`). 첫 퀘스트 fixture는 새 일회용 restore DB에 퀘스트 전 schema를 재구성할 때 의존하는 chain 컬럼도 제거하도록 했다. 첫 빈 운영 fixture 시도와 최초 합성 fixture 의존성 실패는 검증 기록에 남기며 최종 전체 suite 두 개는 모두 통과했다.

2026-10-20 の実 MySQL 検証で後続 chain **264/264** が通過しました（`Verification/quest-chain-store-test-20261020-110255/overall.json`）。既存 test character 48 行と ledger 3,282 行の保存を含みます。最初の quest の既存回帰 **121/121** も通過しました（`Verification/quest-store-test-20261020-110437/overall.json`）。元 quest fixture は新しい使い捨て restore database に pre-quest schema を復元する際、依存する chain column も除くようにしました。最初の空の本番 fixture 試行と元の合成 fixture 依存失敗は検証記録に残し、最終の完全 suite 2 個はともに通過しました。

## 골드·마을 상점·검 강화

## Gold・村の shop・sword 強化

캐릭터는 `gold`(0~999999), `swordUpgrade`(0~3)를 제공한다. 시작 시 같은 schema advisory lock 아래 기본값 0의 `gold`, `sword_upgrade` 컬럼과 잔액/등급/소유권 check를 추가한다. 기존 필드·revision·시각·mutation hash는 보존한다. 기존 성공 변경의 원래 payload hash를 유지하므로 옛 보상·완료 퀘스트 재생은 현재 캐릭터만 반환하고 골드를 소급 지급하지 않는다. 새 commit은 슬라임 5, Slime King 25, 첫 퀘스트 20, Rekindle 15, Echo 30 골드를 지급한다. 보상은 999999에서 제한해 지갑이 가득 차도 정당한 XP·아이템·진행을 막지 않는다.

character は `gold`（0～999999）と `swordUpgrade`（0～3）を公開します。起動時に同じ schema advisory lock の下で、既定 0 の `gold`・`sword_upgrade` column と balance/rank/ownership check を追加します。既存 field・revision・timestamp・mutation hash は保存します。既存成功 mutation の元 payload hash を維持するため、旧 reward/完了 quest replay は現在 character を返し、gold を遡及付与しません。新 commit は slime 5、Slime King 25、最初の quest 20、Rekindle 15、Echo 30 gold を付与します。reward は 999999 で clamp し、満額 wallet でも正当な XP・item・progression を妨げません。

`POST /shop`은 `playerId, resumeToken, requestId, action, expectedUpgrade`를 받는다. `buyPotion`은 `expectedUpgrade=0`을 요구하고 정확히 10 골드로 물약 1개를 추가한다. 소지품 상한/잔액 부족은 상태 변경 없이 409다. `upgradeSword`는 보유 검과 정확한 현재 `expectedUpgrade`를 요구하며 20·40·60 골드를 순서대로 지불해 영구 등급을 한 단계 올린다. 3등급은 더 강화하지 않는다. 기대값 누락/범위 밖과 알 수 없는 action은 400이다. 클라이언트는 가격·아이템 수·공격 능력치를 제공하지 못한다. 게임 서버는 내부 loopback 호출 전에 마을 scene·상인 거리·캐릭터 상태·작업 순서를 확인한다.

`POST /shop` は `playerId, resumeToken, requestId, action, expectedUpgrade` を含みます。`buyPotion` は `expectedUpgrade=0` を要求し、正確に 10 gold で potion 1 個を追加します。inventory 上限/残高不足は一切変更せず 409。`upgradeSword` は所有 sword と正確な現在 `expectedUpgrade` を要求し、20・40・60 gold の順で払い、永久 rank を 1 段階上げます。rank 3 はさらに強化できません。期待値欠落/範囲外、未知 action は 400。client は価格・item 量・attack stats を指定できません。gameplay server は内部 loopback 呼び出し前に村 scene・商人距離・character state・操作順を検証します。

장착한 검의 공격 기여는 `5 + 2*swordUpgrade`다. 해제해도 강화는 유지하고 검의 공격 기여만 전부 제거한다. 상점 ledger kind는 `shop`, 해시는 `shop|action|expectedUpgrade`다. row lock으로 결제·소지품/등급 갱신·ledger 삽입을 한 transaction에서 직렬화한다. 정확한 재시도는 한 번만 commit하고 같은 ID의 충돌 payload는 409다. 같은 기대 등급으로 새 ID 두 개가 경합해도 한 번만 강화하므로 오래된 클릭으로 다음 등급을 잘못 사지 않는다. 거부 요청은 골드를 차감하거나 ledger를 넣지 않는다.

装備 sword の attack 寄与は `5 + 2*swordUpgrade`。外しても upgrade は保持し、sword の寄与全体を除きます。shop ledger kind は `shop`、hash は `shop|action|expectedUpgrade`。row lock で課金・inventory/rank 更新・ledger 挿入を 1 transaction に直列化します。正確な retry は 1 回 commit、同じ ID の競合 payload は 409。異なる新 ID 2 個が同じ期待 rank で競合しても強化は 1 回だけで、古い click が次 rank を誤購入しません。拒否 request は gold を引かず ledger も挿入しません。

`Tools/Test-EconomyStore.ps1 -PreMigrationBackup PATH`는 실제 MySQL migration/보존, 골드 소급 지급 없는 과거 재생 5종, 고정 보상·가격, 장착 능력치, 모든 거부 경로, 동시 요청, 잔액/소지품/등급 상한, 신원 분리, 서비스/MySQL 재시작, DB 불가 재시도, SQL 제약·백업/복원 동등성을 확인한다. `mosslight_test`의 새 신원과 새 restore schema만 사용하며 경계 fixture도 그 새 테스트 신원만 바꾼다. 이 프로젝트의 플레이 중에는 거부하고 label이 있는 게임 소유 MySQL 컨테이너만 재시작한다.

`Tools/Test-EconomyStore.ps1 -PreMigrationBackup PATH` は実 MySQL migration/保存、gold の遡及付与がない旧 replay 5 種、固定 reward/価格、装備 stats、全拒否経路、同時 request、balance/inventory/rank cap、identity 分離、service/MySQL 再起動、database 不可用 retry、SQL 制約、backup/restore 一致を検査します。`mosslight_test` の新 identity と新 restore schema だけを使い、boundary fixture もその新 identity だけを変更します。project の gameplay 中は拒否し、label 付き game 所有 MySQL container だけを再起動します。

2026-10-20 검증에서 실제 MySQL 경제 **288/288**가 통과했다(`Verification/economy-store-test-20261020-113801/overall.json`). 골드 소급 지급 없는 과거 reward/quest 재생 5종을 모두 포함한다. 기존 개인 후속 chain 회귀 **264/264**도 통과했다(`Verification/quest-chain-store-test-20261020-113850/overall.json`). 초기 경제 검사는 ASP.NET strict-binding HTTP 400에 JSON 오류 본문이 있을 것으로 잘못 기대했다. HTTP 거부와 상태 불변은 올바랐고 최종 검사는 그 보장을 확인한다. 최종 전체 suite 두 개가 모두 통과했다.

2026-10-20 の検証で実 MySQL economy **288/288** が通過しました（`Verification/economy-store-test-20261020-113801/overall.json`）。遡及 gold なしの旧 reward/quest replay 5 種をすべて含みます。既存個人 chain 回帰 **264/264** も通過しました（`Verification/quest-chain-store-test-20261020-113850/overall.json`）。初期 economy test は ASP.NET strict-binding HTTP 400 に JSON error body があると誤って期待しました。HTTP 拒否と state 不変は正しく、最終 test はその保証を検査します。最終完全 suite 2 個はともに通過しました。

## Mosslight ruins 지역과 네 번째 개인 퀘스트

## Mosslight ruins 地域と 4 番目の個人 quest

캐릭터는 `zone`(`meadow` 또는 `ruins`, 기본 `meadow`), `ruinsQuestStage`, `ruinsKills`를 추가한다. SQL 컬럼은 `zone_id`, `ruins_quest_stage`, `ruins_kills`다. stage 0은 처치 수 0, stage 1은 0~3, stage 2(보고 가능)·3(완료)는 정확히 4여야 한다. ruins 퀘스트 수락 또는 ruins 위치 저장은 `questChainStage=6`을 요구한다. 시작 시 기존 schema lock 아래 세 컬럼·check 다섯 개만 추가하며 기존 캐릭터 필드·신원·revision·mutation hash는 보존한다.

character は `zone`（`meadow` または `ruins`、既定 `meadow`）、`ruinsQuestStage`、`ruinsKills` を追加します。SQL column は `zone_id`、`ruins_quest_stage`、`ruins_kills`。stage 0 は kill 0、stage 1 は 0～3、stage 2（報告可能）/3（完了）は正確に 4 を要求します。ruins quest の受注、または保存した ruins 位置には `questChainStage=6` が必要です。起動時に既存 schema lock の下で 3 column・5 check だけを追加し、以前の character field・identity・revision・mutation hash は維持します。

`POST /quest`는 Echo 완료 후 `acceptRuins`로 stage 0을 1로 바꾼다. `completeRuins`는 stage 2를 요구하며 stage 3 전환과 80 XP·40 골드·tonic 2개 지급을 원자적으로 실행한다. 새 action은 기존 kind `quest` 아래 `mosslight-ruins|action`을 해시한다. 기존 모든 quest hash는 바이트 단위로 그대로다. 수락 전 처치·옛 보상 재생은 진행을 소급 지급하지 않는다. 완료한 퀘스트는 새 ID로도 재지급되지 않는다.

`POST /quest` は Echo 完了後の `acceptRuins` で stage 0 から 1 へ変更します。`completeRuins` は stage 2 を要求し、stage 3 への変更と 80 XP・40 gold・tonic 2 個の付与を原子的に実行します。新 action は既存 kind `quest` で `mosslight-ruins|action` を hash します。全旧 quest hash は byte 単位で不変。受注前の kill と旧 reward replay は progression を遡及付与しません。完了 quest は新 ID でも再支給しません。

`POST /ruins-reward`는 일반 보상 형식 `playerId, resumeToken, eventId, enemyId, experience`를 사용하며 `enemyId=mossling` 또는 `wisp`, `experience=15`만 허용한다. 새로 commit한 처치는 정확히 15 XP·7 골드·tonic 1개를 주며 검은 주지 않는다. 수락한 stage 1 캐릭터만 진행하고 처치 수를 4에서 제한해 stage 2로 바꾼다. ledger kind는 `ruins_reward`, 해시는 `ruins|enemyId|15`다. 권위 게임 서버가 적 신원과 고유 encounter/event ID를 제공하며 클라이언트가 직접 요청하지 않는다.

`POST /ruins-reward` は通常 reward 形式 `playerId, resumeToken, eventId, enemyId, experience` を使い、`enemyId=mossling` または `wisp`、`experience=15` だけを許可します。新 commit の kill は正確に 15 XP・7 gold・tonic 1 個を付与し、sword は与えません。受注済み stage 1 character だけを進め、kill を 4 で cap して stage 2 にします。ledger kind は `ruins_reward`、hash は `ruins|enemyId|15`。権威 gameplay server が敵 identity と一意な encounter/event ID を渡し、client は直接送信しません。

`POST /travel`은 `playerId, resumeToken, requestId, zone`을 받는다. 목적지는 정확히 `meadow` 또는 `ruins`이며 ruins 진입은 Echo 완료와 저장 HP 양수를 요구한다. 새 전환은 HP와 모든 다른 능력치를 보존하고 안전 위치를 meadow (-8, .22, -3) 또는 ruins (-9, .22, -3)으로 정한다. 현재 zone의 새 요청은 409 `already_in_zone`이다. 정확한 재시도는 ledger kind `travel`과 `travel|zone`으로 현재 상태를 반환하며 다시 순간이동·회복하지 않는다. 같은 ID의 다른 목적지는 409다. 게임 서버는 이동 시작 전에 현재 vitals를 저장한다. `/save-vitals`와 기존 hash는 그대로다.

`POST /travel` は `playerId, resumeToken, requestId, zone` を含みます。目的地は正確に `meadow` または `ruins`。ruins 入場には Echo 完了と正の保存 HP が必要です。新 transition は HP と他の全 stats を保ち、安全な入口座標を meadow (-8, .22, -3)、ruins (-9, .22, -3) にします。現在 zone への新 request は 409 `already_in_zone`。正確な retry は ledger kind `travel` と `travel|zone` で現在 state を返し、再 teleport/回復はしません。同じ ID の異なる目的地は 409。gameplay server は travel 前に現在 vitals を保存し、`/save-vitals` と旧 hash は不変です。

`Tools/Test-RuinsStore.ps1 -PreMigrationBackup PATH`는 migration 보존, 과거 reward/quest/shop/vitals 재생 payload 7종, 선행 조건, 개인 수락·소급 인정 금지, 고정 전리품/완료 보상, 동일·다른 ID 동시 요청, 지역/HP 보존, SQL 제약, API/실제 MySQL 재시작, 불가 시 재시도·SQL 백업/복원 동등성을 확인한다. 새 테스트 신원 4개와 새 `mosslight_restore_*` DB만 사용한다. 플레이 프로세스와 겹치면 거부하고 label이 있는 게임 소유 MySQL 컨테이너만 재시작한다. 기존 quest/chain fixture는 이전 버전 재구성 때 새 일회용 schema에서만 의존 ruins 컬럼을 제거한다.

`Tools/Test-RuinsStore.ps1 -PreMigrationBackup PATH` は migration 保存、旧 reward/quest/shop/vitals replay payload 全 7 種、前提、個人受注/遡及禁止、固定 loot/完了 reward、同一/異なる ID の同時要求、region/HP 保存、SQL 制約、API/実 MySQL 再起動、outage retry、SQL backup/restore 一致を検証します。新 test identity 4 個と新 `mosslight_restore_*` database を使います。gameplay process と重複すれば拒否し、label 付き game 所有 MySQL container だけを再起動します。既存 quest/chain fixture は、旧 version 再構築時に新しい使い捨て schema だけで依存 ruins column を除きます。

2026-10-20 실제 MySQL ruins **279/279**가 통과했다(`Verification/ruins-store-test-20261020-122939/overall.json`). 기존 테스트 캐릭터 84행·ledger 5,158행과 과거 재생 7종 보존을 포함한다. 기존 경제 회귀 **288/288**(`Verification/economy-store-test-20261020-123037/overall.json`), 개인 후속 chain 회귀 **264/264**(`Verification/quest-chain-store-test-20261020-123126/overall.json`)도 통과했다. 세 suite 모두 API/MySQL 재시작·SQL 복원 검사를 완료했다. 테스트·restore 기록을 남기며 운영 fixture 기록은 수정하지 않았다.

2026-10-20 に実 MySQL ruins **279/279** が通過しました（`Verification/ruins-store-test-20261020-122939/overall.json`）。以前の test character 84 行・ledger 5,158 行と旧 replay 7 種の保存を含みます。既存 economy 回帰 **288/288**（`Verification/economy-store-test-20261020-123037/overall.json`）、個人 chain 回帰 **264/264**（`Verification/quest-chain-store-test-20261020-123126/overall.json`）も通過。3 suite とも API/MySQL 再起動と SQL restore 検査を完了しました。test/restore 記録は残し、本番 fixture record は変更していません。

## 계정 로그인과 선택적 기존 신원 가져오기

## Account login と任意の legacy import

계정 schema는 additive다. `accounts`, `account_sessions`는 자체 schema별 advisory migration lock 아래 생성한다. 기존 `characters`, `mutations` 행과 모든 legacy payload hash는 그대로다. 계정 하나의 활성 캐릭터는 하나이며 DB unique key로 같은 캐릭터를 두 계정이 소유하지 못하게 한다.

account schema は additive で、`accounts` と `account_sessions` を専用の schema 別 advisory migration lock の下で作ります。既存 `characters`・`mutations` 行と全 legacy payload hash は不変。1 account に active character は 1 人で、database unique key が同じ character を 2 account が所有することを防ぎます。

| 공개 POST 경로 | 정확한 JSON 요청 | 성공 응답 |
| --- | --- | --- |
| `/auth/register` | `username, password` | `ok, sessionToken, expiresUtc, characterId, username` |
| `/auth/login` | `username, password` | 가입과 같은 필드, 새 session token |
| `/auth/me` | `sessionToken` | `ok, expiresUtc, characterId, username`, token은 돌려주지 않음 |
| `/auth/logout` | `sessionToken` | `ok:true`, 반복/알 수 없는 logout도 변경 없는 성공 |
| `/auth/link` | `sessionToken, playerId, resumeToken` | 가져온 캐릭터에 대한 `/auth/me`와 같은 필드 |

| 公開 POST route | 正確な JSON request | 成功 response |
| --- | --- | --- |
| `/auth/register` | `username, password` | `ok, sessionToken, expiresUtc, characterId, username` |
| `/auth/login` | `username, password` | 登録と同じ field、新しい session token |
| `/auth/me` | `sessionToken` | `ok, expiresUtc, characterId, username`、token は再出力しない |
| `/auth/logout` | `sessionToken` | `ok:true`、反復/不明 logout も何もしない成功 |
| `/auth/link` | `sessionToken, playerId, resumeToken` | import character に対する `/auth/me` と同じ field |

username은 ASCII 영문·숫자·underscore 3~32자를 받으며 대소문자를 구분하지 않는다. password는 12~128자를 원문 그대로 보존한다. PBKDF2-SHA256 600,000회·16바이트 난수 salt·32바이트 hash를 사용하고 constant time으로 비교한다. 알 수 없는 사용자도 password derivation을 실행한 뒤 같은 `auth_failed` 오류를 준다. 중복 가입/잘못된 legacy 소유권은 일반화한 `account_unavailable`, 잘못된 DTO는 `invalid_request`다. 인증 응답에는 `Cache-Control: no-store`, `Pragma: no-cache`를 설정한다.

username は ASCII 英字・数字・underscore の 3～32 文字で、大文字小文字を区別しません。password は 12～128 文字を正確に保存します。PBKDF2-SHA256 600,000 回、16-byte random salt、32-byte hash を使い、constant time で比較します。不明 user も password derivation を実行後に同じ `auth_failed` error を返します。重複登録/不正 legacy ownership は汎用 `account_unavailable`、不正 DTO は `invalid_request`。認証 response は `Cache-Control: no-store` と `Pragma: no-cache` を設定します。

session은 32바이트 난수를 64자리 소문자 16진수로 표현하며 SHA256 hash만 저장한다. 기본 수명은 24시간이고 `accountSessionHours`로 1~168시간 범위에서 설정한다. 계정당 활성 session은 최대 8개이며 가장 오래된 것을 철회한다. logout은 해당 session을 즉시 철회한다. 정규화한 username별 분당 login 8회·가입 6회·legacy 가져오기 8회와 공통 분당 160회 제한이 있다. password derivation은 동시 4개로 제한한다. 제한 요청은 HTTP 429와 `Retry-After: 60`을 받는다. 예산은 프로세스 로컬이며 HTTP 재시작 시 초기화되므로 계정 API 인스턴스는 하나로 배포하고 PC 밖에 공개하면 HTTPS ingress의 자체 남용 방지도 유지한다.

session は random 32 byte を lowercase hexadecimal 64 桁で表し、SHA256 hash だけを保存します。既定寿命は 24 時間、`accountSessionHours` で 1～168 時間に設定でき、account ごとの active session は最大 8 個で最古を revoke します。logout は当該 session を即時 revoke します。正規化 username ごとに 1 分で login 8 回、登録 6 回、legacy import 8 回と、共有 160 回/分の予算があります。password derivation は同時 4 個に制限します。制限 request は HTTP 429 と `Retry-After: 60`。予算は process-local で HTTP 再起動時に reset するため、account API instance は 1 個で deploy し、PC 外へ公開するときは HTTPS ingress 自身の abuse control も維持します。

게임 권위 서버는 `{sessionToken}`과 service key로 내부 `/auth/resolve`를 호출한다. 응답은 `ok, playerId, resumeToken, characterId, username, expiresUtc`이며 신뢰한 서버에만 원시 legacy 캐릭터 자격을 돌려주는 유일한 계정 endpoint다. 외부 reverse proxy에서 `/auth/resolve`, `/session`, 모든 변경 경로, `/health`, `/shutdown`을 차단한다. service key·DB 설정·`/auth/resolve` 응답을 클라이언트 빌드에 넣지 않는다. 권위 서버는 session을 주기적으로 재검증하고 만료/철회된 연결을 끊어야 하며 클라이언트는 logout 시 로컬 로그인을 즉시 지운다.

game authority は `{sessionToken}` と service key で内部 `/auth/resolve` を呼びます。response は `ok, playerId, resumeToken, characterId, username, expiresUtc`。これは信頼する server だけに raw legacy character credential を返す唯一の account endpoint です。外部 reverse proxy では `/auth/resolve`、`/session`、全 mutation route、`/health`、`/shutdown` を遮断します。service key、database 設定、`/auth/resolve` response を client build に入れません。authority は account session を定期再検証し、expired/revoked session を切断します。client は logout 時に local login を直ちに消します。

legacy 가져오기는 검증된 원래 `playerId/resumeToken`과 인증된 계정 session이 필요하다. 계정의 새 캐릭터가 XP·아이템·통화·퀘스트 진행을 얻기 전에 한 번만 가능하다. 가져온 캐릭터·그 ledger·미사용 placeholder 행은 고치지 않으며 기존 profile 파일도 계속 사용할 수 있다. 활성 캐릭터 변경 시 다른 계정 session은 철회한다. 같은 성공 import의 반복은 멱등적이고 다른 캐릭터로 대체하려는 요청은 거부한다.

legacy import には検証済みの元 `playerId/resumeToken` と認証済み account session が必要です。account の新 character が XP・item・currency・quest progress を得る前に 1 回だけ可能です。import character、その旧 ledger、未使用 placeholder 行は変更せず、既存 profile file も使用可能です。active character 変更時は他 account session を revoke します。同じ成功 import の反復は冪等で、別 character への置換は拒否します。

계정의 캐릭터 재개 자격은 AES-256-GCM으로 암호화하며 12바이트 난수 nonce·별도의 16바이트 tag·계정/캐릭터에 묶인 추가 인증 데이터를 사용한다. 설정 `accountKey`로 안정적인 64자리 16진수 master key를 명시할 수 있다. 없으면 기존 `apiKey`에서 domain을 분리한 안정적인 HMAC-SHA256 key를 파생하며 시작 시 무작위 master key를 만들지 않는다. 다른 PC로 옮길 때 정확한 원래 `.local/config.json`을 SQL 백업과 함께 보존한다. fallback을 쓰는 중 `apiKey` 변경, 또는 계정 생성 후 `accountKey` 추가/변경은 암호화 key를 바꾼다. 시작은 일치하지 않는 key 식별자를 거부하고 ciphertext를 덮어쓰지 않는다. 두 key를 모두 보유한 명시적·검증된 decrypt/re-encrypt migration으로만 교체한다. 새 key를 추가하는 것만으로 migration이 되지 않는다.

account の character resume credential は AES-256-GCM で暗号化し、random 12-byte nonce、別の 16-byte tag、account/character に結び付けた追加認証データを使います。設定 `accountKey` で安定した 64 桁 hex master key を明示できます。ない場合は既存 `apiKey` から domain 分離した安定 HMAC-SHA256 key を導出し、起動時に random master key は生成しません。別 PC では正確な元 `.local/config.json` を SQL backup とともに保存します。fallback 利用中の `apiKey` 変更、account 作成後の `accountKey` 追加/変更は暗号鍵を変えます。起動時に不一致 key identifier を拒否し、ciphertext を上書きしません。両 key を使える状態で、明示的・検証済み decrypt/re-encrypt migration だけで rotation します。新 key を単に追加しても migration にはなりません。

`Tools/Test-AccountStore.ps1`는 별도 비공개 테스트 바이너리를 `.local/account-test-publish`에 publish하며 실행 중인 운영 API를 덮어쓰거나 재시작하지 않는다. `TestAccountStore.py`는 `mosslight_test`, 새로 restore한 계정 fixture schema와 loopback 테스트 API만 사용한다. 기존 행/ledger 보존, PBKDF2/암호화된 자격/hash session, 공개/내부 경계, 엄격 payload, 일반화 오류, brute-force 예산, 동시 가입/login/소유권/import 재시도, 만료/철회, 미사용 상태에서만 import, 안정 key HTTP 재시작, 위험 key 변경 거부, 전체 account/session/character/ledger SQL 백업/복원 동등성을 검증한다. 만료 검사는 새 테스트 session의 시각만 명시적으로 과거로 바꾸며 게임 진행은 주입하지 않는다. MySQL 컨테이너와 운영 HTTP API는 계속 실행한다.

`Tools/Test-AccountStore.ps1` は別の非公開 test binary を `.local/account-test-publish` へ publish し、稼働中本番 API の上書き/再起動はしません。`TestAccountStore.py` は `mosslight_test`、新 restore account fixture schema、loopback test API だけを使います。旧行/ledger 保存、PBKDF2/暗号化 credential/hash session、public/internal 境界、strict payload、汎用 error、brute-force 予算、同時登録/login/ownership/import retry、expiry/revocation、未使用限定 import、stable-key HTTP 再起動、危険 key 変更拒否、全 account/session/character/ledger の SQL backup/restore 一致を検証します。expiry test は新 test session の timestamp だけを明示的に過去へ変え、gameplay progression は注入しません。MySQL container と本番 HTTP API は稼働を続けます。

```powershell
powershell -NoProfile -File Tools/Test-AccountStore.ps1 -PreMigrationBackup "D:\path\mosslight_test-TIMESTAMP.sql"
```

## 영속적인 2인 던전 완료

## 永続的な 2 人 dungeon completion

신뢰하는 게임 권위 서버는 정확한 `{attemptId,participants:[{playerId,resumeToken},{playerId,resumeToken}]}`로 `POST /dungeon-complete`를 호출한다. attempt는 32자리 16진수 GUID이며 서로 다른 두 캐릭터 ID는 정규 GUID 형식이다. 두 자격을 모두 검증하며 새 완료 기록 시 두 캐릭터는 모두 `ruins`, `questChainStage=6`이어야 한다. 응답은 `{ok,attemptId,durable,completed,pendingCount,duplicated,members:null}`다. `durable:true` 성공은 header와 두 수신자가 MySQL에 함께 commit됐다는 뜻이다. 수신자 순서나 GUID 대소문자를 바꿔도 새 intent가 생기지 않으며 같은 attempt의 다른 수신자 조합은 409다. 정확한 재시도는 다른 곳으로 이동한 뒤에도 유효하다.

信頼する game authority は正確な `{attemptId,participants:[{playerId,resumeToken},{playerId,resumeToken}]}` で `POST /dungeon-complete` を使います。attempt は 32 桁 hex GUID、異なる 2 character ID は正規 GUID 形式です。両 credential を検証し、新 completion 記録時に両 character が `ruins` かつ `questChainStage=6` である必要があります。response は `{ok,attemptId,durable,completed,pendingCount,duplicated,members:null}`。`durable:true` 成功は header と両 recipient が MySQL に同時 commit した意味です。recipient 順序や GUID の大小文字で別 intent は作らず、同 attempt の別 recipient pair は 409。正確な retry は移動後も有効です。

`POST /dungeon-completion`은 기록된 member의 정확한 `{attemptId,playerId,resumeToken}`을 받는다. 응답은 `{ok,attemptId,durable,completed,pendingCount,duplicated,members:[{playerId,paid},{playerId,paid}]}`다. 상태 조회는 지급을 유발하지 않는다. 두 경로 모두 기존 loopback/service-key 보호가 필요하며 공개 reverse proxy에서 차단해야 한다. 게임은 `durable:true` 후에만 완료를 확정하고 `completed:true`를 기다린 뒤 `/character`로 각 캐릭터를 다시 읽는다.

`POST /dungeon-completion` は記録済み member から正確な `{attemptId,playerId,resumeToken}` を受け取り、`{ok,attemptId,durable,completed,pendingCount,duplicated,members:[{playerId,paid},{playerId,paid}]}` を返します。status read は payout を起こしません。両 route は既存 loopback/service-key 保護を要求し、public reverse proxy で遮断を維持します。game は `durable:true` 後だけ completion を確定し、`completed:true` を待って `/character` で各 character を再読込します。

새 additive 테이블은 `dungeon_completions`, `dungeon_completion_members`다. attempt/수신자 ID, 고정 payload hash와 지급 시각/상태를 저장하며 원시 resume token·암호화 자격·token hash는 추가로 저장하지 않는다. 각 수신자의 기존 `dungeon_reward` ledger, 고정 60 XP·25 골드·물약 2개·item 9·영구 탈것 지급과 member-paid 표식이 한 transaction에서 commit된다. HP는 회복하지 않는다. 원래 직접 `/dungeon-reward`는 같은 `mosslight-dungeon-v1|ruins-guardian|60|25|2|9` payload hash로 호환하며 이미 해당 ledger에 있는 지급은 확인만 하고 다시 지급하지 않는다.

新 additive table は `dungeon_completions` と `dungeon_completion_members`。attempt/recipient ID、固定 payload hash、payout timestamp/status を保存し、追加 raw resume token・暗号化 credential・token hash は保存しません。recipient ごとの既存 `dungeon_reward` ledger、固定 60 XP・25 gold・potion 2 個・item 9・永久 mount 付与、member-paid marker は 1 transaction で commit します。HP は回復しません。従来の直接 `/dungeon-reward` は同じ `mosslight-dungeon-v1|ruins-guardian|60|25|2|9` payload hash で互換を維持し、既存 ledger にある grant は確認するだけで再支給しません。

API는 listen 또는 ready 표식 쓰기 전에 대기 완료 작업을 drain한다. hosted worker는 매초 영속 작업을 재시도한다. 인증된 `/session`, `/character`도 상태 반환 전에 그 캐릭터의 대기 완료를 drain한다. 모든 복구 경로는 member·캐릭터보다 completion header를 먼저 잠그므로 서로 다른 store 프로세스가 같은 attempt를 복구해도 중복 보상하지 않는다. 한 수신자 commit 뒤 crash가 나면 다른 수신자는 pending으로 남는다. 전체 SQL DB를 다른 PC에 복원해도 의무가 유지되므로 백업에는 새 두 테이블과 기존 여덟 테이블을 함께 넣어야 한다.

API 起動時は listen/ready marker 書き込み前に pending completion を drain します。hosted worker は毎秒 durable work を retry します。認証済み `/session`・`/character` も state を返す前にその character の pending completion を drain します。全 recovery path は member/character より completion header を先に lock し、別 store process が同 attempt を recovery しても reward を重複しません。recipient 1 人の commit 後の crash は他方を pending に残します。完全な SQL database を別 PC に restore しても義務は残るため、backup は新 2 table と旧 8 table を一緒に含めます。

보장은 intent transaction이 commit될 때 시작한다. 등록 commit 전에 게임 권위 서버가 죽으면 복구할 영속 완료가 없다. 이것은 지급 복구이지 실행 중인 던전·적·파티·이동 상태 복구가 아니다. MySQL 검사는 새 restore fixture schema의 외부 SQL gate를 사용하고 소유한 테스트 API만 의도적으로 crash시킨다. 운영 fault-injection switch는 없다.

保証は intent transaction の commit 時に始まります。登録 commit 前に game authority が終了した場合、回復する durable completion はありません。これは payout recovery であり、稼働中 dungeon・enemy・party・movement state の復元ではありません。MySQL test は新 restore fixture schema の外部 SQL gate を使い、所有する test API だけを意図的に crash させます。本番 fault-injection switch はありません。
