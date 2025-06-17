# Docker Compose for LibreNMS

```bash
git clone https://github.com/KREONET/librenms-docker /opt/librenms
cd /opt/librenms
vi .env
vi librenms.env
vi msmtpd.env
docker compose up -d
```

## 유의 사항

### 암호 변경

`.env`의 `MYSQL_PASSWORD`와 `librenms.env`의 기본 `LIBRENMS_SNMP_COMMUNITY` 등 환경변수를 적절한 것으로 변경하고 사용한다. [Random Password Generator](https://www.google.com/search?q=Random+Password+Generator) 등을 활용하여 적절한 것으로 변경한다.

### 최신 버전 확인

LibreNMS의 [공식 compose.yml](https://github.com/librenms/docker/tree/master/examples/compose)은 `latest` 버전의 컨테이너를 사용하지만, 본 docker-compose.yml은 패치를 위하여 버전을 고정하였다.

본 Repo가 업데이트된 지 오래된 경우, 공식 [GitHub](https://github.com/librenms/librenms)과 [Docker Hub](https://hub.docker.com/r/librenms/librenms)에서 버전을 확인하고 패치를 업데이트 하고 사용해야 한다.

```yaml
  db:
    image: mariadb:10.5

  librenms:
    image: librenms/librenms:25.5.0

  dispatcher:
    image: librenms/librenms:25.5.0

  syslogng:
    image: librenms/librenms:25.5.0

  snmptrapd:
    image: librenms/librenms:25.5.0
```

### EUC-KR 패치

LibreNMS의 기본 문자코드는 `UTF-8`이다. 장비에 `EUC-KR`로 접속하고 한글로 포트의 description(ifDesrc)을 설정한 경우, 문자열이 맞지 않아 LibreNMS에서 문자열이 깨져서 나오게 된다.

본 Repo에서는 [common.php](https://github.com/librenms/librenms/blob/25.5.0/includes/common.php)과 [NetSnmpQuery.php](https://github.com/librenms/librenms/blob/25.5.0/LibreNMS/Data/Source/NetSnmpQuery.php)는 에서 다음을 추가하였다.

```diff
72a73
>     $output = mb_convert_encoding($output,"UTF-8","EUC-KR");
```

두 파일은 버전에 따라서 수정이 잦은 파일이다. 최신 버전을 사용하고자 하는 경우, 컨테이너와 동일 버전의 파일을 구하여 패치를 하고 사용해야 한다.

### Oxidized

본 Repo의 docker-compose.yml에는 LibreNMS와 함께 유용하게 활용할 수 있는 [Oxidized](https://github.com/ytti/oxidized)를 주석처리 해 두었다. [이 링크](https://github.com/ytti/oxidized/blob/master/docs/Configuration.md)를 참고하여 설정한다.

```yaml
  # oxidized:
  #   image: oxidized/oxidized:latest
  #   container_name: oxidized
  #   ports:
  #     - 8888:8888/tcp
  #   environment:
  #     CONFIG_RELOAD_INTERVAL: 600
  #   volumes:
  #     - oxi_cfg:/home/oxidized/.config/oxidized/
  #     - oxi_ssh:/home/oxidized/.ssh/
  #   restart: always

volumes:
  db:
  redis:
  libre:
  # oxi_cfg:
  # oxi_ssh:
```

### DB, phpMyAdmin 개방

NocoDB 연동이나 디버깅을 위하여 DB나 phpMyAdmin 포트를 개방할 수 있다. 다만 적절한 보안정책을 수립하여 외부에 노출되지 않아야 한다.


```yaml
  db:
    image: mariadb:10.5
    # ports:
    #   - 3306:3306/tcp

  # phpmyadmin:
  #   image: phpmyadmin/phpmyadmin:latest
  #   container_name: librenms_myadmin
  #   ports:
  #     - 8080:80/tcp
  #   depends_on:
  #     - db
  #   environment:
  #     TZ: "${TZ}"
  #     PMA_HOST: "db"
  #     PMA_USER: "librenms"
  #     PMA_PASSWORD: "${MYSQL_PASSWORD}"

```

## 참고

* LibreNMS
  - https://github.com/librenms/librenms
  - https://github.com/librenms/docker/tree/master/examples/compose
  - https://hub.docker.com/r/librenms/librenms
* Oxidized
  - https://github.com/ytti/oxidized
  - https://hub.docker.com/r/oxidized/oxidized/
* phpMyAdmin
  - https://github.com/phpmyadmin
  - https://hub.docker.com/r/phpmyadmin/phpmyadmin
