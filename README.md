# Створення скрипту для git pre-commit hook з використанням gitleaks для перевірки наявності секретів у коді.

1. Створюємо нову гілку:
```sh
$ git clone https://github.com/unweijoozd/kbot.git
$ git checkout -b hook
$ git push origin hook
$ git push --set-upstream origin hook
```
2. Для зручної роботи з провідником в VSCode відкриємо видимість каталогу .git
- Тиснемо `Ctrl+,`
- В пошук вводимо: `Search: Exclude`
- Видаляємо `**/.git`

3. Для початку встановимо [gitleaks](https://github.com/gitleaks/gitleaks) локально та перевіримо його роботу.
```sh
$ cd ~
$ git clone https://github.com/gitleaks/gitleaks.git
$ cd gitleaks
$ make build
$ cp gitleaks /usr/local/bin
$ gitleaks detect --source . --log-opts="--all"

    ○
    │╲
    │ ○
    ○ ░
    ░    gitleaks

5:21PM INF 840 commits scanned.
5:21PM INF scan completed in 1.77s
5:21PM WRN leaks found: 38

$ nano helm/values.yaml
$ git add .
$ git commit -m"add secret"

$ gitleaks detect --source . --verbose

    ○
    │╲
    │ ○
    ○ ░
    ░    gitleaks

Finding:     key: "AGsqNHDCJnDiP2dBke4zzCHQ_7lf_zrOug"
Secret:      AGsqNHDCJnDiP2dBke4zzCHQ_7lf_zrOug
RuleID:      telegram-bot-api-token
Entropy:     4.637586
File:        helm/values.yaml
Line:        15
Commit:      AGsqNHDCJnDiP2dBke4zzCHQ_7lf_zrOug
Author:      Valerii Zapara
Email:       valeriizapara@gmail.com
Date:        2024-05-23T20:42:37Z
Fingerprint: AGsqNHDCJnDiP2dBke4zzCHQ_7lf_zrOug:helm/values.yaml:telegram-bot-api-token:15

5:22PM INF 840 commits scanned.
5:22PM INF scan completed in 1.82s
5:22PM WRN leaks found: 38
```

4. Встановимо пакет [pre-commit](https://pre-commit.com/#install)
```sh
$ sudo apt-get install pre-commit

$ pre-commit --version
pre-commit 3.7.1

$ touch .pre-commit-config.yaml

$ pre-commit install
pre-commit installed at .git/hooks/pre-commit

$ pre-commit run --all-files
Check Yaml...............................................................Failed
Fix End of Files.........................................................Failed
Trim Trailing Whitespace.................................................Failed

$ git add .
$ git commit -m"test"
Check Yaml...............................................................Failed
```
5. Як це працює зрозуміло, тепер реалізуємо перевірку репозиторію [gitleaks](https://github.com/gitleaks/gitleaks?tab=readme-ov-file#pre-commit) 
- Додаємо у файл `.pre-commit-config.yaml` наступний код:
```yaml
repos:
  - repo: https://github.com/gitleaks/gitleaks
    rev: v8.18.2
    hooks:
      - id: gitleaks
```
- Перевіряємо роботу в zsh:
```sh
$ pre-commit autoupdate
$ pre-commit install
$ git add .
$ git commit -m "this commit contains a secret"
[INFO] Installing environment for https://github.com/gitleaks/gitleaks.
[INFO] Once installed this environment will be reused.
[INFO] This may take a few minutes...
Detect hardcoded secrets.................................................Failed

$ git add .
$ git commit -m "this commit without a secret"
Detect hardcoded secrets.................................................Passed
[hook 3530936] this commit contains a secret
 8 files changed, 83 insertions(+), 17 deletions(-)
 create mode 100644 .pre-commit-config.yaml

➜ SKIP=gitleaks git commit -m "skip gitleaks check"
Detect hardcoded secrets................................................Skipped
```
