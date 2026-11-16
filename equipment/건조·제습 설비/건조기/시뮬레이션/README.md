# 건조기 시뮬레이션

## 실행 파일 다운로드

| 파일 | 용도 |
| --- | --- |
| [분무건조기 Windows 시연 ZIP](./SprayDryer.zip?raw=true) | Isaac Sim 장면과 실행 스크립트, BaSyx·PostgreSQL·Grafana 설정, 웹 3D 화면과 설치 가이드 |
| [실행용 AASX](./SprayDryer.aasx?raw=true) | BaSyx 또는 AASX 뷰어에서 `DRYER-SIM-01`의 서브모델과 KPI 매핑 조회 |

Windows에 [Python 3.12 64비트](https://www.python.org/downloads/windows/), [Docker Desktop](https://docs.docker.com/desktop/setup/install/windows-install/), [Isaac Sim 6.2 Workstation](https://docs.isaacsim.omniverse.nvidia.com/6.2.0/installation/install_workstation.html)을 설치합니다. Docker Desktop을 Linux 컨테이너 모드로 실행하고, ZIP을 푼 `Mist_Dryer_Windows_Demo` 폴더에서 아래 명령을 실행합니다. `C:\isaacsim`은 `python.bat`가 있는 설치 경로로 바꿉니다.

```powershell
.\scripts\Initialize-DryerTwin.ps1 -IsaacRoot 'C:\isaacsim'
.\scripts\Start-DryerTwin.ps1 -Batches 2
```

시작 스크립트가 BaSyx에 실행용 AASX를 등록합니다. [BaSyx Viewer](http://127.0.0.1:3015/aasviewer)에서 속성을 열고, [Grafana](http://127.0.0.1:3014/d/mist-dryer/mist-dryer)에서 예열·연속 분무·냉각·완료와 KPI를 확인합니다. Isaac Sim 없이 데이터를 재현하는 방법과 종료 명령은 ZIP 안 `README.md`에 있습니다.

## 수집 신호와 KPI 연결 예시

아래 값은 실증 대상 수집 신호 사양에 맞춰 생성한 Isaac Sim 시연 기록입니다. 분무 시작 후 6모의분 시점의 샘플을 예로 들었습니다.

| 수집 신호 예시 | AAS의 현재값 속성 |
| --- | --- |
| 운전중 `1` / 정지중 `0` | `OperationalData.CurrentOperation.JobContext.OperatingState`에 `true`로 기록 |
| 입구 온도 `111.93 °C` / 설정 `112 °C` | `OperationalData.AirHandlingOperation.AirConditions`의 `InletAirTemperatureActual`과 `InletAirTemperatureSetpointActual` |
| 출구 온도 `84.96 °C` | 같은 컬렉션의 `OutletAirTemperatureActual` |

온도는 Grafana에서 공정 추세로 확인합니다. 두 배치의 모의 운전 상태를 합산한 `OperatingTime`은 아래 KPI의 공통 입력입니다.

| KPI | AAS 입력과 계산 예시 |
| --- | --- |
| 설비 가동률 (`AvailabilityRate`) | `KPIRecord.OperatingTime` 3,600초 ÷ `ProcessVariablesForManufacturingKPICalculation.PlannedBusyTime.currentValue` 3,600초 × 100 = **100%** |
| 시간당 생산량 (`UnitsPerHour`) | `KPIRecord.CompletedOutputQuantity`의 합성 완료 수량 76개 ÷ `KPIRecord.OperatingTime` 1시간 = **76개/시간** |

브리지는 집계 때 BaSyx에서 입력 속성과 `AssetInterfacesMappingConfiguration.MappingConfigurations`의 `Transformation`에 저장된 Lua 산식을 읽습니다. 결과는 `TimeSeries.Segments.InternalSegmentKPI.Records.KPIRecord`에 지표별 속성으로 표시되고, 전체 집계 이력은 PostgreSQL에 저장되어 Grafana에서 조회합니다.

## 건조기 장비 설명

<a href="https://www.youtube.com/watch?v=_2xbALzV0Dg"><img src="https://img.youtube.com/vi/_2xbALzV0Dg/mqdefault.jpg" width="480" alt="건조기 장비 설명 영상"></a>

이미지를 누르면 [YouTube](https://www.youtube.com/watch?v=_2xbALzV0Dg)에서 재생됩니다.

## 건조기 시뮬레이션

<a href="https://www.youtube.com/watch?v=ftLsYC0Sgu8"><img src="https://img.youtube.com/vi/ftLsYC0Sgu8/mqdefault.jpg" width="480" alt="건조기 시뮬레이션 영상"></a>

이미지를 누르면 [YouTube](https://www.youtube.com/watch?v=ftLsYC0Sgu8)에서 재생됩니다.
