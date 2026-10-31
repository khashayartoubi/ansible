# راهنمای اجرای Ansible برای آماده‌سازی سیستم‌ها

این پروژه برای سیستم‌های Debian/Ubuntu طراحی شده و تنظیمات زیر را به‌شکل
قابل‌تکرار اعمال می‌کند:

- نصب Git `2.34.1`
- نصب Docker Engine `29.5.2` و Docker Compose `v5.0.2`
- تنظیم `/etc/docker/daemon.json` شامل DNS و insecure registries
- اجرای `systemctl daemon-reload` و restart سرویس Docker
- نصب و فعال‌سازی OpenSSH Server روی پورت `22`
- نصب `network-manager-l2tp` و `network-manager-l2tp-gnome`
- ثبت نام‌های داخلی سرویس‌ها در `/etc/hosts`

اگر نصب Docker از مخزن با خطایی مانند `403` مواجه شود، نصب از
`https://get.docker.com/` ادامه پیدا می‌کند.

## ساختار پروژه

```text
.
├── ansible.cfg
├── group_vars/all.yml          # نسخه‌ها و رکوردهای hosts
├── inventories/production      # لیست سیستم‌های مقصد
├── roles/
│   ├── common/                 # پیش‌نیازها، Git و L2TP
│   ├── docker/                 # Docker، Compose و daemon.json
│   ├── ssh/                    # OpenSSH Server
│   └── hosts/                  # رکوردهای /etc/hosts
└── site.yml                    # پلی‌بوک اصلی
```

## پیش‌نیازها

روی سیستم کنترل‌کننده، Ansible باید نصب باشد:

```bash
sudo apt update
sudo apt install ansible
```

کاربر تعریف‌شده برای هر مقصد باید دسترسی SSH و `sudo` داشته باشد. SSH باید
پیش از اجرای پلی‌بوک در دسترس باشد؛ Ansible بدون اتصال اولیه نمی‌تواند SSH
را روی سیستم کاملاً تازه نصب کند.

## تعریف سیستم‌های مقصد

فایل `inventories/production/hosts.yml` را ویرایش کنید:

```yaml
all:
  children:
    development:
      hosts:
        pc-ali:
          ansible_host: 192.168.10.50
          ansible_user: ali
          ansible_ssh_private_key_file: ~/.ssh/id_ed25519
```

برای چند سیستم، هر میزبان را زیر `development.hosts` اضافه کنید. اگر با رمز
SSH وصل می‌شوید، خط `ansible_ssh_private_key_file` را حذف کنید.

## تست اتصال

```bash
ansible development -m ping
```

در صورت استفاده از رمز SSH:

```bash
ansible development -m ping --ask-pass
```

## اجرا

از ریشهٔ پروژه اجرا کنید:

```sh
ansible-playbook site.yml --ask-become-pass
```

اگر رمز SSH نیز لازم است:

```sh
ansible-playbook site.yml --ask-pass --ask-become-pass
```
