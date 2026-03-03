```
#!/usr/bin/exec-suid -- /usr/bin/python3 -I

import os
import sys
from collections import namedtuple

Pixel = namedtuple("Pixel", ["ascii"])


def main():
    if len(sys.argv) >= 2:
        path = sys.argv[1]
        assert path.endswith(".cimg"), "ERROR: file has incorrect extension"
        file = open(path, "rb")
    else:
        file = sys.stdin.buffer

    header = file.read1(6)
    assert len(header) == 6, "ERROR: Failed to read header!"

    assert header[:4] == b"<0%r", "ERROR: Invalid magic number!"

    assert int.from_bytes(header[4:6], "little") == 125, "ERROR: Invalid version!"

    with open("/flag", "r") as f:
        flag = f.read()
        print(flag)


if __name__ == "__main__":
    try:
        main()
    except AssertionError as e:
        print(e, file=sys.stderr)
        sys.exit(-1)

```

В задании выполняются следующие проверки:

- Расширение файла: Должно заканчиваться на `.cimg`
- Заголовок (всего 8 байт):

- Магическое число (4 байта): Должно быть `b"<0%r"`

- Версия (4 байта): Должно быть `11` в формате little-endian

exploit:
```
import struct

magic = b"<0%r"                 # 4 bytes
version = struct.pack("<H", 125)  # 2 bytes, little-endian unsigned short

header = magic + version        # total 6 bytes

filename = "/home/hacker/solution.cimg"
with open(filename, "wb") as f:
    f.write(header)

print(f"Wrote {len(header)} bytes: {header} to file: '{filename}'")
```


```
cat << 'EOF' > script.py
#!/usr/bin/env python3

import struct

magic = b"<0%r"                 # 4 bytes
version = struct.pack("<H", 125)  # 2 bytes, little-endian unsigned short

header = magic + version        # total 6 bytes

filename = "/home/hacker/solution.cimg"
with open(filename, "wb") as f:
    f.write(header)

print(f"Wrote {len(header)} bytes: {header} to file: '{filename}'")
EOF


```

