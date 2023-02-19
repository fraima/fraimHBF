FraimHBF Quick Installation
===========================

**Prerequisites**

**OS requirements**

FraimHBF совместим с  `x86_64`, `amd64`, `armhf`, `arm64`,  `s390x` архитектурами.

**Nftables requirements**

`nft --version >= v.1.0.2`

**DB requirements**

`postgresql`

**Install FraimHBF**

=== "SOURCES"

    ``` bash
    ## INSTALL SERVER
    git clone https://github.com/H-BF/sgroups
    cd sgroups
    make sg-service
    cp bin/sg-service /usr/bin/

    cat <<EOF > /etc/systemd/system/fraimhbf-server.service
    [Unit]
    Description=FraimHBF SERVER
    Documentation=https://docs.fraima.io
    After=network.target

    [Service]
    ExecStart=/usr/bin/sg-service
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

    systemctl enable fraimhbf-server.service
    systemctl start fraimhbf-server.service
    ```

    ``` bash
    ## INSTALL CLIENT
    git clone https://github.com/H-BF/sgroups
    cd sgroups
    make to-nft
    cp bin/to-nft /usr/bin/

    cat <<EOF > /etc/systemd/system/fraimhbf-client.service
    [Unit]
    Description=FraimHBF CLIENT
    Documentation=https://docs.fraima.io
    After=network.target

    [Service]
    ExecStart=/usr/bin/to-nft --config=/etc/fraimHBF/config.yaml
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

    systemctl enable fraimhbf-client.service
    systemctl start fraimhbf-client.service
    ```

=== "RPM"

    ``` bash
    ## INSTALL SERVER
    sudo yum -y install fraimhbf-server
    ```

    ``` bash
    ## INSTALL CLIENT
    sudo yum -y install fraimhbf-client
    ```

=== "DEB"

    ``` bash
    ## INSTALL SERVER
    sudo apt -y install fraimhbf-server
    ```

    ``` bash
    ## INSTALL CLIENT
    sudo apt -y install fraimhbf-client
    ```

