# Ubuntu 创建新用户账户


# Ubuntu 创建新用户账户

## 创建新用户账户

```bash
sudo useradd -s /bin/bash -m -G sudo test
```

- `-s /bin/bash`: 将 `/bin/bash` 设置为新账户的登录 shell
- `-m`: 创建用户的主目录
- `-G sudo`: 确保用户可以使用 `sudo`

## 设置新用户账户密码

```bash
sudo passwd test
```

## 删除用户账户

```bash
userdel -r test
```

- `-r`: 删除用户主目录


