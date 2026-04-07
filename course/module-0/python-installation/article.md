# Установка Python

При прохождении нашего курса установка Python не обязательна: все предлагаемые задачи запускаются на наших серверах, а произвольный Python‑код всегда можно выполнить в **[Песочнице](/sandbox)**.

Но если вы хотите запускать код у себя локально, ниже — короткий гайд по установке.

В этом коротком гайде вы установите актуальную версию Python 3 на свою систему и проверите, что всё работает. Поехали! 🚀

## Шаг 1. Проверьте, не установлен ли Python

Откройте терминал/консоль и выполните команды проверки версии.

-   **Windows (PowerShell или CMD):**

    ```bash
    python --version
    py --version
    pip --version
    ```

-   **macOS / Linux (Terminal):**

    ```bash
    python3 --version
    pip3 --version
    ```

Если видите номер версии (например, `Python 3.12.x`), Python уже установлен. Если команды не найдены — устанавливаем Python.

## Установка на Windows

1. Скачайте официальный установщик: **[python.org → Downloads → Windows](https://www.python.org/downloads/windows/)**.
2. Запустите установщик.
3. На первом экране поставьте галочку **«Add python.exe to PATH»** (очень важно!) и нажмите **Install Now**.
4. Дождитесь завершения и закройте установщик.
5. Проверьте установку в PowerShell/CMD:

```bash
py --version
python --version
pip --version
```

Если команда `python` не находится, попробуйте `py` или выйдите/войдите в систему. При необходимости перезагрузите компьютер.

## Установка на macOS

Есть два удобных способа — выберите любой.

### Способ A: Официальный установщик

1. Скачайте **[python.org → Downloads → macOS](https://www.python.org/downloads/macos/)**.
2. Откройте `.pkg` и следуйте инструкциям установщика.
3. Проверьте версии:

```bash
python3 --version
pip3 --version
```

### Способ B: Через Homebrew

1. Убедитесь, что установлен Homebrew (`brew -v`). Если нет, следуйте инструкции на **[brew.sh](https://brew.sh/)**.
2. Установите Python:

```bash
brew install python
```

3. Проверьте версии:

```bash
python3 --version
pip3 --version
```

Примечание: на macOS обычно используется команда `python3` (а не `python`). Это нормально.

## Установка на Linux

Откройте терминал и выполните команды для вашего дистрибутива.

-   **Ubuntu / Debian:**

    ```bash
    sudo apt update
    sudo apt install -y python3 python3-pip
    ```

-   **Fedora:**

    ```bash
    sudo dnf install -y python3 python3-pip
    ```

-   **Arch Linux:**

    ```bash
    sudo pacman -S python python-pip
    ```

Проверьте версии:

```bash
python3 --version
pip3 --version
```

## Частые вопросы

-   **Команда `python` не найдена.** На Windows попробуйте `py`. На macOS/Linux используйте `python3` и `pip3` — это нормально.
-   **Обновить `pip`:**

    ```bash
    python -m pip install --upgrade pip
    # или
    python3 -m pip install --upgrade pip
    ```

Удачи в изучении Python! ✨
