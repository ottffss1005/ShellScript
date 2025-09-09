# 🖥️ Linux Shell Script 예제 - 동적 디렉토리 생성 및 확장

## 👥 팀원 소개
<table>
  <tr>
    <th>최소영</th>
    <th>신기범</th>
  </tr>
  <tr>
    <td align="center">
      <img src="https://github.com/ottffss1005.png" width="120" /><br/>
      <a href="https://github.com/ottffss1005">@ottffss1005</a>
    </td>
    <td align="center">
      <img src="https://github.com/shin-kibeom.png" width="120" /><br/>
      <a href="https://github.com/shin-kibeom">@shin-kibeom</a>
    </td>
  </tr>
</table>

---

## 📌 프로젝트 개요
**리눅스 Shell 스크립트를 이용한 동적 디렉토리 생성 예제**  
사용자가 입력한 디렉토리와 파일 이름을 기반으로 **자동 생성**하고, 파일 내용에 **날짜 정보를 추가**  
**백업 자동화 및 로그 관리 기능**을 포함한 확장 버전도 제공

---

## 🛠️ 사용된 기술
- **Shell Script**
- **Linux 명령어**: `mkdir`, `read`, `if`, `cp`, `tar`, `find`
- **Crontab** (자동화 확장 시 사용)
- **백업 및 로그 관리**

---

## 📂 기본 기능
1. **디렉토리 이름**과 **파일 이름** 사용자 입력 받기
2. 입력받은 디렉토리가 존재하지 않으면 **생성**
3. 파일이 이미 존재하는지 확인하고, 없으면 **파일 생성 후 날짜 정보 기록**

---

### ✅ 기본 코드
```bash
#!/bin/bash

# 사용자 입력 받기
read -p "디렉토리 이름 입력: " DIR_NAME
read -p "파일 이름 입력: " FILE_NAME

# 디렉토리 존재 여부 확인
if [ ! -d "$DIR_NAME" ]; then
    echo "디렉토리가 없습니다. 생성합니다: $DIR_NAME"
    mkdir -p "$DIR_NAME"
else
    echo "디렉토리가 이미 존재합니다: $DIR_NAME"
fi

# 파일 경로 설정
FILE_PATH="$DIR_NAME/$FILE_NAME"

# 파일 존재 여부 확인
if [ -f "$FILE_PATH" ]; then
    echo "해당 위치에 이미 동일한 이름의 파일이 있습니다: $FILE_PATH"
    exit 1
fi

# 날짜 문자열 생성
DATE=$(date +%m월\ %d일)

# 파일 생성 및 내용 입력
echo "오늘은 $DATE입니다." > "$FILE_PATH"
echo "파일이 생성되었습니다: $FILE_PATH"
cat "$FILE_PATH"
```

---

## 🔍 핵심 개념 설명
- **`mkdir -p`**: 디렉토리 생성 시, 상위 디렉토리가 없으면 함께 생성.
- **`read`**: 사용자 입력을 변수에 저장.
- **조건문 `if [ 조건 ]`**:
  - `-d`: 디렉토리가 존재하는지 확인.
  - `-f`: 파일이 존재하는지 확인.
- **`date` 명령어**: 현재 날짜를 문자열로 변환.

---

## 🚀 확장 기능
기본 기능 외에 **백업 및 로그 관리 기능**을 추가했습니다.  
- **로그 기록**: 디렉토리 생성 및 백업 수행 시 `backup.log`에 기록.
- **자동 백업**: 선택한 경로의 데이터를 백업 디렉토리에 저장 후 압축(`tar.gz`).
- **보관 정책**: 7일 이상 된 백업 파일 자동 삭제.
- **Crontab 연동**: 주기적 백업 자동 실행 가능.

---

### ✅ 확장 코드
```bash
#!/bin/bash
# ===== 환경 설정 =====
SRC_DIR="/home/ubuntu/"                  # 원본 디렉토리
MKDIR_ROOT="/home/ubuntu/03.sh"          # 기본 디렉토리 경로
BACKUP_ROOT="/home/ubuntu/backup"        # 백업 저장소
LOGFILE="$BACKUP_ROOT/backup.log"        # 로그 파일
RETENTION_DAYS=7                         # 보관 일수

# ===== 함수 정의 =====
basic_dir() {
    echo "생성할 디렉토리 이름을 입력하세요:"
    read name

    TARGET_DIR="$MKDIR_ROOT/$name"
    if [ -d "$TARGET_DIR" ]; then
        echo "⚠️ $name 은 이미 존재하는 디렉토리입니다."
    else
        mkdir -p "$TARGET_DIR"
        echo "✅ $name 디렉토리 생성 완료 ($TARGET_DIR)"
        echo "[$(date)] 기본 디렉토리 생성: $TARGET_DIR" >> "$LOGFILE"
    fi
}

back_dir() {
    read -p "백업할 경로를 입력하세요(기본=$SRC_DIR): " input
    SRC_DIR=${input:-$SRC_DIR}

    DATE_DIR="$(date +%Y%m%d_%H%M%S)백업_디렉토리"
    TARGET_DIR="$BACKUP_ROOT/$DATE_DIR"
    mkdir -p "$TARGET_DIR"

    # 데이터 복사
    cp -r "$SRC_DIR"/* "$TARGET_DIR"/ 2>/dev/null

    # 압축 생성
    tar -czf "$TARGET_DIR.tar.gz" -C "$BACKUP_ROOT" "$DATE_DIR"

    echo "✅ 백업 완료: $TARGET_DIR.tar.gz"
    echo "[$(date)] 백업 생성: $TARGET_DIR.tar.gz" >> "$LOGFILE"

    # 오래된 백업 삭제
    find "$BACKUP_ROOT" -type f -name "*.tar.gz" -mtime +$RETENTION_DAYS -exec rm -f {} \;
    find "$BACKUP_ROOT" -type d -mtime +$RETENTION_DAYS -exec rm -rf {} \;
}

# 메뉴 실행
while :
do
    echo "============================="
    echo "1. 기본 디렉토리 생성"
    echo "2. 백업 생성"
    echo "0. 종료"
    echo "============================="
    read -p "옵션을 선택하세요: " option

    case "$option" in
        1) basic_dir ;;
        2) back_dir ;;
        0) break ;;
        *) echo "잘못된 입력입니다." ;;
    esac
done

echo "동적 디렉토리 생성을 종료합니다"
```

---

## ⚡ 실행 방법
1. 스크립트에 실행 권한 부여:
```bash
chmod +x script.sh
```
2. 스크립트 실행:
```bash
./script.sh
```

---
