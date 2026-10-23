# 독립 .NET 플레이 실행기 — 선택 사용

# 独立 .NET プレイ実行ツール — 任意で使用

> 공개 문서 범위 · 2026-10-23: 이번 게시 대상은 프로젝트 문서입니다. 아래 명령과 소스 경로는 개발 작업본 기준이며, 문서 게시가 제품 소스·실행 파일의 배포 또는 새 실행 검증을 뜻하지 않습니다. `Verification/`, `Saved/`, `.codex-workspace/`는 공개 저장소에 포함하지 않는 로컬 기록입니다. 과거 작업·승인·검증 상태는 당시 범위로 읽습니다.

> 公開文書の範囲 · 2026-10-23：今回の掲載対象はプロジェクト文書です。以下のコマンドとソースパスは開発作業コピーを基準とし、文書の掲載は製品ソース・実行ファイルの配布や新しい実行検証を意味しません。`Verification/`、`Saved/`、`.codex-workspace/` は公開リポジトリに含めないローカル記録です。過去の作業・承認・検証状態は当時の範囲として読みます。

이 새 PowerShell 7 도구는 기존 CMD 파일이나 Unity 서버 실행기에 연결되어 있지 않다. 일반 프로필 호스트 제안용 소스를 준비한 단계이며, 사용 전에 그 제안과 일치하는 Windows publish를 승격·검증해야 한다. 파일이 있다는 이유로 runtime 성공을 주장하지 않는다.

新しい PowerShell 7 ツールは既存 CMD ファイルや Unity-server launcher に接続されていません。通常 profile host の提案向けにソースを準備した段階で、使用前に提案と対応する Windows publish の昇格・検証が必要です。これらのファイルで runtime 成功を主張しません。

저장소 루트에서 호환되는 Unity 클라이언트와 독립 Windows 서버 publish를 준비한 뒤 실행한다. framework-dependent publish라면 해당 프로세스에서 맞는 .NET 10 / ASP.NET Core 10 runtime에도 접근할 수 있어야 한다.

リポジトリ root で対応する Unity client と独立 Windows server publish を準備してから実行します。framework-dependent publish では、その process から対応する .NET 10 / ASP.NET Core 10 runtime も利用可能である必要があります。

```powershell
pwsh -NoProfile -File Tools/Start-DotNetCoop.ps1 -Mode Coop -Encounter meadow -ProfileA default -ProfileB coop-guest
pwsh -NoProfile -File Tools/Start-DotNetCoop.ps1 -Mode Server -Encounter boss
```

`Mode Coop`는 .NET PID/world/protocol/profile의 ready 영수증이 일치한 뒤 실제 Unity 로그인 창 두 개를 연다. 서로 다른 계정과 로컬 프로필을 사용하고 각 창에서 계속 진행한 뒤 접속한다. `Mode Server`는 Unity 창을 열지 않는다. `Encounter meadow|boss`는 파티 크기 2로 전체 권위 서버에 전달한다. 둘 다 기존 일반 캐릭터 API와 MySQL 데이터를 사용한다. `Ensure-CharacterStore.ps1`를 호출하며, `-UseRunningStore`는 그 대신 인증된 일반 health가 일치하는 이미 관리 중인 API를 요구한다. API/DB/config를 복사·초기화·migration하지 않는다.

`Mode Coop` は .NET PID/world/protocol/profile の ready 記録が一致してから、表示される Unity ログインウィンドウを 2 個開きます。異なるアカウントとローカル profile を使い、各ウィンドウで続行して接続します。`Mode Server` は Unity ウィンドウを開きません。`Encounter meadow|boss` は party size 2 で完全な権威サーバーへ渡します。どちらも既存通常 character API と MySQL データを使い、`Ensure-CharacterStore.ps1` を呼びます。`-UseRunningStore` は代わりに認証済み通常 health が一致する管理下の既存 API を要求します。API/database/config のコピー・reset・migration はしません。

로컬 플레이는 유효한 localhost 개발용 PFX를 재사용하거나 기존 OpenSSL 도구로 비공개 `.local/development-tls-*` 디렉터리에 새 7일 인증서를 생성한다. `-TlsSettingsPath`는 명시적인 기존 개발 설정을, `-OpenSslPath`는 생성이 필요할 때 설치된 OpenSSL 실행 파일을 선택한다. 인증서는 게임 TLS에만 pin하며 root trust를 설치하지 않는다. 로컬 계정 HTTP는 IPv4 loopback에 유지한다. 신뢰하는 로그인 gateway를 쓴다면 `-AuthUrl https://...`를 제공한다.

ローカル play は有効な localhost 開発用 PFX を再利用するか、既存 OpenSSL ツールで非公開 `.local/development-tls-*` directory に新しい 7 日間証明書を作ります。`-TlsSettingsPath` は明示的な既存開発設定、`-OpenSslPath` は生成が必要な場合のインストール済み OpenSSL 実行ファイルを選びます。証明書を pin するのはゲーム TLS だけで、root trust はインストールしません。ローカル account HTTP は IPv4 loopback に保ちます。信頼できる login gateway を使う場合は `-AuthUrl https://...` を渡します。

`-NoRuntimeCache`는 외장 드라이브 빌드에서 Unity 클라이언트를 실행하며, 없으면 기존 PC별 runtime cache를 사용한다. 독립 .NET 실행 파일에는 Unity 서버 cache가 없다. `-CheckOnly`는 정적 경로/옵션만 검사하며 프로세스 시작·API 접속·TLS 파일 생성·cache 채우기를 하지 않는다. 준비 상태·연결·저장·인증서 유효성·LAN 접근 가능성을 증명하지 않는다.

`-NoRuntimeCache` は外付けドライブの build から Unity client を実行し、指定しなければ既存の PC 別 runtime cache を使います。独立 .NET 実行ファイルに Unity server cache はありません。`-CheckOnly` は静的 path/option だけを検査し、process 開始・API 接続・TLS file 作成・cache 生成はしません。readiness、接続、保存、証明書の有効性、LAN 到達性は証明しません。

## LAN

## LAN

다른 PC를 연결하기 전에 기존 Caddy 공개 인증 gateway를 구성한다. HTTPS origin은 그 PC에서 접근 가능하고 Windows가 인증서를 신뢰해야 한다. API와 인증된 제어 listener는 loopback에 두고 기존 공개 인증 허용 목록만 전달한다. 내부 캐릭터·resolve·health·shutdown 경로를 노출하지 않는다. 게임 인증서 pin은 별도 HTTPS 계정 연결을 pin하지 않는다.

他 PC を接続する前に既存 Caddy 公開認証 gateway を設定します。HTTPS origin は他 PC から到達可能で、証明書が Windows に信頼される必要があります。API と認証済み control listener は loopback に置き、既存公開認証 allowlist だけを転送します。内部 character、resolve、health、shutdown route は公開しません。ゲーム証明書 pin は別の HTTPS account 接続を pin しません。

SAN이 지정한 `TlsHost`를 포함하는 게임 인증서를 사용한다. 선택적 SHA256 `-Pin`은 인증서 이름 검사를 유지하면서 명시적으로 검증한 자체 서명 게임 체인을 허용하며 TLS 검증을 끄지 않는다. localhost 개발 인증서를 원격 hostname/IP에 사용하지 않는다. 신뢰하는 경로로 pin을 받아 검증하고, PFX·TLS 설정/암호·store config·API key·계정 세션·DB·제어 token은 클라이언트 PC에 복사하지 않는다.

SAN に指定した `TlsHost` を含むゲーム証明書を使います。任意の SHA256 `-Pin` は証明書名検査を維持したまま、明示的に検証済みの自己署名 game chain を許可します。TLS 検証を無効にするものではありません。localhost 開発証明書を remote hostname/IP に使いません。信頼できる経路で pin を取得・検証し、PFX、TLS 設定/パスワード、store config、API key、account session、database、control token を client PC にコピーしません。

```powershell
# Server: set the PFX password in this process using your private secret mechanism.
# Do not place its value on the command line or in this document.
pwsh -NoProfile -File Tools/Start-DotNetLanServer.ps1 -BindAddress 192.168.3.10 -Port 7777 -AuthUrl https://login.example.test -TlsHost game.example.test -TlsPfxPath C:/private/game.pfx

# Each client: use the server's private IP or DNS name, not 0.0.0.0.
pwsh -NoProfile -File Tools/Start-DotNetLanClient.ps1 -HostAddress 192.168.3.10 -Port 7777 -AuthUrl https://login.example.test -TlsHost game.example.test -Profile default
# For an independently verified self-signed game certificate, also supply -Pin <64-hex-SHA256>.
```

예제 주석의 뜻: 서버 PFX 암호는 비공개 비밀 관리 방식으로 현재 프로세스에 설정하고, 값은 명령행이나 이 문서에 넣지 않는다. 각 클라이언트는 `0.0.0.0` 대신 서버의 사설 IP/DNS 이름을 사용한다. 독립 검증한 자체 서명 게임 인증서라면 `-Pin <64-hex-SHA256>`도 제공한다.

コード例のコメントの意味：server PFX password は非公開の秘密管理手段で現在 process に設定し、値を command line や本書に入れません。各 client は `0.0.0.0` でなく server の private IP/DNS 名を使います。独立検証済み自己署名 game 証明書には `-Pin <64-hex-SHA256>` も指定します。

LAN 서버는 비공개 `-TlsSettingsPath`와 `-TlsPasswordEnvironmentName`도 받는다. 서버 쪽 `AuthUrl`, `TlsHost`, `Pin`은 클라이언트 설정을 설명할 뿐 Caddy를 재구성하거나 PFX에서 읽은 인증서를 바꾸지 않는다. 원격 클라이언트는 실수로 준 클라이언트 자신의 HTTP origin을 포함해 HTTP 로그인 origin을 거부한다. TLS 우회·방화벽 변경·신뢰 저장소 수정은 하지 않는다. 실제 `.NET` 서버 실행 파일과 포트에 대해 Private network/LocalSubnet으로 제한한 명시적 승인 TCP 규칙을 추가해야 한다. 기존 Unity 실행 파일 규칙은 이 프로세스를 포함하지 않는다.

LAN server は非公開 `-TlsSettingsPath` と `-TlsPasswordEnvironmentName` も受け取ります。server 側 `AuthUrl`、`TlsHost`、`Pin` は client 設定を説明するもので、Caddy の再設定や PFX から読み込む証明書の変更はしません。remote client は、誤って指定した client 自身の HTTP origin を含む HTTP login origin を拒否します。TLS bypass、firewall 変更、trust-store 修正は行いません。実際の `.NET` server 実行ファイル・port に対し、Private network/LocalSubnet に限定した明示承認済み TCP rule が必要です。既存 Unity 実行ファイル用 rule はこの process に適用されません。

## 정상 종료와 소유권

## 正常終了と所有関係

실행기 콘솔을 열어 둔다. Enter를 누르면 소유한 Unity 클라이언트에 정상 종료 요청을 쓰고 종료를 기다린 뒤, 소유한 .NET 제어 shutdown을 인증하고 영속 변경/최종 생명 상태의 drain 완료를 기다린다. 기존 API와 MySQL은 계속 제공한다. 매 실행은 `Verification` 아래 새 비공개 세션 디렉터리를 만들고 정확한 PID/시작 시각 소유권, ready, logs, 최종 호스트 보고서와 비밀값을 제거한 `session-summary.json`을 둔다. 프로세스 이름 전체 종료·강제 종료·CLI 암호 인자·평문 암호 영수증은 사용하지 않는다.

launcher console を開いたままにします。Enter を押すと所有する Unity client に正常停止要求を書き、終了を待ちます。その後、所有する .NET control shutdown を認証し、永続 mutation/final vitals の drain を待ちます。既存 API と MySQL は利用可能に保ちます。起動ごとに `Verification` 以下へ新しい非公開 session directory を作り、正確な PID/start-time 所有関係、ready、log、final host report、秘密情報を除去した `session-summary.json` を保存します。process 名による一括終了、force kill、password の CLI 引数、平文 password の記録は使いません。

세션 영수증은 시작 성공과 정상 종료 성공을 구분한다. 실패한 시작의 정리로 세션이 성공한 것은 아니다. timeout은 실패/미완료 종료이며 저장 성공이 아니다. 프로세스와 비공개 영수증을 보존하고 두 번째 서버 실행이나 프로필 변경 전에 최종/대기 상태를 확인한다. 기존 PowerShell 세션에서 실행기를 부르면 `-SessionContextRef ([ref]$session)`으로 소유한 live handle도 명시적 후속 작업용으로 유지할 수 있으며 먼저 `$session=$null`을 초기화한다. `DotNetPlay.ps1`와 `DotNetAuthority.ps1`는 dot-source 때 함수만 정의한다. 후자의 `Start-DotNetAuthority`는 필수 context reference를 항상 받는다.

session 記録は起動成功と正常終了成功を区別し、失敗起動の cleanup で session を成功にはしません。timeout は失敗/未完了の shutdown であり、保存成功ではありません。process と非公開記録を保持し、2 台目の server や profile 変更前に final/pending 状態を調べます。既存 PowerShell session から呼ぶ場合、`-SessionContextRef ([ref]$session)` で所有する live handle も明示的な後続処理用に保持できます。先に `$session=$null` を初期化します。`DotNetPlay.ps1` と `DotNetAuthority.ps1` は dot-source 時に関数だけを定義します。後者の `Start-DotNetAuthority` は必須 context reference を常に受け取ります。

실제 전체 게임 진행, Linux 서비스 운영, 서로 다른 두 물리 PC의 LAN/인증서/방화벽 검사는 별도 검증·배포 실행에서 담당한다. 수락 전까지 기존 진입점은 현재 기본값을 유지한다.

実際の全ゲーム進行、Linux service 運用、2 台の別物理 PC での LAN/証明書/firewall 検査は、別の検証・デプロイ実行が担当します。その検査が受け入れられるまで、既存 entrypoint は現在の既定値を保ちます。
