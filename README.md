
# RevSys Python Builds

## What’s wrong with stock Python images? 

The main image variety is too big when the user is operating in a context
that involves multiple private registries.

It’s base layer is an Ubuntu image (with a few exceptions, this
provides an as-installed-by-Ubuntu environment) which is, essentially, what
one gets with a base minimal install of Ubuntu 16.04.

The [alpine](https://alpinelinux.org/) images are delightfully wee. But, alas,
[musl](https://www.musl-libc.org/). It is awesome. I mean, it *exists*. If you
don’t have build problems with your modules that have a C or C++ component,
use it. It is small, fast and well vetted.

**Note**: There are workarounds for *some* of the more common non-pure-python
module build failures but usually involve much shenaniganry that I have neither the time
nor inclination to pursue.

The **slim** images are similar to the ubuntu images. Rather than an Ubuntu base
image, they use a Debian 8 base.

| REPOSITORY                                      | TAG                     | SIZE  |
| -----------                                     | ----------------------  | ----- |
| python                                          | 3.6.3                   | 690MB |
| python                                          | 3.6-slim                | 201MB |
| python                                          | 3.6.3-alpine            | 89MB  |
| registry.gitlab.com/revsys/docker-builds/python | 3.6.3-wee               | 153MB |

We settled on a Debian 8 image provided by google’s GCE service. It is smaller
by way of excluding systemd-related bits while building the initial bootstrap
chroot.

The resulting images are smaller than -slim but larger than alpine.

## Optimized Python images

Three builds are available per Python version:

 * :wee - stock build without compile or link-time optimizations enabled
 * :wee-optimized - build with profile-guided optimization flag
 * :wee-optimized-lto - build with PGO + Link-Time-Optimization flags


| REPOSITORY                                      | TAG                     | SIZE  |
| -----------                                     | ----------------------  | ----- |
| revolutionsystems/python | 3.5.3-wee-optimized-lto | 164MB |
| revolutionsystems/python | 3.5.3-wee-optimized | 164MB |
| revolutionsystems/python | 3.5.3-wee | 158MB |

## Building

The registry is included from <gitroot>/.versions

make targets:

 * build-image: docker build...
 * push-image: docker push ...
 * image: runs build-image then push-image

### stock build

```

:# LTO= OPTIMAL= PYTHON_VERSION=3.5.3 make image

```

### PGO optimizations

```

:# LTO= OPTIMAL=1 PYTHON_VERSION=3.5.1 make image

```

### LTO Optimizations

```

:# LTO=1 OPTIMAL=1 PYTHON_VERSION=3.6.3 make image

```

### Python 2 builds

The build arguments have stayed fairly consistent at least since Python 2.6.
The build will not fail if the Python 3-specific environment variables remain
set; however, the image will get a misleading tag.  For Python 2 builds, use:

```

:# LTO= OPTIMAL= PYTHON_VERSION=2.7.13 make image

```

## Build Optimization

Build times are radically reduced by using a local proxy. You can find an 
example squid 3/4 configuration in ./examples.

After setting the `cache_dir` path the command, `squid -f /path/to/config -N -z`
to initialize the cache filesystem storage.

To kick off the cache service: `squid -f /path/to/config -N -a 3128`.

Once the proxy is running, set the environment variable `export PKG_PROXY=http://10.129.1.1:3128`.
Docker transparently passes the `HTTP_PROXY` and `HTTPS_PROXY` environment variables
into each build container if they are configured with the `--build-args` flag.

The transparency is significant in that whether or not your build is leveraging
a proxy will not affect the validity of the build host’s cache.


## Origin Notes

Based in large part on the contents of [the Docker Library Python repository](https://github.com/docker-library/python).

---

 1. uses [GNU make](https://www.gnu.org/software/make/) and environment variables to set options
    1. `LTO`: if present and not empty, activate link-time-optimization functionality
    1. `OPTIMAL`: if present and not empty, activate profiler-guided-optimization functionality
    1. `PYTHON_VERSION’: e.g.: 3.6.3, 3.5.1
 1. Does not include [TCL/TK](https://www.tcl.tk/about/uses.html). This is on purpose. If you neeed IDLE or
    pexpect you’ll need to customize the Dockerfile and rebuild.
 1. Uses multi-layer Dockerfile syntax

---

Keep in touch!
--------------

If you have a question about this project, please open a GitHub issue. If you love us and want to keep track of our goings-on, here's where you can find us online:

<a href="https://revsys.com?utm_medium=github&utm_source=optimized-python-docker"><img src="https://pbs.twimg.com/profile_images/915928618840285185/sUdRGIn1_400x400.jpg" height="50" /></a>
<a href="https://twitter.com/revsys"><img src="https://cdn1.iconfinder.com/data/icons/new_twitter_icon/256/bird_twitter_new_simple.png" height="43" /></a>
<a href="https://www.facebook.com/revsysllc/"><img src="https://cdn3.iconfinder.com/data/icons/picons-social/57/06-facebook-512.png" height="50" /></a>
<a href="https://github.com/revsys/"><img src="https://assets-cdn.github.com/images/modules/logos_page/GitHub-Mark.png" height="53" /></a>
<a href="https://gitlab.com/revsys"><img src="https://upload.wikimedia.org/wikipedia/commons/thumb/1/18/GitLab_Logo.svg/2000px-GitLab_Logo.svg.png" height="44" /></a>

