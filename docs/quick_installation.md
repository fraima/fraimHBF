Базовая установка
===========================

Системные требования
---------------------

!!! note "Рой совместим с  `x86_64`, `amd64`, `armhf`, `arm64`,  `s390x` архитектурами."

Для установки потребуется:

`linux kernel >= 3.10.0`

`nft --version >= v0.9.3 (Topsy)`

`go version >= 1.19`

`postgresql`

Сервер
----------

=== "source"

    ``` bash
    ## INSTALL SERVER
    git clone https://github.com/H-BF/sgroups
    cd sgroups
    make sg-service
    cp bin/sg-service /usr/bin/charlotte-server
    chmod +x /usr/bin/charlotte-server

    cat <<EOF > /etc/charlotte/config-server.yaml
    server:
        endpoint: tcp://0.0.0.0:9000
    EOF

    cat <<EOF > /etc/systemd/system/Charlotte-server.service
    [Unit]
    Description=Charlotte SERVER
    Documentation=https://docs.hbf.fraima.io
    After=network.target

    [Service]
    ExecStart=/usr/bin/charlotte-server -config /etc/charlotte/config-server.yaml
    Restart=always
    RestartSec=5
    Delegate=yes
    KillMode=process
    OOMScoreAdjust=-999
    LimitNOFILE=1048576
    LimitNPROC=infinity
    LimitCORE=infinity

    [Install]
    WantedBy=multi-user.target
    EOF

    systemctl enable charlotte-server.service
    systemctl start charlotte-server.service
    ```

=== "docker"

    ``` bash
    ```

=== "deb"

    ``` bash
    ```

=== "rpm"

    ``` bash
    ```

Агент
----------


=== "source"

    ``` bash
    ## INSTALL CLIENT
    git clone https://github.com/H-BF/sgroups
    cd sgroups
    make to-nft
    cp bin/to-nft /usr/bin/charlotte-client
    chmod +x /usr/bin/charlotte-client

    cat <<EOF > /etc/charlotte/config-client.yaml
    server:
      ---
      graceful-shutdown: 10s
      logger:
        level: INFO

      extapi:
        svc:
          def-daial-duration: 10s
          sgroups:
            dial-duration: 3s
            address: tcp://${charlotte_server}:9000
            check-sync-status: 15s
    EOF

    cat <<EOF > /etc/systemd/system/charlotte-client.service
    [Unit]
    Description=Charlotte CLIENT
    Documentation=https://docs.fraima.io
    After=network.target

    [Service]
    ExecStart=/usr/bin/charlotte-client --config=/etc/charlotte/config-client.yaml
    Restart=always
    RestartSec=5
    Delegate=yes
    KillMode=process
    OOMScoreAdjust=-999
    LimitNOFILE=1048576
    LimitNPROC=infinity
    LimitCORE=infinity

    [Install]
    WantedBy=multi-user.target
    EOF

    systemctl enable charlotte-client.service
    systemctl start  charlotte-client.service
    ```

=== "docker"

    ``` bash
    ```

=== "deb"

    ``` bash
    ```

=== "rpm"

    ``` bash
    ```

Terraform провайдер
----------
=== "Terraform provider"

    ``` bash
    ## INSTALL Terraform provider
    git clone https://github.com/H-BF/sgroups
    cd sgroups
    make make sgroups-tf
    mkdir -p ~/.terraform.d/plugins/registry.terraform.io/fraima/charlotte/1.0.0/linux_amd64

    cp bin/terraform-provider-sgroups ~/.terraform.d/plugins/registry.terraform.io/fraima/charlotte/1.0.0/linux_amd64/terraform-provider-charlotte

    chmod +x ~/.terraform.d/plugins/registry.terraform.io/fraima/charlotte/1.0.0/linux_amd64/terraform-provider-charlotte

    cat <<EOF >> ~/.terraformrc
    plugin_cache_dir = "$HOME/.terraform.d/plugin-cache"
    disable_checkpoint = true
    EOF

    ```

