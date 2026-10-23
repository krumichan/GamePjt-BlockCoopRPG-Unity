# 독립 .NET Linux 릴리스 — 아직 기본 경로가 아님

# 独立 .NET Linux リリース — まだ既定経路ではない

> 공개 문서 범위 · 2026-10-23: 이번 게시 대상은 프로젝트 문서입니다. 아래 명령과 소스 경로는 개발 작업본 기준이며, 문서 게시가 제품 소스·실행 파일의 배포 또는 새 실행 검증을 뜻하지 않습니다. `Verification/`, `Saved/`, `.codex-workspace/`는 공개 저장소에 포함하지 않는 로컬 기록입니다. 과거 작업·승인·검증 상태는 당시 범위로 읽습니다.

> 公開文書の範囲 · 2026-10-23：今回の掲載対象はプロジェクト文書です。以下のコマンドとソースパスは開発作業コピーを基準とし、文書の掲載は製品ソース・実行ファイルの配布や新しい実行検証を意味しません。`Verification/`、`Saved/`、`.codex-workspace/` は公開リポジトリに含めないローカル記録です。過去の作業・承認・検証状態は当時の範囲として読みます。

이 템플릿은 현재 Unity 배포와 별개다. 파일을 만든 것만으로 실행기를 전환하거나 릴리스를 게시하거나 서비스를 배포하거나 포트를 열거나 DB를 migration하지 않는다. 전체 .NET 게임 진행 검증과 일반 프로필 승격이 통과한 뒤에만 적용한다.

これらのテンプレートは現在の Unity デプロイとは別です。作成しただけで launcher の切り替え、release 公開、service のデプロイ、port 開放、database migration は行いません。.NET ゲーム全体の進行検証と通常 profile への昇格が通過してから適用します。

## 패키지

## パッケージ

`Tools/Package-DotNetServerRelease.ps1`는 기존 실행 경로에 연결되지 않은 데스크톱 작업 전용 빌드 도구다. root가 소유하는 데스크톱 어댑터가 검증된 입력, SDK 10, Python과 결과 경로를 제공한다. GameServer와 CharacterStore를 각각 `linux-x64 --self-contained true`로 publish하고, 두 x64 ELF 실행 파일과 엔진 독립성을 검사한 뒤 새 릴리스 디렉터리와 SHA256 manifest를 만든다. Python 검증은 정확한 파일과 선택적으로 tar.gz 바이트를 독립 대조한다. 기존 릴리스는 덮어쓰지 않는다. 빌드 로그는 작업의 비공개 디렉터리에 남긴다.

`Tools/Package-DotNetServerRelease.ps1` は既存経路に未接続の、デスクトップ job 専用ビルドツールです。root 所有のデスクトップアダプターが検証済み入力、SDK 10、Python、結果パスを渡します。GameServer と CharacterStore をそれぞれ `linux-x64 --self-contained true` で publish し、両 x64 ELF 実行ファイルとエンジン非依存を確認して、新しい release directory と SHA256 manifest を作ります。Python 検証は正確なファイルと、任意で tar.gz のバイトを独立照合します。既存 release は上書きしません。build log は job の非公開 directory に保持します。

패키지에는 `game/`, `character-store/`와 `deployment/` 아래의 명시적인 운영자 템플릿이 들어간다. 애플리케이션 소스, Unity 라이브러리, `.local`, config, 인증/프로필 파일, SQL, DB, PFX/개인키, 로그와 Git/캐시 디렉터리는 제외한다. 배포용 Python/shell/service 파일은 애플리케이션 소스가 아닌 운영자 템플릿이다. 실제 비공개 설정과 MySQL 데이터를 릴리스에 복사하지 않는다.

パッケージには `game/`、`character-store/` と `deployment/` 内の明示的な運用者テンプレートを含めます。アプリケーションソース、Unity ライブラリー、`.local`、config、認証/profile ファイル、SQL、database、PFX/秘密鍵、log、Git/cache directory は除外します。デプロイ用 Python/shell/service ファイルは運用者テンプレートであり、アプリケーションソースではありません。実際の非公開設定と MySQL データは release へコピーしません。

## 기존 저장소와 공개 로그인

## 既存ストレージと公開ログイン

`/opt/mosslight/private/config.json`, 그 service/account 키, 기존 MySQL volume과 CharacterStore 서비스를 유지한다. 일반 GameServer는 `testDatabase=false`인 인증된 health, TLS와 계정 로그인을 요구한다. 이 전환에는 새 schema, 계정 가져오기, 데이터 초기화, SQL 복원, 키 교체가 포함되지 않는다. 새 호스트에서만 기존 `prepare-private.sh`를 검토한다. 이 도구는 기존 private 디렉터리가 있으면 거부하며 운영 비밀을 교체해서는 안 된다. 기존 `compose.mysql.yml`과 외부 named volume 계약은 그대로다.

`/opt/mosslight/private/config.json` とその service/account 鍵、既存 MySQL volume、既存 CharacterStore service を維持します。通常 GameServer は `testDatabase=false` の認証済み health、TLS、アカウントログインを要求します。この切り替えに新 schema、アカウント import、data reset、SQL restore、鍵 rotation は含めません。新 host の場合だけ既存 `prepare-private.sh` をレビューします。このツールは既存 private directory を拒否し、本番秘密情報を決して置き換えてはいけません。既存 `compose.mysql.yml` と外部 named volume 契約は不変です。

기존 Caddy HTTPS 공개 인증 허용 목록을 재사용한다. POST `/auth/register`, `/auth/login`, `/auth/me`, `/auth/logout`, `/auth/link`만 허용한다. `/auth/resolve`, game-control, 캐릭터/보상 경로와 service key는 비공개다. CharacterStore와 game-control은 IPv4 loopback에 유지하고 의도한 클라이언트에 게임 TLS TCP 포트만 연다. LAN 클라이언트에는 접근 가능한 HTTPS 인증 URL과 그에 맞는 게임 TLS host/trust 또는 pin이 필요하다. 클라이언트 자신의 127.0.0.1은 원격 서버가 아니다.

既存 Caddy HTTPS の公開認証 allowlist を再利用し、POST `/auth/register`、`/auth/login`、`/auth/me`、`/auth/logout`、`/auth/link` だけを許可します。`/auth/resolve`、game-control、character/reward endpoint、service key は非公開を維持します。CharacterStore と game-control は IPv4 loopback に置き、意図する client にゲーム TLS TCP port だけを開きます。LAN client には到達可能な HTTPS auth URL と、一致するゲーム TLS host/trust または pin が必要です。client 自身の 127.0.0.1 は remote server ではありません。

## 실제 Linux 검증 후 설치

## 実 Linux 検証後のインストール

압축 해제한 릴리스에서 `python3 deployment/verify-release.py`를 실행한다. 필요하면 `--archive /path/to/release.tar.gz`를 추가한다. `game/Mosslight.GameServer`, `character-store/CharacterStore`, `deployment/run-game.sh`에 실행 권한을 설정한다. 실제 Linux x64/glibc 호스트에서 `ldd`로 native 의존성을 확인한다. Self-contained .NET은 별도로 설치하는 .NET runtime을 불필요하게 할 뿐 native 라이브러리 호환성을 증명하지 않는다.

展開した release で `python3 deployment/verify-release.py` を実行し、必要なら `--archive /path/to/release.tar.gz` を追加します。`game/Mosslight.GameServer`、`character-store/CharacterStore`、`deployment/run-game.sh` に実行権限を付けます。実際の Linux x64/glibc host で `ldd` により native 依存を確認します。Self-contained .NET は別途 .NET runtime をインストールする必要をなくすだけで、native-library 互換性を証明しません。

새 `mosslight-game.service` 설치는 명시적인 운영자 변경으로만 수행한다. `/opt/mosslight/current` 릴리스 소유권은 root가 통제하고 비공개 config는 밖에 둔다. `/opt/mosslight/private/game.env`에는 `MOSSLIGHT_TLS_PFX_PASSWORD`가 들어간다. 비공개 디렉터리 안에서 mode 0600을 유지하고, root 또는 필요한 읽기 권한만 가진 서비스 신원으로 소유한다. 내용을 명령행·패키지·공개 로그에 넣지 않는다. 기존 `export-game-pfx.sh`는 명시적으로 운영할 때 별도 게임 서버 PFX를 준비한다.

新しい `mosslight-game.service` のインストールは明示的な運用者変更としてだけ行います。`/opt/mosslight/current` release の所有権は root 管理とし、private config は外に置きます。`/opt/mosslight/private/game.env` は `MOSSLIGHT_TLS_PFX_PASSWORD` を含みます。非公開 directory 内で mode 0600、root または必要な読み取り権限だけを持つ service identity の所有に保ちます。内容を command line、package、公開 log に入れません。既存 `export-game-pfx.sh` は明示的な操作時に別の game-server PFX を準備します。

서비스는 mode 0700의 `/run/mosslight-game`을 쓰고 시작할 때마다 고유한 `run-*` 하위 디렉터리를 만든다. 제어 token, ready와 최종 보고서 경로는 실행마다 새로 만들며 기존 파일을 재사용하지 않는다. 비공개 파일은 재시작/중지에도 유지하며 재부팅 또는 운영자의 의도적인 정리까지 보존한다. 제어 token을 공개하거나 이 디렉터리를 무조건 archive에 넣지 않는다.

service は mode 0700 の `/run/mosslight-game` を使い、起動ごとに固有の `run-*` child を作ります。control token、ready、final report のパスは毎回新しくし、古いファイルは再利用しません。再起動/停止をまたいで非公開のまま保存し、reboot または運用者の意図した cleanup まで残します。control token を公開したり、無条件でこの directory を archive に入れたりしません。

SIGTERM은 실제 .NET 프로세스에 전달돼야 하며(`run-game.sh`는 exec 사용), 호스트는 동일한 불변 저장 작업이 완료될 때까지 배출한 뒤 종료한다. `Restart=on-failure`와 `SendSIGKILL=no`는 지연 저장 중의 자동 강제 종료를 피한다. 120초 후에도 중지가 대기할 수 있다. CharacterStore를 계속 제공하고 비공개 상태를 확인하며 배출 완료를 기다린다. 실제 Linux 신호/정상 종료 동작과 native 의존성은 root가 소유하는 별도 runtime 검증이 필요하다. Windows/두 창 결과는 이를 증명하지 않는다. 이 폴더 자체로 기본 템플릿이나 서비스는 바뀌지 않는다.

SIGTERM は実際の .NET process に届く必要があり（`run-game.sh` は exec を使用）、host は同じ不変の保存操作を完了まで drain してから終了します。`Restart=on-failure` と `SendSIGKILL=no` により遅延保存中の自動 hard kill を避けます。120 秒後も stop が pending の場合は CharacterStore を利用可能に保ち、private status を調べ、drain 確認を待ちます。実 Linux signal/normal-stop 動作と native 依存は root 所有の別の runtime 検証が必要です。Windows/2 ウィンドウの結果では証明できません。このフォルダーだけで既定テンプレートや service は変わりません。
