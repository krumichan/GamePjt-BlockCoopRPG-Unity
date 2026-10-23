# Linux 서버 배포 구성

# Linux サーバーのデプロイ構成

> 공개 문서 범위 · 2026-10-23: 이번 게시 대상은 프로젝트 문서입니다. 아래 명령과 소스 경로는 개발 작업본 기준이며, 문서 게시가 제품 소스·실행 파일의 배포 또는 새 실행 검증을 뜻하지 않습니다. `Verification/`, `Saved/`, `.codex-workspace/`는 공개 저장소에 포함하지 않는 로컬 기록입니다. 과거 작업·승인·검증 상태는 당시 범위로 읽습니다.

> 公開文書の範囲 · 2026-10-23：今回の掲載対象はプロジェクト文書です。以下のコマンドとソースパスは開発作業コピーを基準とし、文書の掲載は製品ソース・実行ファイルの配布や新しい実行検証を意味しません。`Verification/`、`Saved/`、`.codex-workspace/` は公開リポジトリに含めないローカル記録です。過去の作業・承認・検証状態は当時の範囲として読みます。

현재 결과물은 **로컬에서 검증한 배포 템플릿과 릴리스 생성 도구**입니다. 실제 외부 서버·도메인·DNS·SSH·방화벽·운영 systemd 서비스는 변경하지 않았습니다. 로컬 Caddy 검사는 임시 컨테이너와 테스트 CA를 사용하며, 외부 인증서 발급이나 운영 배포를 의미하지 않습니다.

現在の成果物は、**ローカルで検証したデプロイテンプレートとリリース生成ツール**です。実際の外部サーバー・ドメイン・DNS・SSH・ファイアウォール・本番 systemd サービスは変更していません。ローカル Caddy 検査は一時コンテナーとテスト CA を使い、外部証明書発行や本番デプロイを意味しません。

## 구성

## 構成

- Linux x86_64/glibc 호스트에서 MySQL 8.4는 Docker로 실행하며 `127.0.0.1:3306`만 공개합니다. 데이터는 **외부 named volume**에 유지합니다.
- CharacterStore는 포함된 self-contained .NET 10 Linux 실행 파일로, `mosslight` 사용자와 `127.0.0.1:17871`에서 실행합니다.
- Unity Dedicated Server는 `mosslight` 사용자로 실행하며 게임 TCP `7777`에 TLS와 계정 인증을 요구합니다.
- 호스트 Caddy는 HTTPS `443`에서 **POST `/auth/register`, `/auth/login`, `/auth/me`, `/auth/logout`, `/auth/link`**만 CharacterStore로 전달합니다. `/auth/resolve`, 모든 기존 캐릭터/보상 API, health, shutdown과 나머지 경로는 외부에서 404입니다.
- 처음에는 같은 공개 호스트명 하나를 사용하세요. 예: 로그인 API `https://play.example.com`, 게임 `play.example.com:7777`, 게임 인증서 이름 `play.example.com`. 예시 도메인은 운영자가 실제 도메인을 정한 뒤 바꿉니다.

- Linux x86_64/glibc ホスト上で MySQL 8.4 を Docker で実行し、`127.0.0.1:3306` だけに公開します。データは**外部 named volume** に維持します。
- CharacterStore は同梱の self-contained .NET 10 Linux 実行ファイルで、`mosslight` ユーザー・`127.0.0.1:17871` で実行します。
- Unity Dedicated Server は `mosslight` ユーザーで実行し、ゲーム TCP `7777` に TLS とアカウント認証を要求します。
- ホストの Caddy は HTTPS `443` で **POST `/auth/register`、`/auth/login`、`/auth/me`、`/auth/logout`、`/auth/link`** だけを CharacterStore へ転送します。`/auth/resolve`、既存の全キャラクター/報酬 API、health、shutdown とその他のパスは外部から 404 になります。
- 最初は同じ公開ホスト名を 1 個使います。例：ログイン API `https://play.example.com`、ゲーム `play.example.com:7777`、ゲーム証明書名 `play.example.com`。例示ドメインは運用者が実ドメインを決めてから置き換えます。

`Caddyfile`은 request body를 16KB로 제한하고, 전달된 `X-Mosslight-Key`를 제거하며 응답에 `no-store`를 적용합니다. 내부 service key는 클라이언트나 Caddy 설정에 넣지 않습니다. Caddy의 관리 API는 loopback `127.0.0.1:2019`에만 있습니다.

`Caddyfile` は request body を 16KB に制限し、転送された `X-Mosslight-Key` を除去し、応答に `no-store` を適用します。内部 service key はクライアントや Caddy 設定に入れません。Caddy 管理 API は loopback `127.0.0.1:2019` だけに置きます。

## Windows에서 릴리스 만들기

## Windows でリリースを作る

Unity Linux Dedicated Server 모듈과 성공한 `Verification/linux-server-build-result.json`이 먼저 필요합니다. 이 단계는 빌드를 자동으로 시작하지 않습니다.

Unity Linux Dedicated Server モジュールと成功済みの `Verification/linux-server-build-result.json` が先に必要です。この段階でビルドを自動開始しません。

```powershell
powershell -NoProfile -File Tools/Package-ServerRelease.ps1
```

도구는 `Builds/LinuxServer`의 허용된 게임 산출물, 별도로 publish한 `linux-x64` self-contained CharacterStore, 이 폴더의 운영 템플릿, SHA256 manifest만 새 `Releases/server-linux-x64-*` 폴더와 tar.gz로 만듭니다. 기존 출력 경로는 덮어쓰지 않습니다. 게임/API 실행 파일은 x86_64 ELF인지 검사합니다. `.local`, `LocalState`, SQL 백업, 실제 config, 인증서/개인 키, 토큰, Unity/애플리케이션 소스, 디버그 심볼은 포함하지 않습니다. `test_deployment.py`도 릴리스에서 제외합니다.

ツールは `Builds/LinuxServer` 内の許可済みゲーム成果物、別途 publish した `linux-x64` self-contained CharacterStore、このフォルダーの運用テンプレート、SHA256 manifest だけから、新しい `Releases/server-linux-x64-*` フォルダーと tar.gz を作ります。既存出力先は上書きしません。ゲーム/API 実行ファイルが x86_64 ELF かを検査します。`.local`、`LocalState`、SQL バックアップ、実 config、証明書/秘密鍵、トークン、Unity/アプリケーションソース、デバッグシンボルは含みません。`test_deployment.py` もリリースから除外します。

배포 스크립트와 checksum 검증용 Python 파일은 운영 템플릿으로 포함됩니다. 서비스는 외부 `/opt/mosslight/private/config.json`을 명시적으로 읽습니다. 일반 서버 릴리스에 기존 DB/계정 비밀이 따라 들어가지 않습니다.

デプロイスクリプトと checksum 検証用 Python ファイルは運用テンプレートとして含めます。サービスは外部の `/opt/mosslight/private/config.json` を明示的に読みます。通常のサーバーリリースに既存 DB/アカウントの秘密情報は入りません。

## 새 Linux 호스트의 사전 준비

## 新しい Linux ホストの事前準備

x86_64 Linux, Docker Engine/Compose, Python 3, OpenSSL, systemd, Caddy 2가 필요합니다. Ubuntu 24.04 같은 glibc 배포판에서 호스트의 네이티브 라이브러리 호환성을 확인하세요. Self-contained API에도 해당 호스트의 기본 네이티브 라이브러리는 필요합니다. 배포 전에 `ldd game/MosslightServer.x86_64`, `ldd game/UnityPlayer.so`, `ldd character-store/CharacterStore`에 누락된 라이브러리가 없는지 확인합니다. 이 검사는 실제 대상 Linux 호스트에서 아직 수행되지 않았습니다.

x86_64 Linux、Docker Engine/Compose、Python 3、OpenSSL、systemd、Caddy 2 が必要です。Ubuntu 24.04 など glibc ディストリビューション上で、ホストのネイティブライブラリー互換性を確認してください。Self-contained API にもホストの基本ネイティブライブラリーは必要です。デプロイ前に `ldd game/MosslightServer.x86_64`、`ldd game/UnityPlayer.so`、`ldd character-store/CharacterStore` でライブラリー不足がないことを確認します。この検査は実際の対象 Linux ホストではまだ実行していません。

운영자가 `mosslight` 시스템 사용자/그룹과 `/opt/mosslight/releases`를 준비한 뒤 새 릴리스를 고유 디렉터리에 풉니다. root가 쓰고 서비스 사용자는 읽기/실행만 가능하도록 두세요.

運用者が `mosslight` システムユーザー/グループと `/opt/mosslight/releases` を準備してから、新しいリリースを固有ディレクトリーへ展開します。root に書き込み権限を持たせ、サービスユーザーは読み取り/実行だけができるようにします。

```sh
# 대상 호스트에서 운영자가 수행할 예시입니다. 현재 PC에서는 실행하지 않았습니다.
sudo groupadd --system mosslight
sudo useradd --system --gid mosslight --home-dir /var/lib/mosslight --shell /usr/sbin/nologin mosslight
sudo install -d -o root -g root -m 0755 /opt/mosslight/releases
# RELEASE에는 실제로 푼 고유 릴리스 디렉터리를 넣습니다.
cd "$RELEASE"
sha256sum -c manifest.sha256
python3 deployment/verify-release.py
sudo chmod 0755 game/MosslightServer.x86_64 character-store/CharacterStore
# UnityCrashHandler64가 있다면 이 파일에도 실행 권한을 줍니다.
# 검증한 릴리스로 /opt/mosslight/current 심볼릭 링크를 연결합니다.
```

コード枠のコメントは、対象 host で運用者が行う例であり現在 PC では未実行であること、`RELEASE` に実際に展開した固有 directory を入れること、`UnityCrashHandler64` があれば実行権限を付けること、検証済み release へ `/opt/mosslight/current` の symbolic link を接続することを説明しています。

압축 파일의 `.sha256`과 폴더 안 manifest는 전송 무결성을 확인합니다. 서비스가 사용하는 release와 `current`의 경로는 systemd 템플릿의 `/opt/mosslight/current`와 일치해야 합니다. 이전 릴리스는 남겨 두어 되돌릴 수 있게 합니다.

アーカイブの `.sha256` とフォルダー内 manifest は転送の整合性を検証します。サービスが使う release と `current` のパスは、systemd テンプレートの `/opt/mosslight/current` と一致させる必要があります。以前のリリースは戻せるよう残します。

## 최초 비밀 생성과 기존 데이터 이동

## 初回の秘密情報生成と既存データ移行

**새 DB를 처음 만드는 경우에만** 다음을 실행합니다.

**新しい DB を初めて作る場合だけ**、次を実行します。

```sh
sudo sh "$RELEASE/deployment/prepare-private.sh" /opt/mosslight
```

이 스크립트는 32-byte 난수로 서로 다른 API/account master key, DB 암호, PFX 암호를 한 번 생성합니다. 기존 `private` 디렉터리나 파일이 있으면 변경 없이 중단합니다. 모든 파일은 0600입니다. `private` 디렉터리는 root 소유 0710, `mosslight` 그룹에는 경로 통과만 허용합니다. API config/PFX env는 서비스 사용자 소유이고 Docker용 DB 암호 파일은 root만 읽습니다. 비밀 값은 stdout에 출력하지 않습니다.

このスクリプトは 32-byte の乱数で、互いに異なる API/account master key、DB パスワード、PFX パスワードを 1 回生成します。既存の `private` ディレクトリーやファイルがあれば、変更せず停止します。全ファイルは 0600 です。`private` ディレクトリーは root 所有の 0710 で、`mosslight` グループにはパスの通過だけを許可します。API config/PFX env はサービスユーザー所有、Docker 用 DB パスワードファイルは root だけが読みます。秘密値は stdout に出力しません。

기존 SQL 백업이나 이미 계정이 있는 DB를 옮길 때는 새 키를 생성하지 않습니다. **원래 `.local/config.json`의 `apiKey`와 `accountKey` 상태까지 그대로 보존**하고 DB 접속 정보만 새 호스트에 맞추세요. 기존 `accountKey`가 없었다면 원래 `apiKey`의 파생 키를 계속 사용하므로 임의로 `accountKey`를 추가하면 안 됩니다. 키 교체는 검증된 decrypt/re-encrypt 마이그레이션이 필요합니다. 원래 키와 SQL 백업을 함께 보관해야 암호화된 캐릭터 자격 증명을 새 PC에서도 복원할 수 있습니다.

既存 SQL バックアップやアカウントのある DB を移す場合は、新しい鍵を生成しません。**元の `.local/config.json` の `apiKey` と `accountKey` の状態までそのまま保存**し、DB 接続情報だけを新ホストに合わせます。既存の `accountKey` がなければ元の `apiKey` からの派生鍵を使い続けるため、勝手に `accountKey` を追加してはいけません。鍵交換には検証済み decrypt/re-encrypt migration が必要です。元の鍵と SQL バックアップを一緒に保存して初めて、暗号化済みキャラクター認証情報を新 PC でも復元できます。

새 MySQL 인스턴스를 쓸 때는 DB 암호 파일과 config의 접속 암호가 일치해야 합니다. MySQL 사용자와 DB를 만든 뒤 SQL 백업을 복원하고 **그 다음** CharacterStore를 시작하세요. 기존 named volume에 새 최초 설치 암호를 적용하여 초기화하려고 하지 마세요.

新しい MySQL インスタンスでは DB パスワードファイルと config の接続パスワードを一致させます。MySQL ユーザーと DB を作成後、SQL バックアップを復元し、**その後で** CharacterStore を起動します。既存 named volume に新規初回インストールのパスワードを適用して初期化しようとしないでください。

## MySQL 데이터 유지

## MySQL データの維持

`compose.mysql.yml`은 기존 named volume을 자동으로 만들거나 지우지 않습니다. 운영자가 이름과 소유권을 확인한 **새 볼륨**을 명시적으로 준비해야 합니다. 같은 이름의 기존 볼륨이 있다면 데이터를 보존하고 사용 중인 DB/암호를 먼저 확인합니다.

`compose.mysql.yml` は既存 named volume を自動作成・削除しません。運用者が名前と所有関係を確認した**新しい volume** を明示的に準備する必要があります。同名の既存 volume があればデータを保存し、使用中の DB/パスワードを先に確認します。

```sh
# 새 설치의 새 볼륨에만 사용합니다. 기존 볼륨은 삭제하지 않습니다.
sudo docker volume create --label com.mosslight.managed=mosslight.production.mysql.v1 mosslight-prod-mysql-data
sudo docker compose --env-file /opt/mosslight/private/compose.env -f "$RELEASE/deployment/compose.mysql.yml" up -d
```

コード枠の注意は、新規インストール用の新 volume にだけ使用し、既存 volume は削除しないという意味です。

기존 데이터에는 `docker compose down -v`, `docker volume rm`, 데이터 디렉터리 초기화 명령을 사용하지 않습니다. 실제 운영에서는 SQL 논리 백업과 private config를 별도로 유지하고, 원본을 덮어쓰지 않는 새 DB로 복원을 검증하세요.

既存データに `docker compose down -v`、`docker volume rm`、データディレクトリー初期化コマンドは使いません。本番運用では SQL 論理バックアップと private config を別途維持し、原本を上書きしない新 DB への復元を検証してください。

## HTTPS와 게임 PFX 인증서

## HTTPS とゲーム用 PFX 証明書

운영자가 도메인을 정한 뒤 DNS를 대상 호스트에 연결하고 Caddy를 공식 방법으로 설치합니다. Caddy 서비스의 환경에 `MOSSLIGHT_AUTH_DOMAIN=play.example.com`을 설정하고 이 `Caddyfile`을 설치합니다. DNS/ACME에 맞춰 HTTP `80`과 HTTPS `443` 접근을 준비하세요. 게임은 별도로 TLS TCP `7777`을 사용합니다. DB `3306`, CharacterStore `17871`, Caddy 관리 `2019`는 외부로 공개하지 않습니다.

運用者がドメインを決めたら DNS を対象ホストへ接続し、Caddy を公式手順でインストールします。Caddy サービスの環境に `MOSSLIGHT_AUTH_DOMAIN=play.example.com` を設定し、この `Caddyfile` を設置します。DNS/ACME に合わせて HTTP `80` と HTTPS `443` へのアクセスを準備します。ゲームは別途 TLS TCP `7777` を使います。DB `3306`、CharacterStore `17871`、Caddy 管理 `2019` は外部公開しません。

운영 Caddy는 `key_type rsa2048`을 명시해 검증한 Unity Mono/RSA 조합과 맞춥니다. Caddy가 발급한 해당 호스트의 PEM 인증서 체인과 개인 키 경로를 확인한 뒤 다음 도구로 **같은 호스트 이름의** 게임용 PFX를 준비합니다. Caddy 저장소 경로는 설치 방식에 따라 다르므로 경로를 고정해서 가정하지 않습니다.

本番 Caddy は `key_type rsa2048` を明示し、検証済み Unity Mono/RSA の組み合わせに合わせます。Caddy が発行した当該ホストの PEM 証明書チェーンと秘密鍵パスを確認後、次のツールで**同じホスト名の**ゲーム用 PFX を準備します。Caddy 保存先はインストール方法で異なるため、固定パスとは仮定しません。

```sh
sudo sh "$RELEASE/deployment/export-game-pfx.sh" /actual/caddy/fullchain.pem /actual/caddy/privkey.pem /opt/mosslight
```

PFX 암호는 파일에서만 읽으며 CLI 값으로 전달하지 않습니다. Unity Mono의 PKCS#12 읽기 호환을 위해 PFX 자체는 PBE-SHA1-3DES/MAC-SHA1 형식으로 내보냅니다. 이는 비밀번호로 보호한 인증서 파일의 저장 형식이며, 게임 통신의 TLS 인증서 검증을 생략하는 설정이 아닙니다. 새 PFX를 검증한 뒤 0600·서비스 사용자 소유로 원자 교체합니다. 그 다음 두 systemd unit을 `/etc/systemd/system/`에 설치하고 `systemctl daemon-reload`, CharacterStore, 게임 순서로 시작합니다. `game.env`를 공개 디렉터리나 release에 복사하지 않습니다.

PFX パスワードはファイルだけから読み、CLI 値で渡しません。Unity Mono の PKCS#12 読み取り互換性のため、PFX 自体は PBE-SHA1-3DES/MAC-SHA1 形式で出力します。これはパスワード保護した証明書ファイルの保存形式であり、ゲーム通信の TLS 証明書検証を省略する設定ではありません。新 PFX を検証してから 0600・サービスユーザー所有で原子的に置き換えます。その後、systemd unit 2 個を `/etc/systemd/system/` へインストールし、`systemctl daemon-reload`、CharacterStore、ゲームの順に起動します。`game.env` を公開ディレクトリーや release へコピーしません。

Caddy의 HTTPS 인증서는 자동 갱신되지만, 이미 내보낸 게임 PFX는 자동으로 바뀌지 않습니다. **인증서 갱신 성공 후 위 export 도구를 다시 실행하고 `sudo systemctl restart mosslight-game`을 수행**해야 새 인증서가 적용됩니다. 운영 전에 해당 호스트의 갱신 hook/timer에 이 두 단계를 연결하고, 실패하면 기존 PFX를 유지하는지 확인하세요. 현재 템플릿은 호스트별 ACME 저장소 경로와 자동 갱신 hook을 임의로 설치하지 않습니다.

Caddy の HTTPS 証明書は自動更新されますが、出力済みのゲーム PFX は自動で変わりません。新しい証明書を適用するには、**証明書更新成功後に上記 export ツールを再実行し、`sudo systemctl restart mosslight-game` を行う**必要があります。本番運用前に対象ホストの更新 hook/timer へこの 2 段階を接続し、失敗時に既存 PFX が維持されることを確認してください。現在のテンプレートはホスト別 ACME 保存パスや自動更新 hook を勝手にインストールしません。

## 로컬 검증

## ローカル検証

```powershell
powershell -NoProfile -File Tools/Test-Deployment.ps1
```

정적 보안 불변식과 실제 Docker Compose 파싱을 확인합니다. 임시 Linux 컨테이너에서는 shell 구문, 최초 비밀 생성/0600 권한/기존 파일 보존과 PEM→PFX 왕복을 검사합니다. 임시 Caddy는 `127.0.0.1:19443`에만 바인딩하고, 테스트 클라이언트가 그 컨테이너의 공개 CA 인증서만 명시적으로 신뢰합니다. 실제 HTTPS로 5개 public POST 경로 전달, internal 경로/잘못된 method/인코딩 변형 경로 404와 service-key header 제거를 검사합니다. 운영 DB/볼륨, 호스트 신뢰 저장소, 방화벽은 변경하지 않습니다. Caddy upstream은 응답 fixture이며 실제 계정 API 검사는 별도 `Test-AccountStore.ps1`이 담당합니다.

静的セキュリティ不変条件と実 Docker Compose の解析を確認します。一時 Linux コンテナーでは shell 構文、初回秘密情報生成/0600 権限/既存ファイル保存、PEM→PFX 往復を検査します。一時 Caddy は `127.0.0.1:19443` だけに bind し、テストクライアントはそのコンテナーの公開 CA 証明書だけを明示的に信頼します。実 HTTPS で 5 個の public POST パス転送、internal パス/誤 method/符号化変種パスの 404、service-key header 除去を検査します。本番 DB/volume、ホスト信頼ストア、ファイアウォールは変更しません。Caddy upstream は応答 fixture であり、実アカウント API 検査は別の `Test-AccountStore.ps1` が担当します。

`-SkipDocker`는 정적 검사만 실행하는 선택지입니다. Docker smoke를 실행한 결과와 같은 의미로 취급하지 마세요.

`-SkipDocker` は静的検査だけを実行する選択肢です。Docker smoke を実行した結果と同じ意味で扱わないでください。

`Tools/Test-LinuxServerRuntime.ps1`은 실제 Linux Unity 실행 파일을 Ubuntu 24.04 Docker 이미지에서 UID 1654로 실행합니다. Plain guest 프로토콜과 인증서를 검증하는 TLS 모드 각각에서 protocol 16 greeting/snapshot, 실제 이동 입력, presentation 객체 0개와 정상 종료를 검사합니다. `--no-persistence`를 쓰는 플랫폼/전송 smoke이므로 **Linux에서 실제 계정/MySQL 파티 전체를 검증했다는 의미는 아닙니다.** 계정 API/두 창 로그인 검증은 별도 로컬 테스트가 담당합니다.

`Tools/Test-LinuxServerRuntime.ps1` は実 Linux Unity 実行ファイルを Ubuntu 24.04 Docker イメージで UID 1654 として実行します。Plain guest プロトコルと証明書を検証する TLS モードのそれぞれで、protocol 16 greeting/snapshot、実移動入力、presentation オブジェクト 0 件と正常終了を検査します。`--no-persistence` を使うプラットフォーム/転送 smoke のため、**Linux で実アカウント/MySQL パーティー全体を検証した意味ではありません。** アカウント API/2 ウィンドウログイン検証は別のローカルテストが担当します。

비root Mono의 file-copy 권한 보존은 Windows 외장드라이브 bind mount와 맞지 않으므로 이 Linux 하네스의 보고서는 native tmpfs에 생성하고 종료 후 외장드라이브로 stream 전송합니다. 실제 Linux 호스트에서는 systemd의 native StateDirectory/LogsDirectory를 사용합니다. Windows에서 OpenSSL 테스트 PFX를 만들 때 암호 파일은 개행 없는 ASCII/UTF-8 bytes를 사용하세요. CRLF 암호 파일과 Docker env-file의 CR 처리 차이는 서로 다른 PFX 암호를 만들 수 있습니다. 이는 개발 fixture의 문자열 불일치이며 .NET/Unity의 PFX 알고리즘 미지원으로 처리하지 않습니다.

非 root Mono の file-copy 権限保持は Windows 外付けドライブ bind mount と合わないため、この Linux ハーネスのレポートは native tmpfs に生成し、終了後に外付けドライブへ stream 転送します。実 Linux ホストでは systemd の native StateDirectory/LogsDirectory を使います。Windows で OpenSSL テスト PFX を作るとき、パスワードファイルは改行なしの ASCII/UTF-8 bytes を使ってください。CRLF パスワードファイルと Docker env-file の CR 処理差は、PFX パスワードの相違を作り得ます。これは開発 fixture の文字列不一致であり、.NET/Unity が PFX アルゴリズムに未対応という扱いにはしません。

## 공식 근거

## 公式根拠

- [Unity Dedicated Server Linux build](https://docs.unity3d.com/6000.3/Documentation/Manual/dedicated-server-build.html)
- [.NET self-contained publishing](https://learn.microsoft.com/en-us/dotnet/core/deploying/)
- [ASP.NET Core Linux/systemd hosting](https://learn.microsoft.com/en-us/aspnet/core/host-and-deploy/linux-nginx?view=aspnetcore-10.0)
- [Docker loopback port publishing](https://docs.docker.com/engine/network/port-publishing/), [external volumes](https://docs.docker.com/reference/compose-file/volumes/)
- [Caddy exact path/method matchers](https://caddyserver.com/docs/caddyfile/matchers), [mutually exclusive handle blocks](https://caddyserver.com/docs/caddyfile/directives/handle), [automatic HTTPS](https://caddyserver.com/docs/automatic-https)

- [Unity Dedicated Server の Linux ビルド](https://docs.unity3d.com/6000.3/Documentation/Manual/dedicated-server-build.html)
- [.NET self-contained publishing](https://learn.microsoft.com/en-us/dotnet/core/deploying/)
- [ASP.NET Core の Linux/systemd hosting](https://learn.microsoft.com/en-us/aspnet/core/host-and-deploy/linux-nginx?view=aspnetcore-10.0)
- [Docker loopback port publishing](https://docs.docker.com/engine/network/port-publishing/)、[external volumes](https://docs.docker.com/reference/compose-file/volumes/)
- [Caddy の正確な path/method matchers](https://caddyserver.com/docs/caddyfile/matchers)、[相互排他 handle blocks](https://caddyserver.com/docs/caddyfile/directives/handle)、[automatic HTTPS](https://caddyserver.com/docs/automatic-https)
