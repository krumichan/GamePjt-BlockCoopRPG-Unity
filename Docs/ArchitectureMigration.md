# Unity / .NET 단계적 이전

# Unity / .NET の段階的移行

> 공개 문서 범위 · 2026-10-23: 이번 게시 대상은 프로젝트 문서입니다. 아래 명령과 소스 경로는 개발 작업본 기준이며, 문서 게시가 제품 소스·실행 파일의 배포 또는 새 실행 검증을 뜻하지 않습니다. `Verification/`, `Saved/`, `.codex-workspace/`는 공개 저장소에 포함하지 않는 로컬 기록입니다. 과거 작업·승인·검증 상태는 당시 범위로 읽습니다.

> 公開文書の範囲 · 2026-10-23：今回の掲載対象はプロジェクト文書です。以下のコマンドとソースパスは開発作業コピーを基準とし、文書の掲載は製品ソース・実行ファイルの配布や新しい実行検証を意味しません。`Verification/`、`Saved/`、`.codex-workspace/` は公開リポジトリに含めないローカル記録です。過去の作業・承認・検証状態は当時の範囲として読みます。

루트의 `mosslight.workspace.json`이 실행 도구의 경로 기준입니다. Unity 소스와 공용 패키지의 상대 구조를 유지하며 드라이브 문자를 소스에 고정하지 않습니다.

ルートの `mosslight.workspace.json` が実行ツールのパス基準です。Unity ソースと共通パッケージの相対構造を保ち、ソースにドライブ文字を固定しません。

```text
Client/Unity/                 Unity 클라이언트와 기존 전체 게임 권위 구현
Shared/com.mosslight.shared/  Unity UPM + netstandard2.1 공통 규칙·DTO·통신
Server/GameServer/            Unity 없이 실행하는 실험용 .NET 10 게임 서버
Server/CharacterStore/        .NET 10 계정·캐릭터 저장 API
Server/Deployment/            기존 운영 준비 자료
Tools/                       빌드·로컬 실행·검증
Builds/                      생성한 실행 파일 (소스 관리 제외)
LocalState/                  개인 캐릭터·인증 상태 (소스 관리 제외)
Verification/                로컬 검증 자료 (소스 관리 제외)
```

コード枠内の構成説明：`Client/Unity/` は Unity client と既存ゲーム全体の権威実装、`Shared/com.mosslight.shared/` は Unity UPM + netstandard2.1 の共通 rule・DTO・通信、`Server/GameServer/` は Unity なしで動く実験用 .NET 10 game server、`Server/CharacterStore/` は .NET 10 account・character 保存 API、`Server/Deployment/` は既存運用準備資料、`Tools/` は build・local 実行・検証です。`Builds/` の生成実行ファイル、`LocalState/` の個人 character・認証状態、`Verification/` の local 検証資料は source 管理から除外します。

## 현재 사용할 경로

## 現在使用する経路

전체 게임의 기본 실행은 기존 `Start-Account-Coop.cmd`와 Unity 전용 서버를 유지합니다. Unity 소스는 `Client/Unity`에 있으며 `Open-Unity.cmd`는 각 PC의 로컬 캐시 미러로 엽니다. Assets/Packages/ProjectSettings와 공용 패키지의 실제 파일은 외장 드라이브에 두고 Library 등 재생성 가능한 캐시는 각 PC의 C에 둡니다. 기존 캐시는 경로·소유·활성 Editor를 검사한 뒤 보존해 재사용합니다.

ゲーム全体の既定実行には、既存の `Start-Account-Coop.cmd` と Unity 専用サーバーを維持します。Unity ソースは `Client/Unity` にあり、`Open-Unity.cmd` は各 PC のローカルキャッシュミラーで開きます。Assets/Packages/ProjectSettings と共通パッケージの実ファイルは外付けドライブに置き、Library など再生成できるキャッシュは各 PC の C に置きます。既存キャッシュはパス・所有関係・稼働中 Editor を確認し、保存して再利用します。

독립 서버 프로젝트는 `Server/GameServer/Mosslight.GameServer.csproj`이며 `Tools/Build-DotNetServer.ps1`의 출력은 `Builds/DotNetServer`입니다. Unity Player·Engine DLL을 의존하지 않는 .NET 10 호스트입니다. `global.json`의 SDK 기준은 **10.0.401**이며 Unity 필수 버전은 **6000.3.26f1**입니다. 공용 프로젝트는 netstandard2.1이고 저장 API도 이를 참조합니다. .NET 중간 산출물은 공용 UPM 패키지 내부 대신 루트 `Artifacts/dotnet`에 둡니다.

独立サーバープロジェクトは `Server/GameServer/Mosslight.GameServer.csproj`、`Tools/Build-DotNetServer.ps1` の出力先は `Builds/DotNetServer` です。Unity Player・Engine DLL に依存しない .NET 10 ホストです。`global.json` の SDK 基準は **10.0.401**、Unity の必須バージョンは **6000.3.26f1** です。共通プロジェクトは netstandard2.1 で、保存 API もこれを参照します。.NET の中間生成物は共通 UPM パッケージ内ではなく、ルートの `Artifacts/dotnet` に置きます。

## 확인된 이전 결과

## 確認済みの移行結果

아래는 2026-10-23 경로·공용 코드 분리 후 확인한 단계별 결과입니다. 범위가 겹치는 검사 수를 합산하지 않습니다. 이후 수정된 .NET 코드의 실행 성공을 대신하는 증거로 사용하지 않습니다.

以下は、2026-10-23 にパス・共通コードを分離した後、段階別に確認した結果です。範囲が重複する検査件数は合算しません。その後変更された .NET コードの実行成功を代替する証拠には使いません。

| 범위 | 확정 결과와 증거 |
| --- | --- |
| 원본 보존 | 정적 감사26/26 (로컬 검증 자료: `Verification/architecture-source-baseline-audit-20261023.json`). 원본 소스 ZIP511개와 이전 전 Unity 전체407개 목록의 합집합547개를 모두 대조했습니다. 누락·의도하지 않은 변경0, 원본 메타197개 바이트·GUID 보존, 공용 C#19개와 메타19개 이동 확인, 기존 API C#12개 바이트 동일입니다. |
| 공용 패키지 | 실제 Unity 검사5/5 (로컬 검증 자료: `Verification/architecture-shared-20261023-104638-055-13340/shared-checks.json`). 컴파일된 공용 어셈블리에 Unity 엔진 참조가 없고, 기존 Unity 지형 판정을 사용하는 bridge oracle1701개 및 Unity JSON 연결을 대조했습니다. |
| 기존 규칙 | Editor320/320 (로컬 검증 자료: `Verification/architecture-rules-20261023-105351-038-47148/overall.json`). 기존 고정 검사 원본과 소스 전후 guard를 유지했습니다. |
| Unity 빌드 | Windows 클라이언트·Windows 서버·Linux 서버 (로컬 검증 자료: `Verification/architecture-unity-builds-20261023-110147-801-40812/overall.json`) 모두 정상 종료, 경고0·오류0입니다. Linux 실제 실행 검사는 포함하지 않습니다. |
| 저장 API | 회귀139/139 (로컬 검증 자료: `Verification/architecture-api-regression-20261023-1050/overall.json`), 정상 재시작 후37/37 (로컬 검증 자료: `Verification/architecture-api-restart-20261023-1055/overall.json`). 별도 테스트 API·테스트 DB의 검사이며 새 테스트 행의 API/SQL fixture와 실제 게임 입력 검사를 구분합니다. |
| 기존 Unity 두 창 | 186/186 (로컬 검증 자료: `Verification/architecture-unity-journey-20261023-120026-34068/overall.json`). 새 전사·지연 마법사의 정상 전투·스킬 포인트/패시브·리셋·중복/오래된 요청 거부·직업 보관·TLS 지연 재접속·새 Unity 권위와 두 창의 저장 필드/위치 복원을 확인했습니다. 실제 게임 프로세스6개 모두 exit0입니다. 모든 퀘스트·던전을 다시 수행한 검사는 아닙니다. |
| 독립 .NET 계약 | 충돌·코덱 등27/27 (로컬 검증 자료: `Verification/architecture-dotnet-contracts-20261023-1121.json`), 경계 검사6/6 (로컬 검증 자료: `Verification/architecture-dotnet-boundaries-20261023.json`). 기하·어셈블리·직렬화 계약 검사이며 실제 두 클라이언트 접속 성공이나 PhysX 전체 동등성을 뜻하지 않습니다. |
| 데스크톱 .NET | 새 소스42개 대조·실제 publish 및 계약27/27·TLS11/11 (로컬 검증 자료: `Verification/architecture-desktop-final-20261023.json`). 사용자 전용 SDK10.0.401로 실행했고 실제 작업 exit0, 컴파일 오류0·경고2(CS0649)입니다. TLS는 올바른 pin의 실제 접속·입력·스냅샷·정상 종료와 잘못된 pin 거부를 확인한 별도 loopback 계약입니다. 실제 Unity 두 창 필드 검사를 대신하지 않습니다. |
| 데스크톱 .NET 첫 필드 | 실제 Unity 두 창103/103 (로컬 검증 자료: `Verification/dotnet-desktop-field-run-20261023-1432/results/desktop-dotnet-field-journey-20261023-143609-5632/overall.json`). 새 전사2명의 메도우 이동·기본 공격·Leaf Whirl, 슬라임3마리 실제 사망에 따른 보상6건(각30XP·15골드), 반복 입력의 중복 보상 방지, TLS 지연 재접속과 새 .NET 호스트·새 두 창의31개 저장 필드·위치 복원을 확인했습니다. .NET 호스트2개·Unity 클라이언트4개 모두 exit0이며 기존 게임 검증 조건을 유지했습니다. |

| 範囲 | 確定結果と根拠 |
| --- | --- |
| 原本保存 | 静的監査 26/26（ローカル記録：`Verification/architecture-source-baseline-audit-20261023.json`）。原本ソース ZIP 511 件と移行前の Unity 全体 407 件の一覧の和集合 547 件をすべて照合。欠落・意図しない変更 0、原本メタ 197 件のバイト・GUID を保存し、共通 C# 19 件とメタ 19 件の移動、既存 API C# 12 件のバイト一致を確認しました。 |
| 共通パッケージ | 実 Unity 検査 5/5（ローカル記録：`Verification/architecture-shared-20261023-104638-055-13340/shared-checks.json`）。コンパイル済み共通アセンブリに Unity エンジン参照がなく、既存 Unity の地形判定を使う bridge oracle 1701 件と Unity JSON 接続を照合しました。 |
| 既存ルール | Editor 320/320（ローカル記録：`Verification/architecture-rules-20261023-105351-038-47148/overall.json`）。既存固定検査の原本とソース前後 guard を維持しました。 |
| Unity ビルド | Windows クライアント・Windows サーバー・Linux サーバー（ローカル記録：`Verification/architecture-unity-builds-20261023-110147-801-40812/overall.json`）がすべて正常終了し、警告 0・エラー 0。Linux の実行検査は含みません。 |
| 保存 API | 回帰 139/139（ローカル記録：`Verification/architecture-api-regression-20261023-1050/overall.json`）、正常再起動後 37/37（`Verification/architecture-api-restart-20261023-1055/overall.json`）。別のテスト API・テスト DB の検査であり、新規テスト行の API/SQL fixture と実際のゲーム入力検査を区別します。 |
| 既存 Unity の 2 ウィンドウ | 186/186（ローカル記録：`Verification/architecture-unity-journey-20261023-120026-34068/overall.json`）。新しい戦士・遅延させた魔法使いの通常戦闘、スキルポイント/パッシブ、リセット、重複/古い要求の拒否、クラス保存、TLS 遅延再接続、新 Unity 権威サーバーと 2 ウィンドウでの保存フィールド/位置復元を確認しました。実ゲームプロセス 6 個はすべて exit0。全クエスト・ダンジョンを再実行した検査ではありません。 |
| 独立 .NET 契約 | 衝突・コーデックなど 27/27（ローカル記録：`Verification/architecture-dotnet-contracts-20261023-1121.json`）、境界検査 6/6（`Verification/architecture-dotnet-boundaries-20261023.json`）。幾何・アセンブリ・シリアライズ契約の検査であり、実際の 2 クライアント接続成功や PhysX 全体の同等性を意味しません。 |
| デスクトップ .NET | 新ソース 42 件照合・実 publish・契約 27/27・TLS 11/11（ローカル記録：`Verification/architecture-desktop-final-20261023.json`）。ユーザー専用 SDK10.0.401 で実行し、実ジョブ exit0、コンパイルエラー 0・警告 2（CS0649）。TLS は正しい pin による実接続・入力・スナップショット・正常終了と、誤った pin の拒否を確認した別の loopback 契約です。実 Unity 2 ウィンドウのフィールド検査を代替しません。 |
| デスクトップ .NET 最初のフィールド | 実 Unity 2 ウィンドウ 103/103（ローカル記録：`Verification/dotnet-desktop-field-run-20261023-1432/results/desktop-dotnet-field-journey-20261023-143609-5632/overall.json`）。新戦士 2 人の meadow 移動・基本攻撃・Leaf Whirl、スライム 3 体の実死亡による報酬 6 件（各 30XP・15 ゴールド）、反復入力による二重報酬の防止、TLS 遅延再接続、新 .NET ホスト・新 2 ウィンドウでの 31 保存フィールド・位置復元を確認しました。.NET ホスト 2 個・Unity クライアント 4 個はすべて exit0 で、既存ゲームの検証条件を維持しました。 |

기존 Unity 여정의 첫 시도는 로그인 후 ‘계속’과 접속 명령을 서로 다른 비동기 파일에 동시에 발행해 주소창이 다시 열리는 순서 경합으로 중단됐습니다. 새 검증 어댑터에서 계정 명령 ACK와 로그인 창 닫힘을 먼저 기다리고 접속하도록 했으며, 기존 로그인·TLS·화면 검사는 유지했습니다. 제품 코드와 고정 원본 검증 도구는 이 수정으로 바꾸지 않았습니다. 수정 근거13/13 (로컬 검증 자료: `Verification/architecture-setup-order-fix-receipt-20261023.json`)과 원본 실패42/51 (로컬 검증 자료: `Verification/architecture-unity-journey-20261023-113318-41200/overall.json`)을 따로 보존합니다.

既存 Unity ジャーニーの最初の試行は、ログイン後の「続行」と接続命令を別々の非同期ファイルへ同時発行し、アドレスダイアログが再度開く順序競合で中断しました。新しい検証アダプターではアカウント命令 ACK とログイン画面の閉鎖を先に待ってから接続し、既存のログイン・TLS・画面検査を維持しました。この修正で製品コードと固定原本の検証ツールは変更していません。修正根拠 13/13（ローカル記録：`Verification/architecture-setup-order-fix-receipt-20261023.json`）と原本失敗 42/51（`Verification/architecture-unity-journey-20261023-113318-41200/overall.json`）は別々に保存します。

## 독립 서버의 첫 이전 범위와 미완료 항목

## 独立サーバーの最初の移行範囲と未完了項目

첫 필드 구현은 실험용입니다. 원래 메도우 섬의 전사2명, 이동·점프·회피·기본 공격·Leaf Whirl·슬라임3마리·보상·저장·재접속부터 이전하는 범위입니다. 실제 Unity 맵3개의 BoxCollider와 컨트롤러4종을 데이터로 추출했지만 세 맵의 모든 플레이 이전을 완료한 것은 아닙니다. 새 충돌 motor의 PhysX 완전 동등성도 확인하지 않았습니다.

最初のフィールド実装は実験用です。元の meadow 島の戦士 2 人、移動・ジャンプ・回避・基本攻撃・Leaf Whirl・スライム 3 体・報酬・保存・再接続から移行する範囲です。実 Unity マップ 3 個の BoxCollider とコントローラー 4 種をデータ抽出しましたが、3 マップの全プレイ移行が完了したわけではありません。新衝突 motor と PhysX の完全な同等性も確認していません。

**데스크톱에서 새 .NET 서버와 실제 Unity 두 창의 첫 필드 검증103/103 (로컬 검증 자료: `Verification/dotnet-desktop-field-run-20261023-1432/results/desktop-dotnet-field-journey-20261023-143609-5632/overall.json`)을 통과했습니다.** 두 새 전사의 실제 이동·기본 공격·Leaf Whirl과 슬라임3마리 사망 기록을 보상6건에 대조했고, 각30XP·15골드를 받았습니다. 반복 공격·스킬 카운터가 추가 보상을 만들지 않았으며, 지연 상태 재접속과 새 .NET 호스트·새 Unity 클라이언트2개의 저장31필드·권위/표시 위치 복원도 확인했습니다. 방향별100~150ms 설정의 TLS 지연·분할을 적용했고 프록시의 요청 지연 표본은100~149ms, 평균125.42ms입니다. 이는 실제 종단 간 왕복 지연 측정값이 아닙니다. 원래 게임 판정 조건을 유지한 별도 데스크톱 환경 어댑터의 결과이며 과거 검사를 통과 수에 합산하지 않았습니다.

**デスクトップで新 .NET サーバーと実 Unity 2 ウィンドウの最初のフィールド検証 103/103 が通過しました**（ローカル記録：`Verification/dotnet-desktop-field-run-20261023-1432/results/desktop-dotnet-field-journey-20261023-143609-5632/overall.json`）。新戦士 2 人の実移動・基本攻撃・Leaf Whirl とスライム 3 体の死亡記録を報酬 6 件に照合し、各 30XP・15 ゴールドを受け取りました。反復攻撃・スキルカウンターは追加報酬を作らず、遅延状態での再接続、新 .NET ホスト・新 Unity クライアント 2 個での保存 31 フィールド・権威/表示位置の復元も確認しました。方向ごとに 100～150ms 設定の TLS 遅延・分割を適用し、プロキシの要求遅延標本は 100～149ms、平均 125.42ms です。実際の端から端までの往復遅延測定値ではありません。元のゲーム判定条件を維持する別のデスクトップ環境アダプターの結果であり、過去の検査件数は合算していません。

이전 30/39 실패 (로컬 검증 자료: `Verification/dotnet-field-journey-20261023-121708-48564/overall.json`)는 두 TLS 클라이언트 접속을 기다리다 중단된 기록으로 보존합니다. 더 이른 24/33 실패 (로컬 검증 자료: `Verification/dotnet-field-journey-20261023-112438-32864/overall.json`)는 새 호스트의 실행 기록에 `connectionPort`가 빠진 검증 도구 문제였으며 한 필드만 보완하고 원본을 보존했습니다.

以前の 30/39 失敗（ローカル記録：`Verification/dotnet-field-journey-20261023-121708-48564/overall.json`）は、TLS クライアント 2 個の接続待ちで中断した記録として保存します。さらに前の 24/33 失敗（`Verification/dotnet-field-journey-20261023-112438-32864/overall.json`）は、新ホストの実行記録に `connectionPort` がない検証ツールの問題で、1 フィールドだけ補い原本を保存しました。

Windows .NET 호스트에만 인증서 키 가져오기 `UserKeySet`을 명시했고 기존 Unity의 두 인자 호출은 `EphemeralKeySet` 기본값을 유지합니다. 데스크톱의 새 TLS 계약11/11과 이번 첫 필드는 통과했으며, 계약 환경에서 기존 Ephemeral 호출도 동작했다는 관측은 설명용입니다. 노트북의 원래 TLS 실패 원인을 확정하거나 다른 PC·전체 기능의 접속을 보장하는 결과는 아닙니다. 최초 데스크톱9/11 실패는 서버 쪽 TLS 실패 카운터가 반드시 증가한다는 검사 가정을 바로잡아 원본과 별도 보존했습니다. 이후 Unity 재빌드는 하지 않았고 위 Unity 빌드·186/186은 변경 전 바이너리의 결과입니다.

Windows .NET ホストだけで証明書鍵の取り込みに `UserKeySet` を明示し、既存 Unity の 2 引数呼び出しは既定の `EphemeralKeySet` を維持します。デスクトップの新 TLS 契約 11/11 と今回の最初のフィールドは通過しましたが、契約環境で既存 Ephemeral 呼び出しも動作した観測は説明用です。ノート PC の元の TLS 失敗原因の確定や、他 PC・全機能の接続を保証する結果ではありません。最初のデスクトップ 9/11 失敗は、サーバー側 TLS 失敗カウンターが必ず増加するという検査上の仮定を修正し、原本と別に保存しました。その後 Unity の再ビルドはしておらず、前記 Unity ビルド・186/186 は変更前バイナリの結果です。

실험 호스트는 계정 TLS 접속, 명시한 새 테스트 캐릭터 최대2명, loopback, `testDatabase=true`인 API를 요구합니다. 운영 API로 실행하지 않습니다. 기존 진행 캐릭터·마법사·장비 조작·퀘스트·던전·파티·등반·수영·버프 등의 이전은 후속 단계이며 지원하지 않는 입력·지역은 제한합니다. 기존 전체 기능과 데이터는 기본 Unity 권위 서버에서 계속 사용합니다.

実験ホストにはアカウント TLS 接続、明示した新規テストキャラクター最大 2 人、loopback、`testDatabase=true` の API が必要です。本番 API では実行しません。既存進行キャラクター・魔法使い・装備操作・クエスト・ダンジョン・パーティー・登攀・水泳・バフなどの移行は後続段階とし、未対応入力・地域は制限します。既存の全機能とデータは既定の Unity 権威サーバーで引き続き使います。

다음 순서는 기존 Unity 권위와 첫 필드의 입력·충돌·전투 판정 비교, 지형·등반·원거리 직업, 퀘스트·파티·던전·성장 기능 이전입니다. 기본 서버 전환은 각 범위의 동등성·저장 검증 후에 합니다.

次の順序は、既存 Unity 権威サーバーと最初のフィールドの入力・衝突・戦闘判定比較、地形・登攀・遠距離クラス、クエスト・パーティー・ダンジョン・成長機能の移行です。既定サーバーの切り替えは各範囲の同等性・保存検証後に行います。

## 저장·개인 데이터·Git

## 保存・個人データ・Git

API의 실제 `.local`, `publish`, 계정 키와 설정 위치 및 MySQL 볼륨은 유지합니다. 이번 폴더 이전을 이유로 기존 DB를 복원하거나 초기화하지 않습니다. 운영 API는 유지하고 별도 publish의 테스트 API로 검사합니다. MySQL은 로컬 Docker 볼륨이므로 외장 소스만 연결해서 다른 PC의 DB가 자동 연결되지는 않습니다. 데이터 이전에는 별도 백업·복원 계획이 필요합니다.

API の実際の `.local`、`publish`、アカウント鍵・設定場所と MySQL volume は維持します。今回のフォルダー移行を理由に既存 DB を復元・初期化しません。本番 API を維持し、別 publish のテスト API で検査します。MySQL はローカル Docker volume のため、外付けソースを接続するだけで別 PC の DB に自動接続するわけではありません。データ移行には別のバックアップ・復元計画が必要です。

공개 소스에는 `.local`, LocalState, 캐시, 실행 파일, SQL 덤프, 개인 키를 넣지 않습니다. 현재 사용자 지시에 따라 **추가 stage/add·commit·원격 설정·push·업로드를 하지 않습니다.** 그 지시 전에 만든 이전 전 로컬 안전 백업 커밋1개는 그대로 보존했습니다. 추가 Git 게시나 저장소 분리는 이 작업의 완료 조건으로 삼지 않습니다.

公開ソースには `.local`、LocalState、キャッシュ、実行ファイル、SQL ダンプ、秘密鍵を入れません。当時のユーザー指示に従い、**追加 stage/add・commit・remote 設定・push・アップロードは行いません。** その指示前に作成した移行前のローカル安全バックアップコミット 1 個は保存しました。追加 Git 公開やリポジトリ分離を、この作業の完了条件にはしません。

## 다음 실행은 데스크톱 우선

## 次回の実行はデスクトップを優先

다음 기동·빌드·검증부터는 [중앙 데스크톱 운영 지침](https://github.com/krumichan/CatPjt-TranslaCat-docs/blob/main/repositories/shared/docs/desktop-workloads.md)과 프로젝트별 준비 상태 (로컬 검증 자료: `project-cat/.codex-workspace/verification/shared/desktop-workloads-20261023/projects.json`)를 먼저 확인합니다. 코딩과 원본은 노트북에서 유지하고, 무거운 실행은 공용 잠금과 프로젝트별 어댑터를 사용하는 데스크톱 경로를 우선합니다. 현재 기능 개발을 중단하거나 완료한 과거 검사를 자동 재실행하라는 방침은 아닙니다.

次回の起動・ビルド・検証からは、[中央デスクトップ運用指針](https://github.com/krumichan/CatPjt-TranslaCat-docs/blob/main/repositories/shared/docs/desktop-workloads.md)と、プロジェクト別準備状態（ローカル設定：`project-cat/.codex-workspace/verification/shared/desktop-workloads-20261023/projects.json`）を先に確認します。コーディングと原本はノート PC で維持し、重い実行には共通ロックとプロジェクト別アダプターを使うデスクトップ経路を優先します。現在の機能開発を止めたり、完了済みの過去検査を自動再実行したりする方針ではありません。

위 표의 데스크톱 .NET 계약과 첫 필드 행이 새 원격 실행 결과이며, 기존 Unity·API 검증은 로컬 기록입니다. 데스크톱 사용자 전용 **.NET SDK10.0.401**과 `Tools/Invoke-DesktopDotNet.ps1`로 실제 publish·27개 계약·11개 TLS 계약을 실행하고 생성한 publish10개 파일의 해시를 대조해 회수했습니다. 이전 publish도 별도 보존했습니다. 계약 실행 결과 (로컬 검증 자료: `Verification/architecture-desktop-final-20261023.json`)를 실제 첫 필드 검사와 구분합니다.

上表のデスクトップ .NET 契約と最初のフィールドの行が新しいリモート実行結果であり、既存 Unity・API 検証はローカル記録です。デスクトップユーザー専用 **.NET SDK10.0.401** と `Tools/Invoke-DesktopDotNet.ps1` で実際の publish・契約 27 件・TLS 契約 11 件を実行し、生成した publish 10 ファイルのハッシュを照合して回収しました。以前の publish も別に保存しました。契約実行結果（ローカル記録：`Verification/architecture-desktop-final-20261023.json`）は実際の最初のフィールド検査と区別します。

첫 필드는 `Tools/Invoke-DesktopInteractive.ps1`이 같은 사용자의 대화형 세션2에 예약 작업을 만들고, 공통 실행기가 작업 잠금을 보유한 채 실제 워커와 예약 작업의 종료까지 기다리는 방식으로 실행했습니다. PowerShell7과 .NET 런타임 경로를 프로세스 환경에 명시 전달하고 Python3.12.10을 사용했습니다. 실제 Windows 창2개가 각 초기/복원 실행에서 서로 다른 PID로 보였고, .NET 호스트2개·Unity 클라이언트4개가 모두 정상 종료됐습니다. 예약 작업도 종료 코드0·남은 인스턴스0을 확인한 뒤 비활성화했으며 강제 종료는 없었습니다. 대화형 실행 결과 (로컬 검증 자료: `Verification/dotnet-desktop-field-run-20261023-1432/results/interactive-adapter-result.json`).

最初のフィールドは、`Tools/Invoke-DesktopInteractive.ps1` が同じユーザーの対話セッション 2 に予約タスクを作り、共通実行器がジョブロックを持ったまま実 worker と予約タスクの終了まで待つ方式で実行しました。PowerShell7 と .NET runtime パスをプロセス環境へ明示的に渡し、Python3.12.10 を使用しました。実 Windows ウィンドウ 2 個は初回/復元の各実行で異なる PID として表示され、.NET ホスト 2 個・Unity クライアント 4 個はすべて正常終了しました。予約タスクも終了コード 0・残存インスタンス 0 を確認後に無効化し、強制終了はありませんでした。対話実行結果はローカル記録 `Verification/dotnet-desktop-field-run-20261023-1432/results/interactive-adapter-result.json` です。

데스크톱의 MySQL8.4.13은 새 작업 전용 데이터 디렉터리·포트33316을 사용했습니다. 별도의 빈 일반 비교용 API와 테스트 API를 준비하고 새 계정2개만 사용했으며, 실제 사용자 DB·설정·개인 키는 복사하지 않았습니다. DB 대조는 이 새 저장소의30테이블과 새 캐릭터 소유 행 범위입니다. 기존 실제 DB 전체 행을 다시 대조했다는 뜻은 아닙니다. 서비스 준비·정리 기록10개가 모두 exit0이고 새 API·MySQL은 정상 종료한 뒤 데이터 디렉터리를 보존했습니다. 워커·정리 결과 (로컬 검증 자료: `Verification/dotnet-desktop-field-run-20261023-1432/results/field-worker.json`).

デスクトップの MySQL8.4.13 は新しいジョブ専用データディレクトリー・ポート 33316 を使用しました。別の空の通常比較用 API とテスト API を用意し、新アカウント 2 個だけを使用し、実ユーザー DB・設定・秘密鍵はコピーしませんでした。DB 照合は、この新しい保存先の 30 テーブルと新キャラクターの所有行の範囲です。既存実 DB の全行を再照合した意味ではありません。サービス準備・整理記録 10 件はすべて exit0 で、新 API・MySQL を正常終了後、データディレクトリーを保存しました。worker・整理結果はローカル記録 `Verification/dotnet-desktop-field-run-20261023-1432/results/field-worker.json` です。

이번 두 창은 기존 검증된 Windows Player 사본을 실행한 결과입니다. Unity **6000.3.26f1** 에디터·모듈 설치나 새 Unity 빌드를 수행한 결과로 해석하지 않습니다. 게임 클라이언트와 서버를 서로 다른 물리 PC에서 연결하는 검사는 포함하지 않습니다.

今回の 2 ウィンドウは、既存の検証済み Windows Player コピーを実行した結果です。Unity **6000.3.26f1** Editor・モジュールのインストールや新しい Unity ビルドの結果とは解釈しません。ゲームクライアントとサーバーを別々の物理 PC で接続する検査は含みません。

중앙 실행기의 연결·잠금·검증 사본·종료 코드·결과 회수 절차를 따르고 원본 `.meta`와 `Client/Unity`·`Shared`·`Server`·`Tools` 상대 구조를 보존합니다. 기존 개인 캐시 링크·운영 API·MySQL 데이터를 그대로 원격 실행에 연결하지 않습니다. 다음 실행도 작업별 새 입력과 격리된 개발용 저장소·계정·포트의 시작·상태·정상 종료 구성을 사용합니다. 위 데스크톱 결과는 중앙 실행기로 별도 수집한 원본 보고서에 근거합니다.

中央実行器の接続・ロック・検証コピー・終了コード・結果回収手順に従い、原本 `.meta` と `Client/Unity`・`Shared`・`Server`・`Tools` の相対構造を保ちます。既存の個人キャッシュリンク・本番 API・MySQL データをそのままリモート実行へ接続しません。次回もジョブ別の新入力と隔離された開発用保存先・アカウント・ポートの起動・状態確認・正常終了構成を使います。上記デスクトップ結果は、中央実行器で別に回収した原本レポートに基づきます。

## 검증 도구

## 検証ツール

- `Tools/Test-SharedMigration.ps1`: 실제 Unity 컴파일, 기존 지형 판정과 공용 코드 비교, 실제 맵 추출.
- `Tools/Test-ArchitectureRules.ps1`: 기존320개 검사와 원본 검사 파일 불변, 새 소스 전체 경로 guard.
- `Server/CharacterStore/TestSkillGrowthStore.py`: 저장·성장·중복 방지 API 검사. 새 테스트 행의 fixture 사용을 명시합니다.
- `Tools/Test-ArchitectureJourney.py`: 이전한 클라이언트와 기존 Unity 권위의 실제 두 창 스킬·저장 회귀.
- `Tools/Test-DotNetFieldJourney.py`: 새 .NET 첫 필드와 Unity 두 창의 별도 제한 범위 검증. 데스크톱 어댑터는 원래 게임 검증 조건을 유지하고 실행 환경·격리된 저장소 경로만 연결합니다. 이번 결과는103/103입니다.

- `Tools/Test-SharedMigration.ps1`：実 Unity コンパイル、既存地形判定と共通コードの比較、実マップ抽出。
- `Tools/Test-ArchitectureRules.ps1`：既存 320 件の検査と原本検査ファイルの不変、新ソース全体のパス guard。
- `Server/CharacterStore/TestSkillGrowthStore.py`：保存・成長・重複防止 API 検査。新規テスト行の fixture 使用を明示します。
- `Tools/Test-ArchitectureJourney.py`：移行後クライアントと既存 Unity 権威サーバーによる、実 2 ウィンドウのスキル・保存回帰。
- `Tools/Test-DotNetFieldJourney.py`：新 .NET の最初のフィールドと Unity 2 ウィンドウの別個の限定範囲検証。デスクトップアダプターは元のゲーム検証条件を維持し、実行環境・隔離保存先パスだけを接続します。今回の結果は 103/103 です。

과거 `Verification` 보고서와 당시 고정한 테스트 파일은 덮어쓰지 않습니다. 기존 Unity 회귀와 새 .NET 필드 검증은 범위·카운트·실제 종료 결과를 각각 기록합니다.

過去の `Verification` レポートと当時固定したテストファイルは上書きしません。既存 Unity 回帰と新 .NET フィールド検証は、範囲・件数・実際の終了結果をそれぞれ記録します。
