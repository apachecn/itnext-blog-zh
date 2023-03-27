# 使用 Ansible 通过 OpenVPN 设置隐私

> 原文：<https://itnext.io/setup-privacy-with-openvpn-using-ansible-b9e613a66f85?source=collection_archive---------3----------------------->

![](img/f41bdcf89066550e3b3bba5158afd15e.png)

照片由[戴恩·托普金](https://unsplash.com/@dtopkin1?utm_source=medium&utm_medium=referral)在 [Unsplash](https://unsplash.com?utm_source=medium&utm_medium=referral) 上拍摄

在当今世界，员工经常需要远程办公的灵活性。这些远程位置可以是咖啡馆、酒店大堂或员工使用公共 wifi 的任何其他公共场所。公共 wifis 通常不安全，通过它们传输的数据很容易被黑客窃取。防止公司数据暴露的最佳防御是使用安全 wifi，但这并不总是可行的。更实用的解决方案是使用 VPN，它通过公共互联网在员工系统和服务器之间提供安全连接。这确保了客户端使用的数据在网络上可靠地传输并且不可见。另一个用例是当员工需要访问特定的资源，而这些资源在员工居住或旅行的国家/地区是不可用的。在这种情况下，员工可以通过 VPN 访问所有这些资源，因为流量是通过它路由的，网站提供商认为它是由 VPN 服务器访问的。

启用 VPN 访问公司资源有助于加强数据安全性，因为现在数据在公共互联网上不可用。VPN 有助于提高可管理性和可追溯性，因为每个人都通过该 VPN 服务器，所以我们可以跟踪谁访问了资源，这有助于隔离和识别任何可疑的活动。

在这篇文章中，我们将使用开源包 [OpenVPN](https://openvpn.net/) 建立一个 VPN。我们将使用 Ubuntu 操作系统进行设置。这个设置需要两个包 OpenVPN 和 easy-rsa。简单-rsa 允许我们创建自己的[认证机构](https://en.wikipedia.org/wiki/Certificate_authority)，生成 VPN 服务器和客户端使用的证书和密钥。这有助于我们加密、解密消息，验证客户端和服务器以提供安全的通信。OpenVPN 服务器允许用户使用正确的证书，这有助于确保通信得到授权。在客户端，客户端也使用这些证书和密钥验证它是否连接到正确的服务器。我们需要激活防火墙来阻止除 SSH (22)和 VPN(1194)之外的所有端口。我们将使用 ufw 工具进行设置。我们需要更新 IPtable 规则，以便通过互联网对 VPN 客户端进行 NAT。我们还需要通过在“sysctl”配置文件中添加 net.ipv4.ip_forward=1 并更改 ufw 配置以允许转发策略来启用数据包转发。

在我们进入实际设置之前，让我们了解一下什么是“ [ufw](https://help.ubuntu.com/community/UFW) ”。ufw 代表简单防火墙，它为管理防火墙规则提供了一个用户友好的框架。防火墙是一组网络规则，我们可以定义谁可以连接到我们的服务器，或者我们的服务器可以与谁连接。我们可以使用简单的 ufw 命令简单地允许/不允许所有传入和传出的流量。例如，我们可以使用以下命令启用对特定端口的访问:

```
sudo ufw allow ssh
```

我们可以在运行任何终端提供的规则之前，在 ufw 使用的 before.rules 文件中添加更高级的规则。设置规则后，我们可以使用以下命令检查 ufw 的状态:

```
sudo ufw status
```

如果我们发现 ufw 被禁用，那么我们可以使用以下命令启用它:

```
sudo ufw enable
```

让我们从使用 ansible 设置 VPN 服务器开始。作为第一步，我们需要更新我们的库，这样我们就可以下载和安装最新的包。我们需要安装 OpenVPN 和 easy-rsa 包。

```
- name: Update apt packages
  become: true
  apt:
    upgrade: yes- name: Install openvpn
  package:
   name: "{{ item }}"
   state: present
  with_items:
    - openvpn
    - easy-rsa
```

我们需要创建一个新的自定义证书颁发机构，这将有助于生成服务器证书和密钥对以及 Diffie-Hellman 参数和 tls-auth 密钥。我们创建 CA 目录，所有的证书/密钥都在这个目录中生成。

```
- name: "Remove CA directory"
  become: yes
  file:
    state: absent
    path: "{{ ansible_env.HOME }}/openvpn-ca/"- name: "Create CA dir"
  become: yes
  command: make-cadir {{ ansible_env.HOME }}/openvpn-ca
```

我们需要设置变量，这些变量将用于设置在 CA 创建和生成证书和密钥期间使用的值。

```
- name: Customize CA variable configuration
  lineinfile:
    dest: "{{ ansible_env.HOME }}/openvpn-ca/vars"
    regexp: "^{{ item.property | regex_escape() }}="
    line: "{{ item.property }}={{ item.value }}"
  with_items:
    - { property: 'export KEY_NAME', value: '"server"' }
    - { property: 'export KEY_COUNTRY', value: '"US"' }
    - { property: 'export KEY_PROVINCE', value: '"CA"' }
    - { property: 'export KEY_CITY', value: '"SF"' }
    - { property: 'export KEY_ORG', value: '"MT"' }
    - { property: 'export KEY_EMAIL', value: '"[mt@mt.com](mailto:mt@mt.com)"' }
    - { property: 'export KEY_OU', value: '"MT"' }
    - { property: 'export KEY_CONFIG', value: '{{ ansible_env.HOME }}/openvpn-ca/openssl-1.0.0.cnf' }
    - { property: 'export KEY_DIR', value: '{{ ansible_env.HOME }}/openvpn-ca/keys' }
```

现在让我们使用 CA 目录中提供的脚本来构建我们的认证机构。

```
- name: "Build the certificate authority"
  become: yes
  shell: >
    source vars;
    ./clean-all;
    yes "" | ./build-ca;
  args: 
    chdir: "{{ ansible_env.HOME }}/openvpn-ca/"
    executable: /bin/bash
```

下一步是为服务器生成证书。我们创建了 [Diffie-Hellman](https://en.wikipedia.org/wiki/Diffie%E2%80%93Hellman_key_exchange) 参数，这些参数用于使用公共和不安全的通道交换密钥。为了增加证书的安全性，我们将生成一个密钥来使用共享秘密。所有这些都是在 keys 文件夹中生成的。我们需要将它们移动到 OpenVPN 文件夹中，并在配置文件中指向它们。

```
- name: "Build server certificate"
  become: yes
  shell: >
    source vars;
    ./build-key-server --batch server;
  args: 
    chdir: "{{ ansible_env.HOME }}/openvpn-ca/"
    executable: /bin/bash- name: "Build Diffie-Hellman parameters and key generation"
  become: yes
  shell: >
    source vars;
    yes "" | ./build-dh;
    openvpn --genkey --secret keys/ta.key;
  args: 
    chdir: "{{ ansible_env.HOME }}/openvpn-ca/"
    executable: /bin/bash- name: "Copy key and certificates to /etc/openvpn"
  become: yes
  copy:
    remote_src: yes
    src: "{{ ansible_env.HOME }}/openvpn-ca/keys/{{ item }}"
    dest: "/etc/openvpn/"
  with_items:
    - "ca.crt"
    - "server.crt"
    - "server.key"
    - "ta.key"
    - "dh2048.pem"
```

我们将复制示例配置文件作为我们的 OpenVPN 配置文件。在此配置文件中进行必要的更改，以指向正确的证书、用户/组和 DHCP 配置。

```
- name: "Generate server.conf from sample config"
  become: yes
  shell: >
     gzip -d -c /usr/share/doc/openvpn/examples/sample-config-files/server.conf.gz | sudo tee /etc/openvpn/server.conf > /dev/null
- name: Adjust OpenVPN server configuration
  lineinfile:
    dest: "/etc/openvpn/server.conf"
    regexp: "^{{ item.regex | regex_escape() }}"
    line: "{{ item.value }}"
  with_items:
    - { regex: ';user nobody', value: 'user nobody' }
    - { regex: ';group nogroup', value: 'group nogroup' }
    - { regex: ';push "redirect-gateway def1 bypass-dhcp"', value: 'push "redirect-gateway def1 bypass-dhcp"' }
    - { regex: 'cert server.crt', value: 'cert server.crt' }
    - { regex: 'key server.key', value: 'key server.key' }
```

我们将使用 ufw 启用防火墙，允许 VPN 和 SSH 端口。更新路由规则，启用 IP 转发并接受转发策略。

```
- name: Configuration IP forwarding
  become: true
  sysctl:
    name: net.ipv4.ip_forward
    value: 1
    state: present- name: Add ufw before content
  become: true
  blockinfile:
    path: /etc/ufw/before.rules
    insertbefore: BOF
    content: |
      # NAT table rules
      *nat
      :POSTROUTING ACCEPT [0:0]
      -A POSTROUTING -s 10.8.0.0/8 -o eth0 -j MASQUERADE
      COMMIT
- name: Customize ufw forwarding policy
  become: true
  lineinfile:
    line: "DEFAULT_FORWARD_POLICY=\"ACCEPT\""
    path: "/etc/default/ufw"
    regexp: "^DEFAULT_FORWARD_POLICY=\"DROP\""- name: Open ufw ports for openvpn and ssh
  become: true
  shell:  ufw allow openvpn && ufw allow OpenSSH- name: Enable ufw
  become: true
  shell: ufw --force enable
```

完成所有工作后，我们将重启 OpenVPN 服务器以使所有更新的更改生效。这样，我们的 OpenVPN 服务器就启动并运行了。

```
- name: Start openvpn systemd service
  become: true
  systemd:
    name: openvpn@server
    state: started
    daemon_reload: yes
    enabled: yes
```

由于我们已经准备好了服务器，我们需要创建客户端配置，它可以用来连接到我们的 VPN 服务器并通过它重定向流量。我们将创建一个可以被任何 OpenVPN 客户端工具使用的“ovpn”文件。作为第一步，我们需要生成一个证书/密钥对。

```
- name: "Generate client certificate key"
  become: yes
  shell: source vars; ./build-key --batch {{client_name}}
  args: 
    chdir: "{{ ansible_env.HOME }}/openvpn-ca/"
    executable: /bin/bash- name: "Create client certificate configs dir"
  become: yes
  file: 
    owner: "{{ ansible_env.USER }}"
    group: "{{ ansible_env.USER }}"
    path: "{{ ansible_env.HOME }}/openvpn-ca/{{client_name}}"
    state: directory
    mode: 0700
```

我们将复制一个示例客户端配置文件，然后我们将修改该文件以更新配置参数，如服务器 IP 地址、客户端证书、客户端密钥、ta 密钥，从而使用共享密钥进行其他所需的更改。

```
- name: "Copy client sample configs from remote host itself"
  become: yes
  copy:
      remote_src: yes
      src: /usr/share/doc/openvpn/examples/sample-config-files/client.conf
      dest: "{{ ansible_env.HOME }}/openvpn-ca/{{client_name}}/{{client_name}}.ovpn"- name: Set the server ip and port
  lineinfile:
    dest: "{{ ansible_env.HOME }}/openvpn-ca/{{client_name}}/{{client_name}}.ovpn"
    regexp: "^{{ item.regex | regex_escape() }}"
    line: "{{ item.value }}"
  with_items:
    - { regex: 'remote my-server-1 1194', value: 'remote {{ groups["openVPN"][0] }} 1194' }
    - { regex: ';user nobody', value: 'user nobody' }
    - { regex: ';group nogroup', value: 'group nogroup' }
    - { regex: 'ca ca.crt', value: '#ca ca.crt' }
    - { regex: 'cert client.crt', value: '#cert client.crt' }
    - { regex: 'key client.key', value: '#key client.key' }
    - { regex: 'tls-auth ta.key 1', value: '#tls-auth ta.key 1' }- name: "Create client ovpn file"
  become: yes
  shell: "{{ item }}"
  with_items:
    - echo -e '<ca>' >> {{ ansible_env.HOME }}/openvpn-ca/{{client_name}}/{{client_name}}.ovpn
    - cat {{ ansible_env.HOME }}/openvpn-ca/keys/ca.crt >> {{ ansible_env.HOME }}/openvpn-ca/{{client_name}}/{{client_name}}.ovpn
    - echo -e '</ca>\n<cert>' >> {{ ansible_env.HOME }}/openvpn-ca/{{client_name}}/{{client_name}}.ovpn
    - cat {{ ansible_env.HOME }}/openvpn-ca/keys/{{client_name}}.crt >> {{ ansible_env.HOME }}/openvpn-ca/{{client_name}}/{{client_name}}.ovpn
    - echo -e '</cert>\n<key>' >> {{ ansible_env.HOME }}/openvpn-ca/{{client_name}}/{{client_name}}.ovpn
    - cat {{ ansible_env.HOME }}/openvpn-ca/keys/{{client_name}}.key >> {{ ansible_env.HOME }}/openvpn-ca/{{client_name}}/{{client_name}}.ovpn
    - echo -e '</key>\n<tls-auth>' >> {{ ansible_env.HOME }}/openvpn-ca/{{client_name}}/{{client_name}}.ovpn
    - cat {{ ansible_env.HOME }}/openvpn-ca/keys/ta.key >> {{ ansible_env.HOME }}/openvpn-ca/{{client_name}}/{{client_name}}.ovpn
    - echo -e '</tls-auth>' >> {{ ansible_env.HOME }}/openvpn-ca/{{client_name}}/{{client_name}}.ovpn
    - echo -e 'key-direction 1' >> {{ ansible_env.HOME }}/openvpn-ca/{{client_name}}/{{client_name}}.ovpn
  args:
    chdir: "{{ ansible_env.HOME }}/openvpn-ca/"
    executable: /bin/bash
```

一旦一切完成，我们就可以将“ovpn”文件提取到本地实例，并与客户端共享，这样他们就可以使用它来连接我们的 vpn 服务器。

```
- name: Fetch client configurations
  fetch:
    src: "{{ ansible_env.HOME }}/openvpn-ca/{{client_name}}/{{ item|basename }}"
    dest: "{{ destination_key }}/"
    flat: yes
  with_items:
    - "{{client_name}}.ovpn"
```

我们可以在任何 VPN 客户端上使用这个文件来连接 VPN 服务器。如果你使用的是 mac，那么你可以使用 [Tunnelblick](https://tunnelblick.net/) ，这是一个易于使用的 VPN 客户端。

完整的代码可以在这个 git 资源库中找到:【https://github.com/MiteshSharma/OpenVPNAnsible

***PS:如果你喜欢这篇文章，请鼓掌支持*** 👏 ***。欢呼***