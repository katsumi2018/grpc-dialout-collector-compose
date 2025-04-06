# grpc-dialout-collector-compose

gRPCのDial-Out方式のTelemetryデータを収集するためのdocker-compose設定です

![image](https://github.com/user-attachments/assets/63aca5f8-6c02-44b6-8f3a-7327fb2904d0)

以下の4個のDockerが動作
- pmtelemetryd : gRPC の Telemetry を受信して JSON形式ファイルに出力
  - mdt-dialout-collectorの gRPC dial-out libraries 入れて利用
  - https://github.com/network-analytics/mdt-dialout-collector/blob/main/doc/integration-with-pmtelemetryd.md
- telegraf : pmtelemetrydで受信したテレメトリーデータファイルを tail して内容の編集とPrometheusに取り込めるメトリクスにする
- Prometheus: メトリクスの収集
- Grafana : 可視化

## 構築方法
```
git clone --depth=1 https://github.com/katsumi2018/grpc-dialout-collector-compose.git
cd grpc-dialout-collector-compose

docker compose up --build
```

## 初期設定内容
- Telemetry利用ポートは 20000
- Juniper のみ有効
  - 設定変更することで Cisco / Huawai も利用化
- 取得するメトリクスは `in-octets` と `out-octets`
  - ネットワーク機器コンフィグと`telegraf.conf`の設定変更で他のメトリクスも取得可

## 動作確認したJuniperのTelemetryコンフィグ抜粋
```
set services analytics streaming-server TELEMETRY_SERVER remote-address 10.99.0.1
set services analytics streaming-server TELEMETRY_SERVER remote-port 20000
set services analytics export-profile TELEMETRY_PROFILE local-address 10.99.0.2
set services analytics export-profile TELEMETRY_PROFILE reporting-rate 30
set services analytics export-profile TELEMETRY_PROFILE format json-gnmi
set services analytics export-profile TELEMETRY_PROFILE transport grpc
set services analytics sensor TELEMETRY_SENSOR_IF_IN_OCTETS server-name TELEMETRY_SERVER
set services analytics sensor TELEMETRY_SENSOR_IF_IN_OCTETS export-name TELEMETRY_PROFILE
set services analytics sensor TELEMETRY_SENSOR_IF_IN_OCTETS resource /interfaces/interface/state/counters/in-octets
set services analytics sensor TELEMETRY_SENSOR_IF_OUT_OCTETS server-name TELEMETRY_SERVER
set services analytics sensor TELEMETRY_SENSOR_IF_OUT_OCTETS export-name TELEMETRY_PROFILE
set services analytics sensor TELEMETRY_SENSOR_IF_OUT_OCTETS resource /interfaces/interface/state/counters/out-octets
```

## 参考
- [pmacct project: IP accounting iconoclasm](http://www.pmacct.net/)
- [mdt-dialout-collector](https://github.com/network-analytics/mdt-dialout-collector)
- [Multivendor (async) gRPC dial-out collector | APNIC Blog](https://blog.apnic.net/2022/10/17/multivendor-async-grpc-dial-out-collector/)
- [Dial-Out方式のTelemetryデータを収集する(gRPC) - Qiita](https://qiita.com/k-maki/items/19c6b383fb1eb8f0690d)
