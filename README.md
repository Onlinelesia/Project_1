# Project_1
README for HW
# Установить 2 VM, настроить статические IP-адреса, чтобы VM работали в одной сети и проверить ping.
# Установка и настройка Apache на VM1 через apt
sudo apt update
sudo apt install apache2

# После завершения установки сервер Apache должен быть запущен автоматически. Проверить его статус командой
sudo systemctl status apache2

# Установить фаервол
sudo apt install ufw

# Проверить статус фаервола
sudo ufw status

