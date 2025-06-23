# GitPushingTest

# Description

## Mimikatz 개요
Mimikatz는 Benjamin Delpy가 개발한 오픈소스 보안 도구로, Windows 운영체제의 인증 메커니즘을 연구하고 취약점을 입증하기 위해 만들어졌습니다. 원래는 보안 연구 목적으로 개발되었지만, 현재는 침투 테스트와 악성 공격 모두에서 널리 사용되고 있습니다.
주요 기능
자격 증명 추출
- LSASS(Local Security Authority Subsystem Service) 프로세스 메모리에서 평문 패스워드, 해시, 커베로스 티켓 추출
- WDigest, MSV, Kerberos, SSP 등 다양한 인증 패키지에서 자격 증명 수집

## Windows 인증 패키지별 Mimikatz 추출 정보

### WDigest 인증 패키지

| 항목 | 내용 |
|---|---|
| **Mimikatz 명령어** | `sekurlsa::wdigest` |
| **추출 가능한 정보** | • 평문 패스워드<br>• 사용자명<br>• 도메인명<br>• 로그온 세션 ID |
| **메모리 위치** | LSASS 프로세스 내 WDigest SSP 영역 |
| **레지스트리 설정** | `HKLM\SYSTEM\CurrentControlSet\Control\SecurityProviders\WDigest\UseLogonCredential` |
| **기본 상태** | • Windows 8.1/2012R2 이전: 활성화<br>• Windows 10/2016 이후: 비활성화 |
| **활성화 조건** | • 레지스트리 값이 1로 설정<br>• HTTP 인증 사용 시<br>• 특정 애플리케이션 요구 시 |
| **Windows 사용 이유** | • **HTTP Digest 인증 지원**: 웹 서버 Basic 인증의 보안 강화 버전<br>• **IIS 웹 서버 인증**: Internet Information Services에서 사용<br>• **RFC 2617 표준 준수**: HTTP Digest Access Authentication 구현<br>• **Challenge-Response 방식**: 네트워크 상에서 패스워드를 평문으로 전송하지 않기 위함<br>• **레거시 호환성**: 기존 웹 애플리케이션과의 호환성 유지 |
| **공격자 사용 이유** | • **평문 패스워드 획득**: 가장 직접적인 자격증명 형태<br>• **패스워드 재사용 공격**: 다른 시스템에서 동일 패스워드 시도<br>• **소셜 엔지니어링**: 실제 사용자 패스워드 패턴 분석<br>• **추가 인증 우회**: 2차 인증이 없는 다른 서비스 접근 |
| **보안 위험도** | **높음** - 평문 패스워드 노출 |
| **방어 방법** | • UseLogonCredential 값을 0으로 설정<br>• GPO로 정책 강제 적용 |

### MSV (Microsoft Authentication Package)

| 항목 | 내용 |
|---|---|
| **Mimikatz 명령어** | `sekurlsa::msv` |
| **추출 가능한 정보** | • NTLM 해시<br>• LM 해시 (레거시)<br>• 사용자명<br>• 도메인명<br>• 로그온 세션 정보 |
| **메모리 위치** | LSASS 프로세스 내 MSV1_0 패키지 영역 |
| **인증 프로토콜** | • NTLM v1/v2<br>• LM (레거시) |
| **저장 형태** | • NT 해시 (MD4)<br>• LM 해시 (DES 기반) |
| **사용 시나리오** | • 로컬 로그온<br>• 워크그룹 환경<br>• NTLM 인증 |
| **Windows 사용 이유** | • **로컬 인증 처리**: 로컬 SAM 데이터베이스 사용자 인증<br>• **워크그룹 환경 지원**: 도메인이 없는 환경에서의 네트워크 인증<br>• **NTLM 호환성**: 레거시 시스템과 애플리케이션 지원<br>• **빠른 인증 처리**: Kerberos보다 단순한 인증 과정<br>• **네트워크 공유 접근**: SMB/CIFS 파일 공유 인증<br>• **웹 애플리케이션 통합**: IIS NTLM 인증 지원 |
| **공격자 사용 이유** | • **Pass-the-Hash 공격**: 평문 패스워드 없이 인증 가능<br>• **Lateral Movement**: 네트워크 내 다른 시스템 접근<br>• **크래킹 대상**: Rainbow Table이나 브루트포스 공격<br>• **권한 상승**: 높은 권한 계정의 해시 획득 시 즉시 사용<br>• **지속성 확보**: 패스워드 변경 없이 장기간 사용 가능 |
| **보안 위험도** | **중간** - Pass-the-Hash 공격 가능 |
| **방어 방법** | • Kerberos 인증 우선 사용<br>• NTLM 제한 정책 적용 |

### Kerberos 인증 패키지

| 항목 | 내용 |
|---|---|
| **Mimikatz 명령어** | `sekurlsa::kerberos` |
| **추출 가능한 정보** | • TGT (Ticket Granting Ticket)<br>• TGS (Ticket Granting Service)<br>• Kerberos 키 (AES, RC4)<br>• 세션 키 |
| **메모리 위치** | LSASS 프로세스 내 Kerberos SSP 영역 |
| **티켓 형태** | • TGT: krbtgt 서비스용<br>• TGS: 특정 서비스용 |
| **암호화 타입** | • AES128/256<br>• RC4-HMAC<br>• DES (레거시) |
| **티켓 수명** | • TGT: 기본 10시간<br>• TGS: 기본 10시간 |
| **공격 기법** | • Golden Ticket<br>• Silver Ticket<br>• Pass-the-Ticket |
| **Windows 사용 이유** | • **Single Sign-On (SSO)**: 한 번 인증으로 모든 도메인 리소스 접근<br>• **강력한 보안**: 상호 인증, 티켓 기반 보안<br>• **확장성**: 대규모 엔터프라이즈 환경에 적합<br>• **네트워크 효율성**: 반복 인증 없이 서비스 접근<br>• **Active Directory 통합**: Windows 도메인 환경의 기본 인증<br>• **위임 지원**: 서비스 간 인증 위임 가능<br>• **타임스탬프 보안**: 리플레이 공격 방지 |
| **공격자 사용 이유** | • **Golden Ticket**: krbtgt 해시로 도메인 관리자 권한 획득<br>• **Silver Ticket**: 특정 서비스 접근을 위한 위조 티켓<br>• **Pass-the-Ticket**: 획득한 티켓으로 직접 인증<br>• **장기간 접근**: 티켓 수명 동안 지속적 접근<br>• **탐지 회피**: 정상적인 Kerberos 트래픽으로 위장<br>• **권한 상승**: 높은 권한의 서비스 티켓 획득 |
| **보안 위험도** | **높음** - 장기간 유효한 티켓 |
| **방어 방법** | • 정기적 krbtgt 패스워드 변경<br>• AES 암호화 강제<br>• 티켓 수명 단축 |

### SSP (Security Support Provider)

| 항목 | 내용 |
|---|---|
| **Mimikatz 명령어** | `sekurlsa::ssp` |
| **추출 가능한 정보** | • 평문 패스워드<br>• NTLM 해시<br>• 도메인 정보<br>• 인증 컨텍스트 |
| **메모리 위치** | LSASS 프로세스 내 SSP 영역 |
| **주요 SSP** | • NTLMSSP<br>• Kerberos<br>• Negotiate<br>• Digest |
| **인증 방식** | • 통합 Windows 인증<br>• 협상 인증 |
| **저장 조건** | • SSP 사용 애플리케이션<br>• 서비스 인증 시 |
| **Windows 사용 이유** | • **통합 Windows 인증**: 웹 애플리케이션과 Windows 인증 통합<br>• **프로토콜 협상**: 최적의 인증 방법 자동 선택<br>• **애플리케이션 지원**: IIS, SQL Server 등 서버 애플리케이션 인증<br>• **개발자 편의성**: SSPI API를 통한 간편한 인증 구현<br>• **다중 인증**: 하나의 세션에서 여러 인증 방식 지원<br>• **보안 컨텍스트**: 인증 상태 및 보안 정보 관리 |
| **공격자 사용 이유** | • **다양한 자격증명**: 여러 인증 방식의 자격증명 동시 획득<br>• **애플리케이션 타겟팅**: 특정 서비스의 인증 정보 획득<br>• **컨텍스트 정보**: 인증 과정의 상세 정보 분석<br>• **서비스 접근**: 웹 애플리케이션이나 데이터베이스 직접 접근<br>• **권한 분석**: 각 서비스별 권한 레벨 파악 |
| **보안 위험도** | **중간** - 컨텍스트 의존적 |
| **방어 방법** | • SSP 사용 최소화<br>• 강력한 인증 방식 사용 |

### LiveSSP (Microsoft Live SSP)

| 항목 | 내용 |
|---|---|
| **Mimikatz 명령어** | `sekurlsa::livessp` |
| **추출 가능한 정보** | • Microsoft 계정 토큰<br>• Live ID 자격증명<br>• OAuth 토큰 |
| **메모리 위치** | LSASS 프로세스 내 LiveSSP 영역 |
| **사용 시나리오** | • Microsoft 계정 로그인<br>• Office 365 인증<br>• Windows Store 앱 |
| **토큰 타입** | • Access Token<br>• Refresh Token<br>• ID Token |
| **Windows 사용 이유** | • **Microsoft 계정 통합**: Hotmail, Outlook.com, Xbox Live 등 통합 로그인<br>• **클라우드 서비스 접근**: OneDrive, Office 365, Azure 서비스<br>• **Single Sign-On**: 여러 Microsoft 서비스 간 SSO<br>• **개인 설정 동기화**: 사용자 설정, 테마, 즐겨찾기 동기화<br>• **앱 스토어 인증**: Windows Store 앱 구매 및 설치<br>• **Modern Authentication**: OAuth 2.0 기반 현대적 인증 |
| **공격자 사용 이유** | • **클라우드 리소스 접근**: Office 365, OneDrive 등 클라우드 서비스<br>• **개인정보 획득**: 이메일, 연락처, 파일 등 개인 데이터<br>• **토큰 재사용**: Refresh Token으로 지속적 접근<br>• **다중 서비스 접근**: 하나의 토큰으로 여러 Microsoft 서비스<br>• **기업 데이터 접근**: 회사 Office 365 환경 침투 |
| **보안 위험도** | **중간** - 클라우드 서비스 접근 |
| **방어 방법** | • 조건부 액세스 정책<br>• MFA 강제 적용 |

### TsPkg (Terminal Services Package)

| 항목 | 내용 |
|---|---|
| **Mimikatz 명령어** | `sekurlsa::tspkg` |
| **추출 가능한 정보** | • RDP 세션 자격증명<br>• 평문 패스워드<br>• 터미널 서비스 인증 정보 |
| **메모리 위치** | LSASS 프로세스 내 TsPkg 영역 |
| **사용 시나리오** | • 원격 데스크톱 연결<br>• 터미널 서비스 |
| **저장 조건** | • RDP 연결 시<br>• 자격증명 위임 활성화 시 |
| **Windows 사용 이유** | • **원격 접속 지원**: RDP(Remote Desktop Protocol) 인증 처리<br>• **터미널 서비스**: 다중 사용자 원격 세션 관리<br>• **자격증명 위임**: 원격 세션에서 다른 리소스 접근<br>• **세션 보안**: 암호화된 원격 연결 보안<br>• **서버 관리**: 원격 서버 관리 및 유지보수<br>• **가상화 환경**: VDI, 터미널 서버 팜 지원 |
| **공격자 사용 이유** | • **원격 접근 권한**: RDP 자격증명으로 직접 원격 로그인<br>• **관리자 계정 탈취**: 서버 관리용 고권한 계정 획득<br>• **네트워크 확산**: 원격 접속을 통한 다른 시스템 접근<br>• **지속적 접근**: 원격 접속 권한으로 장기간 시스템 사용<br>• **은밀한 활동**: 정상적인 관리 활동으로 위장 |
| **보안 위험도** | **높음** - 원격 접근 자격증명 |
| **방어 방법** | • 자격증명 위임 비활성화<br>• NLA 강제 사용 |

### CloudAP (Cloud Authentication Package)

| 항목 | 내용 |
|---|---|
| **Mimikatz 명령어** | `sekurlsa::cloudap` |
| **추출 가능한 정보** | • Azure AD 토큰<br>• PRT (Primary Refresh Token)<br>• 하이브리드 조인 정보 |
| **메모리 위치** | LSASS 프로세스 내 CloudAP 영역 |
| **사용 시나리오** | • Azure AD 조인<br>• 하이브리드 조인<br>• SSO 인증 |
| **토큰 특징** | • 장기간 유효<br>• 다중 리소스 접근 가능 |
| **Windows 사용 이유** | • **Azure AD 통합**: 클라우드 기반 ID 관리 시스템<br>• **하이브리드 환경**: 온프레미스와 클라우드 통합 인증<br>• **Modern Workplace**: Microsoft 365 생태계 완전 활용<br>• **디바이스 관리**: Intune을 통한 디바이스 정책 관리<br>• **조건부 액세스**: 디바이스 기반 보안 정책<br>• **SSO 확장**: 수천 개의 SaaS 앱 SSO<br>• **보안 강화**: MFA, 위험 기반 인증 지원 |
| **공격자 사용 이유** | • **기업 클라우드 접근**: Office 365, Azure 리소스 전체 접근<br>• **PRT 악용**: Primary Refresh Token으로 지속적 접근<br>• **디바이스 신뢰 우회**: 신뢰된 디바이스로 위장<br>• **다중 테넌트 접근**: 여러 조직의 클라우드 환경 접근<br>• **높은 가치 타겟**: 기업의 핵심 데이터 및 시스템<br>• **탐지 회피**: 정상 디바이스로 인식되어 보안 정책 우회 |
| **보안 위험도** | **높음** - 클라우드 리소스 접근 |
| **방어 방법** | • PRT 갱신 주기 단축<br>• 디바이스 기반 조건부 액세스 |

## Pass-the-Hash 공격
- NTLM 해시를 이용한 인증 우회
- 평문 패스워드 없이도 시스템 접근 가능

## Golden/Silver Ticket 공격
- 도메인 컨트롤러의 krbtgt 계정 해시를 이용한 Golden Ticket 생성
- 서비스 계정 해시를 이용한 Silver Ticket 생성


# Execution
## 필수 
관리자 권한 체크
```
privilige debug
```

## 옵션
| 명령어 | 기능 | 추출 범위 |
|---|---|---|
| `sekurlsa::logonpasswords` | 모든 로그온 세션 자격증명 | 모든 SSP 패키지 |
| `sekurlsa::tickets` | 모든 Kerberos 티켓 | Kerberos 티켓만 |
| `sekurlsa::ekeys` | 암호화 키 추출 | Kerberos 암호화 키 |
| `sekurlsa::credman` | 자격증명 관리자 | Windows 자격증명 저장소 |

# Build
[https://github.com/gentilkiwi/mimikatz]
