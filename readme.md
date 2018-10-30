# Spring Boot Actuator with Prometheus and Grafana Monitoring

## Spring Boot

### maven dependency
```
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-actuator</artifactId>
</dependency>

<dependency>
    <groupId>io.micrometer</groupId>
    <artifactId>micrometer-core</artifactId>
</dependency>

<dependency>
    <groupId>io.micrometer</groupId>
    <artifactId>micrometer-registry-prometheus</artifactId>
</dependency>
```

### Grafana 에서 Application 이름을 인식하기 위해 Configuration Bean 추가
```
#
@Configuration
public class MetricsConfig {
    
    @Value("${spring.application.name}")
    private String applicationName;

    @Bean
    public MeterRegistryCustomizer<MeterRegistry> metricsCommonTags() {
        return registry -> registry.config().commonTags("application", applicationName);
    }
    
}
```

### application.properties 
```
# actuator metrics 에서 prometheus 만 보도록 설정
# http://localhost:8080/actuator
# http://localhost:8080/actuator/prometheus
management.endpoint.metrics.enabled=true
management.endpoints.web.exposure.include=prometheus
management.endpoint.prometheus.enabled=true
management.metrics.export.prometheus.enabled=true
```

## Prometheus

### Install
- Download : https://prometheus.io/download/
- prometheus.yml Job 설정
    ```
      - job_name: 'spring-boot-application'
        metrics_path: '/actuator/prometheus'
        scrape_interval: 5s
        static_configs:
        - targets: ['localhost:8080']
    ```

### systemd configuration
- /etc/systemd/system/prometheus.service
    ```
    [Unit]
    Description=Prometheus Server
    After=network-online.target
    
    [Service]
    User=root
    Restart=on-failure
    ExecStart=/usr/local/prometheus-2.4.3.linux-amd64/prometheus \
      --config.file=/usr/local/prometheus-2.4.3.linux-amd64/prometheus.yml
      --storage.tsdb.path=/usr/local/prometheus-2.4.3.linux-amd64/data
    
    [Install]
    WantedBy=multi-user.target
    ```

    ```
    sudo systemctl daemon-reload
    sudo systemctl enable prometheus
    sudo systemctl start prometheus
    ```

## Grafana

### Install
- Downlad : https://grafana.com/grafana/download
- Download Dashboard : https://grafana.com/dashboards/6756
- Account : localhost:3000, admin / admin


## Reference 
- https://dzone.com/articles/monitoring-using-spring-boot-2-prometheus-and-graf?fromrel=true
- https://dzone.com/articles/monitoring-using-spring-boot-20-prometheus-and-gra

