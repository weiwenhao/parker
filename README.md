# Parker

A lightweight tool and runtime to package a directory into an executable and run it as a lightweight container.

It is used for linux systems. Illustration:

![](https://raw.githubusercontent.com/weiwenhao/pictures/main/blogs20230915112230.png)

The example is a C language-based IP parsing service `gcc -o ipservice`, which relies on the ipdb resource file.

Using Parker, compress and package the executable file `ipservice` and its dependencies into a new executable `ipserviced`.

Running `ipserviced` on the target machine will create a lightweight container environment to run the original `ipservice`.

## 💾  Installation

Download and unpack the Parker installation package from [github releases](https://github.com/weiwenhao/parker/releases). It is recommended to move the unpacked `parker` folder to `/usr/local/` and add the `/usr/local/parker/bin` directory to your system's environment variable.

```
> parker --version
1.0.1
```

## 📦 Usage

Navigate to the working directory and execute `parker target`. This command will package the target along with the current working directory into a new executable named `targetd`. Transfer this executable to the target machine and run.

```
> cd :workdir && parker :target
```

#### Example

The packaging of the executable and resource files mentioned above is a **standard use case**. Of course, there are some non-standard ways to use it, for instance, with a python3.11 based server:

```
> tree .
├── bar.png
├── foo.txt
├── python # cp /usr/bin/python3.11 ./
└── server.py

0 directories, 4 files
```

Content of `server.py`:

```python
from http.server import SimpleHTTPRequestHandler, HTTPServer

def run():
    print("listen on http://127.0.0.1:8000")
    server_address = ('127.0.0.1', 8000)
    httpd = HTTPServer(server_address, SimpleHTTPRequestHandler)
    httpd.serve_forever()

run()
```

Navigate to the working directory and execute `parker python`. You'll get a `pythond` file, which is the packaged executable. Transfer it to the target machine and run.

```bash
> parker python
pythond
├── server.py
├── python
├── foo.txt
└── bar.png
🍻 parker successful

------------------------------------------------------------------------ move pyhond to target
> tree .
.
└── pythond

0 directories, 1 file

------------------------------------------------------------------------ run pythond
> ./pythond server.py
listen on http://127.0.0.1:8000

```

Now you just need to replace `python` with `pythond`, and no other start-up parameters need to change. `pythond` will pass the parameters to the `python` process.

> ❗️ Parker does not address the dynamic library dependencies of python.

## 🚢 Runtime details

`pythond` is a lightweight container runtime built by Parker, and it's a statically compiled executable. When executed, it uses the Linux namespace to create an isolated environment, unpacks the working directory, and runs the target `python`.

`pythond` monitors the main `python` process. Once the `python` process stops or encounters an error, `pythond` cleans the container environment through `cgroup` and also cleans all child processes of `python`.

All parameters and signals passed to `pythond` will be passed on to the `python` process unchanged.

## 🐧 Runtime Dependencies

The container runtime depends on `cgroup` and `namespace`, requiring a Linux kernel version greater than 2.6.24. Ensure that `cgroup` is correctly mounted. Check for the existence of either the `/sys/fs/cgroup/cgroup.controllers` file or the `/sys/fs/cgroup/freezer` directory.

Tested environments: ubuntu:22 / ubuntu:20

## 🛠️ Make build

The source code is developed in the programming language [nature](https://github.com/nature-lang/nature). You'll need the `nature` compiler version >= 0.4.0. After installation, execute `make amd64 && make install` in the source directory to install to the `/usr/local/parker` directory.


> `nature` primarily supports amd64 builds. Executables built with `nature` are more compact and efficient. For other architectures, the main repository provides a Golang implementation.

## 🎉 Thinks

[nature](https://github.com/nature-lang/nature) is the next-generation system-level programming language, destined to work alongside C for high-performance and efficient development.

The `nature` community version will be released soon. You can try it out now and provide feedback. We invite you to contribute to the standard library, and all contributions will be merged into the main repository.

## 🪶 License

This project is open-sourced software licensed under the [MIT license](https://opensource.org/licenses/MIT).

Copyright (c) 2020-2023 WEIWENHAO, all rights reserved.
