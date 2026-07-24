# 🏧 ATMS - CICD 트러블슈팅(SSH Permission Denied편)

#프로젝트 #트러블슈팅 #SSH #Jenkins #Linux #SELinux #AI #ClaudeCode

---

<br>

지난 시간에 Jenkins 컨테이너를 띄우고, GitLab과 배포 서버 양쪽에 인증을 연결해 파이프라인의 뼈대를 만들었다.

## ▶ [[🏧 ATMS - CICD 구성]]

이번 시간에는 그 파이프라인을 실제로 돌리자마자 마주친 첫 번째 벽 — **SSH Permission Denied**를 다루겠다.

---

<br>

# <font color="#76923c">증상</font>

<br>

Jenkinsfile의 Deploy 스테이지, `sshagent` 블록 안에서 배포 서버로 접속하는 순간 다음 에러가 떨어졌다.

```
deploy@10.0.20.15: Permission denied (publickey,gssapi-keyex,gssapi-with-mic).
```

키 페어는 분명히 Jenkins 컨테이너 안에서 생성해서 서버 `~/.ssh/authorized_keys`에 등록했는데도 거부당했다.

<br>

>[!tip] Claude Code와 함께 접근한 방식
> 무작정 설정을 이것저것 바꿔보기 전에, `ssh -vvv` 로 확보한 상세 로그를 그대로 Claude Code에 붙여넣고 **"Offering public key 다음에 바로 거부당하는 패턴이면 뭘 의심해야 하나"** 를 물으며 시작했다. 돌아온 방향은 "키 자체보다, 서버가 키를 신뢰하지 않는 조건(권한 · SELinux)부터 하나씩 소거하자"였다 — 그 순서를 그대로 따라갔다.

---

<br>

# <font color="#76923c">가설 1 — 키 페어가 서로 다른가?</font>

<br>

가장 먼저 의심한 건 단순한 실수 — 등록한 공개키와 실제 사용 중인 개인키가 서로 짝이 안 맞는 경우다.

```bash
# Jenkins 컨테이너: 개인키 → 공개키 도출
ssh-keygen -y -f /var/jenkins_home/.ssh/atms_deploy_key

# 양쪽 핑거프린트 비교
ssh-keygen -l -f /var/jenkins_home/.ssh/atms_deploy_key
ssh-keygen -l -f /home/deploy/.ssh/authorized_keys
```

두 SHA256 해시는 **정확히 일치**했다. 키 자체는 문제가 없다는 뜻이다.

**→ 기각.**

---

<br>

# <font color="#76923c">가설 2 — authorized_keys 줄바꿈이 깨졌나?</font>

<br>

Windows 환경에서 공개키를 복사해 붙여넣는 과정이 있었기 때문에, CRLF가 섞여 들어가 키가 깨졌을 가능성도 확인했다.

```bash
cat -A /home/deploy/.ssh/authorized_keys | grep '\^M'
```

`^M`은 보이지 않았다. 줄바꿈 오염도 아니었다.

**→ 기각.**

<br>

여기까지 확인하고 나니 슬슬 초조해졌다. **키도 맞고, 파일도 멀쩡한데 왜 거부당하는가?**

---

<br>

# <font color="#76923c">가설 3 — 홈 디렉토리 권한이 너무 열려있다</font>

<br>

SSH는 `authorized_keys`뿐 아니라 그 **상위 디렉토리들의 권한까지 깐깐하게 검사**한다. 확인해보니:

```bash
stat /home/deploy | grep Access
# (0775/drwxrwxr-x)
```

그룹(Group)에 **쓰기(w) 권한**이 열려있었다. SSH 입장에서는 "다른 그룹 계정이 이 디렉토리 내용을 바꿀 수 있다 = 신뢰할 수 없다"고 판단하고 키 인증 자체를 거부한다.

```bash
chmod 755 /home/deploy
```

>[!info] 이 chmod, 정말 안전한가?
> - 소유자(`deploy`)는 원래도 `rwx`라 변화 없음 — 기존 동작에 영향 없다.
> - 하위 디렉토리(`atms/`, `.ssh/` 등)는 그대로다.
> - 실제로 막히는 건 "같은 그룹의 다른 계정이 홈 디렉토리 **루트**에 파일을 직접 쓰는 것" 뿐이고, 애초에 리눅스 홈 디렉토리의 표준 권한이 755다. 775였던 쪽이 비정상이었다.

권한을 고치고 다시 시도했다.

**...여전히 `Permission denied`.**

---

<br>

# <font color="#76923c">가설 4 — SELinux가 막고 있다</font>

<br>

배포 서버는 **CentOS 7**이었고, SELinux가 `Enforcing` 상태로 켜져 있었다. 이번엔 파일 권한(chmod)이 아니라 **SELinux 컨텍스트(라벨)** 를 의심했다.

```bash
ls -Z /home/deploy/.ssh/authorized_keys
```

```
unconfined_u:object_r:httpd_sys_content_t:s0
```

`ssh_home_t`여야 할 컨텍스트가 **`httpd_sys_content_t`**(웹서버용 라벨)로 지정되어 있었다. SELinux는 파일 권한과 별개로 "이 프로세스가 이 라벨의 파일에 접근해도 되는가"를 정책으로 한 번 더 검사하는데, sshd 입장에서는 이 라벨을 신뢰하지 않으니 조용히 접속을 거부한 것이다.

```bash
sudo restorecon -Rv /home/deploy/.ssh/
ls -Z /home/deploy/.ssh/authorized_keys
# → ssh_home_t 로 복원됨
```

>[!info] restorecon이 건드리는 범위
> SELinux **라벨만** 정책 기본값으로 되돌릴 뿐, 파일 내용·소유자·chmod 권한은 전혀 건드리지 않는다. `/home/deploy/.ssh/` 밖의 다른 서비스에도 영향이 없다.

<br>

다시 접속을 시도했다.

```bash
ssh -i /var/jenkins_home/.ssh/atms_deploy_key \
    -o PasswordAuthentication=no \
    deploy@10.0.20.15 "echo 접속성공"
```

```
접속성공
```

**드디어 뚫렸다.**

---

<br>

**<font color="#76923c">결론</font>**
- 키 페어 불일치 X
- authorized_keys 줄바꿈 오염 X
- 홈 디렉토리 권한(775) — 원인 O (일부)
- SELinux 컨텍스트(httpd_sys_content_t) — <u>**진짜 원인 O**</u>

하나의 에러 메시지 뒤에 원인이 네 개나 겹쳐 있을 수 있다는 걸 이번에 제대로 배웠다. 특히 SELinux는 `chmod`/`ls -l`만 봐서는 절대 드러나지 않아서, **권한을 다 맞췄는데도 여전히 거부당한다면 `ls -Z`부터 확인**하는 습관을 들이게 됐다.

---

# 다음에는, 배포에 성공한 후 곧바로 마주친 두 번째 미스터리 — 로그 한 줄 없이 뜨는 **CORS 403**을 다뤄보겠다.

## ▶ [[🏧 ATMS - CICD 트러블슈팅(CORS 403편)]]
