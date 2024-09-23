# WooriFISA-Linux 평균 부하 분석
**Linux 시스템의 평균 부하(Average Load)** 를 분석하고, 임계치를 초과할 경우 경고 메시지를 출력하는 방법을 다룹니다. 평균 부하가 시스템 성능에 미치는 영향을 실습을 통해 배울 수 있습니다.
<h2 style="font-size: 25px;"> 🗽 개발 팀원<br>
  
|<img src="https://avatars.githubusercontent.com/u/175371231?v=4" width="150" height="150"/>|<img src="https://avatars.githubusercontent.com/u/82391356?v=4" width="150" height="150"/>|
|:-:|:-:|
|[@오재웅](https://github.com/ohwoong2)|[@이정민](https://github.com/jjeong1015)

## 목표
1. Linux의 평균 부하 개념 이해<br>
2. uptime 명령어를 사용하여 시스템의 평균 부하를 확인하는 방법 학습<br>
3. 평균 부하를 CPU 코어 수와 비교하여 과부하 상태를 분석하는 방법 실습<br>
4. 실시간으로 평균 부하를 모니터링하고 임계치를 초과할 경우 경고를 발생시키는 Bash 스크립트 작성

### 평균 부하란?
Linux의 평균 부하는 CPU 사용 또는 I/O 대기를 포함하여, 실행 중이거나 실행 대기 중인 프로세스의 평균 수를 나타냅니다. 이는 CPU 사용률뿐 아니라 시스템 전체의 자원 상태를 파악하는 중요한 지표입니다.

#### 평균 부하 확인 방법
0. stress 도구 설치
```bash
apt install stress sysstat
```
1. 기본 평균 부하 확인
```bash
$ uptime
12:34:56 up 1 day, 4:32,  3 users,  load average: 0.42, 0.55, 0.75
# 0.42: 최근 1분 동안의 평균 부하.
# 0.55: 최근 5분 동안의 평균 부하.
# 0.75: 최근 15분 동안의 평균 부하.
# 위의 결과를 시스템의 CPU 코어 수와 비교하여 시스템의 부하 상태를 파악할 수 있습니다.
```

실습
1. 평균 부하 확인 및 과부하 경고 스크립트
<br> **시스템의 평균 부하를 확인하고, CPU 코어 수와 비교하여 시스템이 과부하 상태인지 분석**합니다.

```bash
#!/bin/bash
# check_average_load.sh
# CPU 코어 수를 확인
cpu_cores=$(grep -c 'model name' /proc/cpuinfo)

# 현재 평균 부하를 가져옴
load_average=$(uptime | awk -F'load average:' '{ print $2 }' | cut -d',' -f1 | xargs)

# 평균 부하와 CPU 코어 수를 출력
echo "현재 평균 부하: $load_average"
echo "CPU 코어 수: $cpu_cores"

# 평균 부하가 CPU 코어 수보다 높으면 경고 메시지 출력
if (( $(echo "$load_average > $cpu_cores" | bc -l) )); then
  echo "경고: 시스템이 과부하 상태입니다!"
else
  echo "시스템이 정상적으로 운영되고 있습니다."
fi
```

![11_720](https://github.com/user-attachments/assets/0dc5affb-953a-4403-8a93-addcdbe06ca6)
```bash
# 권한 부여
chmod u+x check_average_load.sh
# 실행
sudo ./check_average_load.sh
```

2. 부하 추세 분석 스크립트
<br> **1분, 5분, 15분 동안의 평균 부하를 분석하고, 최근 부하 추세를 확인**합니다.

```bash
#!/bin/bash
# analyze_load_trends.sh
# 현재 시스템의 평균 부하를 가져옴
load_average=$(uptime | awk -F'load average:' '{ print $2 }')

# 1분, 5분, 15분 평균 부하를 각각 변수로 저장
load_1=$(echo $load_average | cut -d',' -f1 | xargs)
load_5=$(echo $load_average | cut -d',' -f2 | xargs)
load_15=$(echo $load_average | cut -d',' -f3 | xargs)

# 부하 출력
echo "1분 평균 부하: $load_1"
echo "5분 평균 부하: $load_5"
echo "15분 평균 부하: $load_15"

# 부하 추세 분석
if (( $(echo "$load_1 > $load_5" | bc -l) )); then
  echo "부하가 최근 1분 동안 증가했습니다."
else
  echo "부하가 최근 1분 동안 감소했습니다."
fi
```

![22_720](https://github.com/user-attachments/assets/7b0319ff-6b52-482d-b7f4-4d6e6c70243e)

```bash
# 권한 부여
chmod u+x analyze_load_trends.sh
# 실행
sudo ./analyze_load_trends.sh
```

3. 부하 모니터링 및 경고 시스템
<br> **주기적으로 평균 부하를 확인하고, 설정된 임계치를 초과하면 경고 메시지를 출력**합니다. cron 작업과 결합하여 주기적인 시스템 모니터링에 활용할 수 있습니다.

```bash
#!/bin/bash
# load_monitor.sh
# 시스템의 평균 부하를 모니터링하고, 지정된 임계치를 초과하면 경고 메시지를 출력합니다.

# 설정: 경고를 발생시킬 평균 부하 임계치
THRESHOLD=4.00

# 현재 시스템의 평균 부하 (1분 기준) 확인
load_average=$(uptime | awk -F'load average:' '{ print $2 }' | cut -d',' -f1 | xargs)

# 부하 출력
echo "현재 1분 평균 부하: $load_average"
echo "설정된 임계치: $THRESHOLD"

# 평균 부하가 임계치를 초과하면 경고
if (( $(echo "$load_average > $THRESHOLD" | bc -l) )); then
  echo "경고: 평균 부하가 $THRESHOLD를 초과했습니다!"
else
  echo "시스템 부하가 정상 범위 내에 있습니다."
fi
```

![33_720](https://github.com/user-attachments/assets/c55cdd15-91c8-4d33-93a3-639428e19c8d)


```bash
# 권한 부여
chmod u+x load_monitor.sh
# 실행
sudo ./load_monitor.sh
```

cron 작업을 추가하여 주기적으로 부하를 모니터링할 수 있습니다.

```bash
* * * * * /path/to/load_monitor.sh >> /var/log/load_monitor.log
```

## 결론

이번 프로젝트를 통해 Linux 시스템의 평균 부하를 이해하고, 이를 모니터링하는 방법에 대해 배웠습니다.<br>
평균 부하는 CPU 사용 및 I/O 대기를 포함하여 시스템의 자원 상태를 파악하는 중요한 지표로, 이를 통해 시스템의 성능을 효과적으로 관리할 수 있습니다.<br>
부하 추세 분석과 주기적인 모니터링을 통해 시스템의 상태를 지속적으로 점검할 수 있는 기반을 마련하였습니다. <br>
이러한 모니터링 시스템은 시스템 관리자가 효율적으로 리소스를 관리하고, 문제가 발생하기 전에 미리 대처할 수 있는 중요한 도구가 될 것입니다.
