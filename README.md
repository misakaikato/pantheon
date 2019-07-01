# Pantheon of Congestion Control [![Build Status](https://travis-ci.org/StanfordSNR/pantheon.svg?branch=master)](https://travis-ci.org/StanfordSNR/pantheon)
The Pantheon contains wrappers for many popular practical and research
congestion control schemes. The Pantheon enables them to run on a common
interface, and has tools to benchmark and compare their performances.
Pantheon tests can be run locally over emulated links using
[mahimahi](http://mahimahi.mit.edu/) or over the Internet to a remote machine.

Our website is <https://pantheon.stanford.edu>, where you can find more
information about Pantheon, including supported schemes, measurement results
on a global [testbed](https://github.com/StanfordSNR/observatory)
so far, and our paper at [USENIX ATC 2018](https://www.usenix.org/conference/atc18/presentation/yan-francis)
(**Awarded Best Paper**).

To discuss and talk about Pantheon-related topics and issues, feel free to
post in the [Google Group](https://groups.google.com/forum/#!forum/pantheon-stanford)
or send an email to `pantheon-stanford <at> googlegroups <dot> com`.

## Disclaimer
This is research software. Our scripts will write to the file system in the
`pantheon` folder. We never run third party programs as root, but we cannot
guarantee they will never try to escalate privilege to root.

You might want to install dependencies and run the setup on your own, because
our handy scripts will install packages and perform some system-wide settings
(e.g., enabling IP forwarding, loading kernel modeuls) as root.
Please run at your own risk.

## Preparation
To clone this repository, run:

```
git clone https://github.com/StanfordSNR/pantheon.git
```

Many of the tools and programs run by the Pantheon are git submodules in the
`third_party` folder. To add submodules after cloning, run:

```
git submodule update --init --recursive  # or tools/fetch_submodules.sh
```

## Dependencies
We provide a handy script `tools/install_deps.sh` to install globally required
dependencies, but you might want to inspect the contents of this script and
install these dependencies by yourself. In particular, we created
the [Pantheon-tunnel](https://github.com/StanfordSNR/pantheon-tunnel)
that is required to instrument each scheme.

For those dependencies required by each congestion control scheme `<cc>`,
run `src/wrappers/<cc>.py deps` to print a dependency list. You could install
them by yourself, or run

```
src/experiments/setup.py --install-deps (--all | --schemes "<cc1> <cc2> ...")
```

to install dependencies required by all schemes or a list of schemes separated
by spaces. Please note that Pantheon only supports Python 2.7.

## Setup
After installing dependencies, run

```
src/experiments/setup.py [--setup] [--all | --schemes "<cc1> <cc2> ..."]
```

to set up supported congestion control schemes. `--setup` is required
to be run only once. In contrast, `src/experiments/setup.py` is
required to be run on every reboot (without `--setup`).

## Running the Pantheon
To test schemes in emulated networks locally, run

```
src/experiments/test.py local (--all | --schemes "<cc1> <cc2> ...")
```

To test schemes over the Internet to remote machine, run

```
src/experiments/test.py remote (--all | --schemes "<cc1> <cc2> ...") HOST:PANTHEON-DIR
```

Run `src/experiments/test.py local -h` and `src/experiments/test.py remote -h`
for detailed usage and additional optional arguments, such as multiple flows,
running time, arbitrary set of mahimahi shells for emulation tests,
data sender side for real tests; use `--data-dir DIR` to specify an
an output directory to save logs.

## Pantheon analysis
To analyze test results, run

```
src/analysis/analyze.py --data-dir DIR
```

It will analyze the logs saved by `src/experiments/test.py`, then generate
performance figures and a full PDF report `pantheon_report.pdf`.

## Running a single congestion control scheme
All the available schemes can be found in `src/config.yml`. To run a single
congestion control scheme, first follow the **Dependencies** section to install
the required dependencies.

At the first time of running, run `src/wrappers/<cc>.py setup`
to perform the persistent setup across reboots, such as compilation,
generating or downloading files to send, etc. Then run
`src/wrappers/<cc>.py setup_after_reboot`, which also has to be run on every
reboot. In fact, `test/setup.py` performs `setup_after_reboot` by
default, and runs `setup` on schemes when `--setup` is given.

Next, execute the following command to find the running order for a scheme:
```
src/wrappers/<cc>.py run_first
```

Depending on the output of `run_first`, run

```
# Receiver first
src/wrappers/<cc>.py receiver port
src/wrappers/<cc>.py sender IP port
```

or

```
# Sender first
src/wrappers/<cc>.py sender port
src/wrappers/<cc>.py receiver IP port
```

Run `src/wrappers/<cc>.py -h` for detailed usage.

## How to add your own congestion control
Adding your own congestion control to Pantheon is easy! Just follow these
steps:

1. Fork this repository.

2. Add your congestion control repository as a submodule to `pantheon`:

   ```
   git submodule add <your-cc-repo-url> third_party/<your-cc-repo-name>
   ```

   and add `ignore = dirty` to `.gitmodules` under your submodule.

3. In `src/wrappers`, read `example.py` and create your own `<your-cc-name>.py`.
   Make sure the sender and receiver run longer than 60 seconds; you could also
   leave them running forever without the need to kill them.

4. Add your scheme to `src/config.yml` along with settings of
   `name`, `color` and `marker`, so that `src/experiments/test.py` is able to
   find your scheme and `src/analysis/analyze.py` is able to plot your scheme
   with the specified settings.

5. Add your scheme to `SCHEMES` in `.travis.yml` for continuous integration testing.

6. Send us a pull request and that's it, you're in the Pantheon!

===========================================================================

misakaikato 中文安装tips
1、submodles 没法自动安装时，可以手动git clone下来
2、在准备阶段的某些文件无法下载，是因为文件保存在谷歌的云服务器上，这个时候手动下载就好
```
将”gs://”替换成”https://storage.googleapis.com/“，即通过HTTPS协议进行下载
wget --no-check-certificate https://storage.googleapis.com/chromium-binutils/5e71702981e5f3b45632f2f209eb3a85d65ca764 -O /home/misaka/lb/pantheon/third_party/proto-quic/src/third_party/binutils/Linux_x64/binutils.tar.bz2
```
