<h1 align="center">
    <br>
    <a href="https://github.com/ClaudiuGeorgiu/Obfuscapk">
        <img alt="Logo" src="./docs/logo/logo.png" width="700">
    </a>
    <br>
    <br>
</h1>

> A black-box obfuscation tool for Android apps.

[![Python Version](http://img.shields.io/badge/Python-3.7-green.svg)](https://www.python.org/downloads/release/python-374/)
[![License](https://img.shields.io/badge/license-MIT-blue.svg)](https://github.com/ClaudiuGeorgiu/Obfuscapk/blob/master/LICENSE)

**Obfuscapk** is a modular Python tool for obfuscating Android apps without needing their source code, since
[`apktool`](https://ibotpeaches.github.io/Apktool/) is used to decompile the original apk file and to build a new
application, after applying some obfuscation techniques on the decompiled smali code, resources and manifest.
The obfuscated app retains the same functionality as the original one, but the differences under the hood sometimes
make the new application very different from the original (e.g., to signature based antivirus software).

***More details coming soon***



## ❱ Demo

![Demo](./docs/demo/cli.gif)



## ❱ Installation

There are two ways of getting a working copy of Obfuscapk on your own computer: either by [using Docker](#docker-image)
or by [using directly the source code](#from-source) in a `Python 3.7` environment. In both cases, the first thing to
do is to get a local copy of this repository, so open up a terminal in the directory where you want to save the project
and clone the repository:

```Shell
$ git clone https://github.com/ClaudiuGeorgiu/Obfuscapk.git
```

### Docker image

------------------------------------------------------------------------------------------------------------------------

#### Prerequisites

This is the suggested way of installing Obfuscapk, since the only requirement is to have a recent version of Docker
installed:

```Shell
$ docker --version             
Docker version 19.03.0, build aeac949
```

#### Official Docker Hub image

***Coming soon, until then build the Docker image manually***

#### Install

If you downloaded the official image from Docker Hub, you are ready to use the tool so go ahead and check the
[usage instructions](#-usage), otherwise execute the following commands in the previously created `Obfuscapk/src/`
directory (the folder containing the `Dockerfile`) in order to build the Docker image:

```Shell
$ # Make sure to run the command in Obfuscapk/src/ directory.
$ # It will take some time to download and install all the dependencies.
$ docker build -t obfuscapk .
```

When the Docker image is ready, make a quick test to check that everything was installed correctly:

```Shell
$ docker run --rm -it obfuscapk --help
usage: python3.7 -m obfuscapk.cli [-h] -o OBFUSCATOR [-w DIR] [-d OUT_APK]
...
```

Obfuscapk is now ready to be used, see the [usage instructions](#-usage) for more information.

### From source

------------------------------------------------------------------------------------------------------------------------

#### Prerequisites

Make sure to have [`apktool`](https://ibotpeaches.github.io/Apktool/),
[`jarsigner`](https://docs.oracle.com/javase/8/docs/technotes/tools/windows/jarsigner.html) and
[`zipalign`](https://developer.android.com/studio/command-line/zipalign) installed and available from command line:

```Shell
$ apktool          
Apktool v2.4.0 - a tool for reengineering Android apk files
...
```
```Shell
$ jarsigner
Usage: jarsigner [options] jar-file alias
       jarsigner -verify [options] jar-file [alias...]
...
```
```Shell
$ zipalign
Zip alignment utility
Copyright (C) 2009 The Android Open Source Project
...
```

To install and use `apktool` you need a recent version of Java, which should also have `jarsigner` bundled. `zipalign`
is included in the Android SDK. The location of the executables can also be specified through the following environment
variables: `APKTOOL_PATH`, `JARSIGNER_PATH` and `ZIPALIGN_PATH` (e.g., in Ubuntu, run
`export APKTOOL_PATH=/custom/location/apktool` before running Obfuscapk in the same terminal).

Apart from the above tools, the only requirement of this project is a working `Python 3.7` installation (along with
its package manager `pip`).

#### Install

Run the following commands in the main directory of the project (`Obfuscapk/`) to install the needed dependencies: 

```Shell
$ # Make sure to run the commands in Obfuscapk/ directory.

$ # The usage of a virtual environment is highly recommended, e.g., virtualenv.
$ # If not using virtualenv (https://virtualenv.pypa.io/), skip the next 2 lines.
$ virtualenv -p python3.7 venv
$ source venv/bin/activate

$ # Install Obfuscapk's requirements.
$ python3.7 -m pip install -r src/requirements.txt
```

After the requirements are installed, make a quick test to check that everything works correctly:

```Shell
$ cd src/
$ # The following command has to be executed always from Obfuscapk/src/ directory
$ # or by adding Obfuscapk/src/ directory to PYTHONPATH environment variable.
$ python3.7 -m obfuscapk.cli --help
usage: python3.7 -m obfuscapk.cli [-h] -o OBFUSCATOR [-w DIR] [-d OUT_APK]
...
```

Obfuscapk is now ready to be used, see the [usage instructions](#-usage) for more information.



## ❱ Usage

From now on, Obfuscapk will be considered as an executable available as `obfuscapk`, so you need to adapt the commands
according to how you installed the tool:

* **Docker image**: a local directory containing the application to obfuscate has to be mounted to `/workdir` in the
container (e.g., the current directory `"${PWD}"`), so the command:
```Shell
$ obfuscapk [params...]
```
becomes:
```Shell
$ docker run --rm -it -u $(id -u):$(id -g) -v "${PWD}":"/workdir" obfuscapk [params...]
```

* **From source**: every instruction has to be executed from the `Obfuscapk/src/` directory (or by adding
`Obfuscapk/src/` directory to `PYTHONPATH` environment variable) and the command:
```Shell
$ obfuscapk [params...]
```
becomes:
```Shell
$ python3.7 -m obfuscapk.cli [params...]
```

Let's start by looking at the help message: 

```Shell
$ obfuscapk --help
obfuscapk [-h] -o OBFUSCATOR [-w DIR] [-d OUT_APK] [-i] [-p] [-k VT_API_KEY] <APK_FILE>
```

There are two mandatory parameters: `<APK_FILE>`, the path (relative or absolute) to the apk file to obfuscate and the
list with the names of the obfuscation techniques to apply (specified with the `-o` option, e.g.,
`-o Rebuild -o NewSignature -o NewAlignment`). The other optional arguments are as follows:

* `-w DIR` is used to set the working directory where to save the intermediate files (generated by `apktool`). If not
specified, a directory named `obfuscation_working_dir` is created in the same directory as the input application. This
can be useful for debugging purposes, but if it's not needed it can be set to a temporary directory (e.g., `-w /tmp/`).

* `-d OUT_APK` is used to set the path of the destination file: the apk file generated by the obfuscation process
(e.g., `-d /home/user/Desktop/obfuscated.apk`). If not specified, the final obfuscated file will be saved inside the
working directory. Note: existing files will be overwritten without any warning.

* `-i` is a flag for ignoring known third party libraries during the obfuscation, in order to use less resources, to
increase performances and to reduce the risk of errors. The
[list of libraries](./src/obfuscapk/resources/libs_to_ignore.txt) to ignore is adapted from
[LiteRadar](https://github.com/pkumza/LiteRadar) project.

* `-p` is a flag for showing progress bars during the obfuscation operations. When using the tool in batch
operations/automatic builds it's convenient to have progress bars disabled, otherwise this flag should be enabled to
see the obfuscation progress.

* `-k VT_API_KEY` is needed only when using `VirusTotal` obfuscator, to set the API key(s) to be used when
communicating with Virus Total. Can be set multiple times to cycle through the API keys during the requests
(e.g., `-k VALID_VT_KEY_1 -k VALID_VT_KEY_2`). 

Let's consider now a simple working example to see how Obfuscapk works:

```Shell
$ # original.apk is a valid Android apk file.
$ obfuscapk -o RandomManifest -o Rebuild -o NewSignature -o NewAlignment original.apk
```

When running the above command, this is what happens behind the scenes:

* since no working directory was specified, a new working directory (`obfuscation_working_dir`) is created in the same
location as `original.apk` (this can be useful to inspect the `smali` files/manifest/resources in case of errors)

* some checks are performed in order to make sure that all the needed files/executables are available

* the actual obfuscation process begins: the specified obfuscators are executed (in order) one by one until there's no
obfuscator left or until an error is encountered

    - when running the first obfuscator, `original.apk` is decompiled with `apktool` and the results are stored
    into the working directory

    - since the first obfuscator is `RandomManifest`, the entries in the decompiled Android manifest are reordered
    randomly (without breaking the `xml` structures)

    - `Rebuild` obfuscator simply rebuilds the application (now with the modified manifest) using `apktool`, and since
    no output file was specified, the resulting apk file is saved in the working directory created before

    - `NewSignature` obfuscator signs the newly created apk file with a custom certificate contained in
    [this keystore](./src/obfuscapk/resources/obfuscation_keystore.jks)

    - `NewAlignment` obfuscator uses `zipalign` tool to align the resulting apk file

* when all the obfuscators have been executed without errors, the resulting obfuscated apk file can be found in
`obfuscation_working_dir/original_obfuscated.apk`, signed, aligned and ready to be installed into a device/emulator

As seen in the previous example, `Rebuild`, `NewSignature` and `NewAlignment` obfuscators are always needed to complete
an obfuscation operation, in order to build the final obfuscated apk. They are not actual obfuscators, but are needed
for the build process and they are included in the list of obfuscators to keep the overall architecture modular.



## ❱ Contributing

Questions, bug reports and pull requests are welcome on GitHub at
[https://github.com/ClaudiuGeorgiu/Obfuscapk](https://github.com/ClaudiuGeorgiu/Obfuscapk).



## ❱ License

You are free to use this code under the [MIT License](./LICENSE).
