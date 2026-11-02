# herdr-settings

[Herdr](https://herdr.dev) 0.1.12 (Windows) 개인 설정: `sw-lee` 레이아웃.

- 왼쪽 사이드바: Space Folders(스페이스를 폴더로 묶기), Agents
- 상단 `+` 메뉴: Open in VS Code (`ctrl-shift-g`)
- 오른쪽 패널: Files, Changes
- 테마 `vscode-wezterm`: UI는 VS Code Dark Modern, 터미널은 WezTerm 기본 팔레트
- WSL(Ubuntu-22.04)을 SSH로 연결한 Herdr 머신 `wsl`

## 구성

| 경로 | 내용 |
|---|---|
| `layouts/sw-lee.json` | 레이아웃 (Herdr `main` 복제본, spaces는 `space-folders`, 탭 바는 `my-tabs`) |
| `themes/vscode-wezterm.json` | 테마 |
| `keys.json`, `settings.json` | 키와 개인 설정 (D2Coding 글꼴, zoom 1.1) |
| `mods/open-vscode/` | [CacaoRick/herdr-open-vscode](https://github.com/CacaoRick/herdr-open-vscode) 그대로 (MIT, `LICENSE`) |
| `patches/space-folders.patch` | [jankeesvw/herdr-space-folders](https://github.com/jankeesvw/herdr-space-folders) `aad4a34` 대비 변경: Windows 경로·파일 저장, WSL 폴더 선택, 원격 스페이스를 `$HOME`에서 시작, `space-folders.new` |
| `patches/my-tabs.patch` | Herdr `tabs` 대비: `+`가 `my-new-menu`를 연다 |
| `patches/my-new-menu.patch` | Herdr `new-menu` 대비: Open in VS Code 행, New space는 `space-folders.new` |
| `wsl/` | WSL sshd 설정과 Windows `~/.ssh/config`의 `Host wsl` 항목 |

space-folders는 원본에 라이선스가 없고, `my-tabs`·`my-new-menu`는 Herdr 기본 모드의 복제본이라 수정본 전체 대신 patch만 둔다.
폴더 구조 데이터(`~/.config/herdr-space-folders.json`)는 개인 경로가 들어 있어 넣지 않았다.

## 복원 (Git Bash)

```sh
H=~/.config/herdr-app
git clone https://github.com/Sudo42b/herdr-settings.git ~/herdr-settings && cd ~/herdr-settings

cp layouts/sw-lee.json $H/layouts/
cp themes/vscode-wezterm.json $H/themes/
cp keys.json settings.json $H/            # 기존 keys.json/settings.json을 덮어쓴다
cp -r mods/open-vscode $H/mods/

# Herdr 기본 모드 복사 (mod.clone은 경로를 미리 바꿔 patch와 충돌하므로 cp; mod.json 먼저)
mkdir $H/mods/my-tabs $H/mods/my-new-menu
cp $H/mods/tabs/mod.json $H/mods/my-tabs/ && cp $H/mods/tabs/*.svg $H/mods/tabs/main.js $H/mods/my-tabs/
cp $H/mods/new-menu/mod.json $H/mods/my-new-menu/ && cp $H/mods/new-menu/main.js $H/mods/my-new-menu/
# space-folders 원본 (mod.json 먼저: Herdr가 새 모드 폴더를 레이아웃에 꽂지 않도록)
git clone https://github.com/jankeesvw/herdr-space-folders.git /tmp/space-folders
git -C /tmp/space-folders checkout aad4a34
mkdir $H/mods/space-folders
cp /tmp/space-folders/mods/space-folders/mod.json $H/mods/space-folders/
cp /tmp/space-folders/mods/space-folders/main.js $H/mods/space-folders/

(cd $H && for p in space-folders my-tabs my-new-menu; do git apply "$(cygpath -w ~/herdr-settings/patches/$p.patch)"; done)
for m in space-folders my-tabs my-new-menu open-vscode; do herdr-app check $H/mods/$m; done

herdr-app do layout.switch --name sw-lee
herdr-app do app.theme --name vscode-wezterm
```

patch는 Herdr 0.1.12의 `tabs`/`new-menu` 기준이라 다른 버전에서는 맞지 않을 수 있다.

## WSL 머신

1. WSL: `sudo cp wsl/sshd_config.d-herdr-wsl.conf /etc/ssh/sshd_config.d/herdr-wsl.conf && sudo systemctl restart ssh`
   (Windows OpenSSH가 22번을 쓰고 WSL이 `networkingMode=mirrored`라 2222번, localhost 전용, 키 인증만)
2. WSL: Windows 공개키를 `~/.ssh/authorized_keys`에 추가
   `cat /mnt/c/Users/<user>/.ssh/id_ed25519.pub >> ~/.ssh/authorized_keys`
3. Windows `~/.ssh/config`에 `wsl/ssh-config-host-wsl.txt` 내용 추가, PowerShell에서 `ssh wsl uname -a` 확인
4. Claude Code 밖의 셸에서 `hc machine add wsl`
   (Claude Code 세션 안에서 띄우면 WSL herdr-core가 `CLAUDE_CODE_CHILD_SESSION`을 물려받아, 그 터미널의 `claude`가 대화 기록을 저장하지 않는다. 그랬다면 `hc machine update wsl`로 재시작)

## 알아둘 것

- Git Bash에서 `herdr-app`/`hc`에 `/home/...` 같은 리눅스 경로를 넘길 때는 `MSYS_NO_PATHCONV=1`을 붙인다. 안 그러면 `C:/Program Files/Git/home/...`으로 바뀐다.
- `sw-lee`의 `ctrl-shift-n`은 `space-folders.new`로, 사이드바를 숨기면(`ctrl-shift-b`) 동작하지 않는다.
- 복제한 모드는 Herdr 업데이트를 따라가지 않는다.
